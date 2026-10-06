# 17 – 30-Day Study Plan

## Week 1 – ELK Fundamentals (Days 1–7)
- D1: Read [01](01-elk-overview.md); run `lab/docker-compose.yml`.
- D2: Elasticsearch APIs `_cat`, `_cluster/health` ([02](02-elasticsearch-basics.md)).
- D3: Kibana Discover, KQL, Data Views ([03](03-kibana-basics.md)).
- D4: Filebeat install, config, `test config/output` ([04](04-filebeat-basics.md)).
- D5: Ship a sample log file; see it in Kibana.
- D6: Modules + `filebeat setup`.
- D7: Review + 10 interview questions.

## Week 2 – Docker + Linux Logs (Days 8–14)
- D8: Linux logs ([05](05-linux-server-logs.md)); ship syslog/auth.
- D9: Generate SSH failures; search them.
- D10: Nginx logs ([06](06-nginx-logs.md)); traffic generator.
- D11: Docker log internals ([07](07-docker-logs.md)); `docker inspect`.
- D12: Filebeat autodiscover for containers.
- D13: Parse nginx with ingest pipeline.
- D14: Mini project: **Nginx Observability Dashboard**.

## Week 3 – Kubernetes + EKS + AKS Logs (Days 15–21)
- D15: Create Kind cluster ([08](08-kubernetes-logs.md)).
- D16: Deploy Filebeat DaemonSet; fix connectivity (host IP).
- D17: Metadata, RBAC, noisy-log dropping.
- D18: Autodiscover + annotations.
- D19: AWS/EKS logging ([09](09-aws-logs.md)).
- D20: Azure/AKS logging ([10](10-azure-logs.md)); skim GCP ([11](11-gcp-logs.md)).
- D21: Mini project: **Kubernetes Centralized Logging**.

## Week 4 – Dashboards + Alerting + Production (Days 22–30)
- D22–23: Build 4 dashboards ([12](12-dashboard-creation.md)).
- D24–25: Alerts: error rate, pod crash, SSH ([13](13-alerting.md)); Slack/email connector.
- D26: ILM, retention, snapshots ([14](14-production-best-practices.md)).
- D27: Security: TLS, RBAC, API keys.
- D28: Scaling and topology design (draw diagram).
- D29: Troubleshooting drills ([15](15-troubleshooting.md)): break things deliberately and fix.
- D30: Mock interview ([16](16-interview-questions.md)); pick a [project](../projects/README.md) for your portfolio.
