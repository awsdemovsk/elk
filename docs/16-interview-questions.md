# 16 – Interview Questions (DevOps ELK)

**1. What is ELK?**
Elasticsearch (search/store), Logstash (processing), Kibana (visualisation); usually with Beats for collection. Used for centralised logging, search, dashboards and alerting.

**2. Difference between Logstash and Filebeat?**
Filebeat is a lightweight Go shipper on each host that tails files reliably. Logstash is a JVM pipeline for heavy parsing/enrichment/routing. Typical: Filebeat everywhere, Logstash centrally only if needed.

**3. Why use Filebeat in Kubernetes?**
Low overhead, runs per node as a DaemonSet, reads container logs without changing apps, adds K8s metadata, and tracks file offsets.

**4. How does a Filebeat DaemonSet work?**
One pod per node mounts host `/var/log` (and container paths), reads `/var/log/containers/*.log` symlinks, parses CRI/JSON format, enriches with `add_kubernetes_metadata` (via RBAC to the API server), and ships to Elasticsearch/Logstash.

**5. How do you troubleshoot missing logs?**
Follow the chain: app writes? → Filebeat running and config valid? → `filebeat test output` → Filebeat logs → ES reachable and index created (`_cat/indices`) → Data View correct → time range. (See [15](15-troubleshooting.md).)

**6. How do you scale Elasticsearch?**
Add data nodes, dedicated masters, hot/warm/cold tiers, right-size shards (10–50GB), ILM rollover, replicas, buffer ingestion with Kafka/Logstash.

**7. How do you secure ELK?**
TLS (HTTP + transport), authentication (native/SSO), RBAC with least privilege, API keys for shippers, network isolation, secrets management, audit logging, patching.

**8. Why does Kibana show no data even though Filebeat runs?**
Missing Data View, wrong index pattern, time range, ES unreachable, or events not acked (check Filebeat logs).

**9. What is ILM?**
Automates index lifecycle: hot → warm → cold → delete, with rollover by size/age to control cost and retention.

**10. What causes yellow/red cluster status?**
Yellow: unassigned replicas. Red: unassigned primaries – node down, disk watermark, corrupted shard.

**11. Data stream vs daily indices?**
Data streams give append-only time series with automatic rollover/ILM; simpler management than manual date-named indices.

**12. How do you handle log volume spikes?**
Queue/buffer (Kafka), backpressure-aware Filebeat, drop noisy logs at source, scale ingest, ILM retention.

**13. How do you collect logs from EKS/AKS/GKE?**
Pod logs via Filebeat DaemonSet; control-plane/managed logs via cloud service (CloudWatch / Diagnostic Settings → Event Hub / Log Router → Pub/Sub) then Filebeat/Logstash.

**14. How do you avoid duplicates/loss in Filebeat?**
Persistent registry (hostPath/PVC), at-least-once delivery, use document IDs/fingerprint if dedupe is required.

**15. Why does `host.docker.internal` fail in Kind?**
Kind nodes are Docker containers; the name isn't resolved on Linux. Use the host IP or the kind network gateway.

**16. How do you back up Elasticsearch?**
Snapshot repository (S3/Blob/GCS) + SLM policy; test restores.

**17. How do you monitor ELK itself?**
Stack Monitoring/Metricbeat; alerts on health, disk, heap, ingest rate/lag, no-data.

**18. Where are Docker/Kubernetes logs stored?**
Docker: `/var/lib/docker/containers/<id>/<id>-json.log`. K8s: `/var/log/pods/...` with symlinks in `/var/log/containers/`.

**19. What is ECS?**
Elastic Common Schema – standard field names (`source.ip`, `http.response.status_code`) so dashboards/alerts work across sources.

**20. Elastic Agent vs Filebeat?**
Agent (Fleet-managed) unifies logs/metrics/security with central policy management; Filebeat is single-purpose and file-config driven.

## Scenario questions
- *Logs stopped arriving from one node* → check Filebeat pod on that node, taints/tolerations, disk pressure, registry lock.
- *Index growing too fast* → find top namespaces/pods by volume, drop debug logs, shorten ILM retention.
- *Need to hide PII* → `drop_fields`/redaction at Filebeat or ingest pipeline; restrict via RBAC/field-level security.
