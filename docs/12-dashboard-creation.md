# 12 – Dashboard Creation (Kibana Lens)

General steps: Dashboards → Create → **Create visualization** (Lens) → choose Data View → drag fields → Save → add to dashboard → set auto-refresh and time range.

Screenshots go in [`/screenshots`](../screenshots) (e.g. `nginx-dashboard.png`).

## 1. Nginx Dashboard
Data view: `docker-logs-*` / `k8s-logs-*` / `nginx-logs-*` (parsed via module/pipeline).
| Panel | Lens setup |
|---|---|
| **Request count** | Metric → Count of records; or Bar over `@timestamp` |
| **Status codes** | Donut/Bar → Top values of `http.response.status_code` |
| **Top URLs** | Horizontal bar → Top 10 `url.original` by count |
| **Top client IPs** | Table → Top 10 `source.ip` |
Extra: 5xx rate (`http.response.status_code >= 500` filter), bytes sent sum.

## 2. Kubernetes Dashboard
Data view: `k8s-logs-*`
| Panel | Setup |
|---|---|
| **Pod restarts** | Needs Metricbeat/kube-state-metrics (`kubernetes.container.status.restarts`) – max by pod. Log-only proxy: count of "Started container"/"Back-off" events |
| **Namespace activity** | Bar → Top `kubernetes.namespace` by count |
| **Log volume** | Stacked area over time, break down by `kubernetes.namespace` |
| **Noisiest pods** | Table → Top `kubernetes.pod.name` |
| **Errors by namespace** | Filter `message : "error"` |

## 3. Linux Dashboard
Data view: `linux-logs-*`
| Panel | Setup |
|---|---|
| **SSH failures** | Filter `message : "Failed password"` or `event.outcome: failure and event.category: authentication`; bar over time |
| **System errors** | Filter `message : (error or fail* or critical)`; top `process.name` |
| **Auth events** | Count by `log_type: auth`, split by `user.name` / `source.ip` |
| **Top attacker IPs** | Table of `source.ip` for failed logins |

## 4. Application Dashboard
| Panel | Setup |
|---|---|
| **Error logs** | Count where `log.level: error` (or message contains ERROR) |
| **Warning logs** | Same for warn |
| **Response times** | Average/percentiles of `event.duration` or parsed `response_time` field (requires JSON logs or dissect) |
| **Error ratio** | Formula: `count(kql='log.level:error') / count()` |

> Tip: log in **JSON** from apps so fields appear without parsing.

## Practice data
```bash
kubectl apply -f kubernetes/examples/log-generator.yaml
for i in $(seq 1 300); do curl -s -o /dev/null localhost:8080/$((i%7)); done
```

## Import prebuilt module dashboards
```bash
filebeat setup --dashboards -E setup.kibana.host=localhost:5601
```
Export your own: Stack Management → Saved Objects → Export (`.ndjson`) and commit it to the repo under `projects/`.
