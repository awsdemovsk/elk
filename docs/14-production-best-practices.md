# 14 – Production Best Practices

## Topology
```
Beats → (Kafka/Logstash) → 3 dedicated masters
                         → hot data nodes (SSD) → warm → cold/frozen (searchable snapshots on S3/Blob/GCS)
Kibana x2 behind a load balancer; Logstash x2+ ingest tier
```
- **Separate node roles**: 3 dedicated master-eligible, data (hot/warm/cold), optional ingest/coordinating. Avoid all-in-one in prod.
- 3 AZs where possible, shard allocation awareness.

## Security (never disable in prod)
- **TLS** for HTTP (9200) and transport (9300); Kibana → ES over HTTPS; Filebeat with CA cert:
```yaml
output.elasticsearch:
  hosts: ["https://es01:9200"]
  username: "filebeat_writer"
  password: "${ES_PASSWORD}"
  ssl.certificate_authorities: ["/etc/filebeat/ca.crt"]
```
- **RBAC**: roles per team/index pattern; dedicated least-privilege writer role for shippers (`monitor`, `create_index`, `auto_configure`/`write` on `logs-*`); read-only role for viewers. Use API keys over passwords.
- SSO (SAML/OIDC/LDAP) for Kibana, audit logging, secrets in K8s Secrets / keystore (`filebeat keystore`).
- Network: ES not publicly exposed; security groups/NetworkPolicies; Kibana behind reverse proxy/VPN.

## Index Lifecycle Management (ILM) & retention
```json
PUT _ilm/policy/logs-policy
{ "policy": { "phases": {
  "hot":    { "actions": { "rollover": { "max_primary_shard_size": "50gb", "max_age": "1d" } } },
  "warm":   { "min_age": "7d",  "actions": { "forcemerge": { "max_num_segments": 1 }, "shrink": { "number_of_shards": 1 } } },
  "delete": { "min_age": "30d", "actions": { "delete": {} } }
}}}
```
Decide retention by compliance + cost (e.g. app logs 14–30d, audit 1y). Use **data streams** (Filebeat 8 default `filebeat-8.19.0`) rather than daily indices where possible.

## Backups – Snapshots
```bash
PUT _snapshot/s3_repo { "type": "s3", "settings": { "bucket": "elk-snapshots" } }
PUT _slm/policy/nightly
{ "schedule": "0 30 1 * * ?", "name": "<nightly-{now/d}>", "repository": "s3_repo",
  "config": { "indices": ["*"] }, "retention": { "expire_after": "30d", "min_count": 5, "max_count": 50 } }
```
**Test restores** regularly. Snapshots ≠ replicas.

## Scaling
**Elasticsearch**
- Add data nodes; keep shards 10–50GB; avoid thousands of tiny shards (shards per GB heap limits).
- Hot/warm/cold tiers; rollover; adequate replicas (1) for HA.
- Heap ≤ 50% RAM (max ~31GB); SSDs; watch disk watermarks (85/90/95%).
**Kibana**
- Run multiple instances behind LB with same `xpack.encryptedSavedObjects.encryptionKey` and `xpack.security.encryptionKey`.
**Filebeat**
- Scales with nodes (DaemonSet/agent per host).
- Tune `queue.mem.events`, `output.elasticsearch.bulk_max_size`, `worker`.
- Put Kafka/Logstash between Beats and ES for buffering and spikes.
- Use `drop_event`/exclude noisy logs at source.

## Operations checklist
- [ ] Monitoring of the ELK itself (stack monitoring/Metricbeat)
- [ ] Alerts: cluster health, disk, JVM heap, ingest lag, no-data
- [ ] ILM + retention defined
- [ ] Snapshots + restore tested
- [ ] TLS + RBAC + SSO
- [ ] Config in Git, deployed via CI/CD/Helm/Ansible
- [ ] Log fields standardised (ECS), JSON app logs
- [ ] PII masking/redaction policy
- [ ] Capacity plan: GB/day × retention × (1+replicas) × ~1.1–1.5 overhead
- [ ] Runbooks and upgrade plan (rolling upgrade, one minor version at a time)
