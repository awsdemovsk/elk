# 03 – Kibana Basics

Open <http://localhost:5601>.

## Key areas
- **Discover** – search raw logs (KQL).
- **Dashboards / Lens** – visualizations.
- **Stack Management → Data Views** – map index patterns.
- **Stack Management → Rules** – alerting.
- **Observability → Logs / Stream** – log tailing.
- **Dev Tools** – run Elasticsearch API calls.

## Create a Data View (needed before logs appear!)
1. Stack Management → Data Views → Create.
2. Name `k8s-logs`, pattern `k8s-logs-*`.
3. Timestamp field `@timestamp`.
4. Open Discover, choose the view, widen the **time range** (e.g. Last 24 hours).

## KQL cheat sheet
```
kubernetes.namespace : "default"
http.response.status_code >= 500
message : "error" and not message : "healthcheck"
container.name : "nginx" and log.level : "error"
source.ip : "10.0.0.0/8"
```

## Troubleshooting Kibana
```bash
curl localhost:5601/api/status
docker logs kibana --tail 50
```
Kibana "server not ready" → Elasticsearch unreachable or (8.x) enrollment/auth settings wrong.
