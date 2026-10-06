# DevOps Projects

Each project: goal → architecture → steps → deliverables → interview talking points. Add your own configs/screenshots in a subfolder per project.

## 1. Kubernetes Centralized Logging
**Goal:** All pod logs from a Kind cluster searchable in Kibana.
**Architecture:** Pods → Filebeat DaemonSet → Elasticsearch → Kibana.
**Steps**
```bash
docker compose -f lab/docker-compose.yml up -d
kind create cluster --name elk-lab --config kubernetes/kind/kind-config.yaml
# set HOST_IP then
kubectl apply -f kubernetes/filebeat-daemonset/filebeat-daemonset.yaml
kubectl apply -f kubernetes/examples/
```
**Deliverables:** Data view, namespace/pod dashboard, error alert.
**Talk about:** DaemonSet, RBAC, host IP vs host.docker.internal, metadata enrichment.

## 2. EKS Log Monitoring Platform
**Architecture:** EKS → Filebeat DaemonSet (pods) + CloudWatch control-plane logs → Logstash → ES (on EC2/Elastic Cloud) → Kibana.
**Steps:** Terraform/eksctl cluster; enable control-plane logging; IRSA for Logstash; ILM; alerts.
**Talk about:** cost vs CloudWatch, IRSA, audit log analysis.

## 3. AKS Monitoring Platform
**Architecture:** AKS → Filebeat DaemonSet; Diagnostic Settings → Event Hub → Filebeat azure module → ES.
**Steps:** create Event Hub, diagnostic setting, consumer group, checkpoint storage.
**Talk about:** LAW vs ELK, managed identity, category selection.

## 4. Multi-Cloud Log Aggregation
**Architecture:** AWS (S3/SQS, CloudWatch), Azure (Event Hub), GCP (Pub/Sub) → Filebeat/Logstash → one ES cluster with `cloud.provider` field and per-cloud ILM → single Kibana with spaces.
**Talk about:** ECS normalisation, egress cost, latency, RBAC per cloud/team.

## 5. Nginx Observability Dashboard
**Goal:** requests, status codes, top URLs/IPs, 5xx rate, latency.
**Steps:** nginx container + traffic generator ([06](../docs/06-nginx-logs.md)); grok pipeline; Lens dashboard ([12](../docs/12-dashboard-creation.md)); 5xx alert.

## 6. DevOps Monitoring Platform
**Goal:** logs + metrics + alerts for CI/CD, servers and K8s in one place.
**Architecture:** Filebeat + Metricbeat → ES; Jenkins/GitHub Actions logs; Kibana dashboards; Slack alerts; ILM; snapshots.

## 7. Security Audit Dashboard
**Sources:** `auth.log`/`secure`, K8s audit logs, CloudTrail/Azure Activity/GCP audit.
**Panels:** failed logins, sudo usage, top source IPs/geo (GeoIP processor), privileged API calls, RBAC changes.
**Alerts:** brute-force, root login, new IAM/role binding.

## 8. Cloud Cost Monitoring Dashboard
**Sources:** AWS CUR/Cost Explorer export (S3), Azure Cost Management export, GCP Billing export (BigQuery→file).
**Method:** Logstash/Filebeat S3 input or scripted ingestion to ES; dashboards by service/team/tag/day; alert on budget threshold spikes.
**Talk about:** tagging strategy, anomaly detection.

---
## Portfolio tips
- Commit configs + a short README + architecture diagram + screenshots per project.
- Keep a "What broke and how I fixed it" section (see [troubleshooting](../docs/15-troubleshooting.md)).
