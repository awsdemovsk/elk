# 15 – Troubleshooting

## General pipeline checklist (follow in order)
1. **Source** – is the app writing? `docker logs nginx --tail 5` / `kubectl logs`
2. **Filebeat running?** – `docker ps` / `kubectl -n kube-system get pods -l app=filebeat`
3. **Filebeat config OK?** – `filebeat test config`
4. **Output reachable?** – `filebeat test output`
5. **Filebeat logs** – errors like `connection refused`, `no such host`, `403`, `x509`
6. **Elasticsearch** – `curl localhost:9200/_cluster/health?pretty`
7. **Index exists & count grows** – `curl localhost:9200/_cat/indices?v`
8. **Kibana Data View** exists and matches the index name
9. **Time range** covers the event time (and timezone is right)

---

## Issue 1 – Filebeat unable to reach Elasticsearch
**Symptom** (Filebeat pod logs):
```
Failed to connect to backoff(elasticsearch(http://host.docker.internal:9200)): dial tcp: lookup host.docker.internal: no such host
```
**Cause:** `host.docker.internal` is not reachable/resolvable from Kind nodes.
**Fix:** use the host machine IP.
```yaml
output.elasticsearch:
  hosts:
    - "192.168.x.x:9200"
```
```bash
ip -4 addr          # or: hostname -I
kubectl apply -f kubernetes/filebeat-daemonset/filebeat-daemonset.yaml
kubectl -n kube-system rollout restart ds/filebeat
kubectl -n kube-system logs ds/filebeat --tail=20
```
Also confirm Elasticsearch listens on all interfaces/published port (`-p 9200:9200`, not bound to 127.0.0.1 only) and the host firewall allows it (`sudo ufw status`).

---

## Issue 2 – Nginx logs not visible in Kibana
Checks:
- Filebeat running: `kubectl -n kube-system get pods` / `docker ps`
- Elasticsearch reachable: `curl localhost:9200`
- Index created:
```bash
curl localhost:9200/_cat/indices?v
```
- Data View created for the right pattern (e.g. `k8s-logs-*`)
- Nginx is actually logging: `docker logs nginx`; generate traffic with `curl localhost:8080`
- Filter in Discover isn't hiding it: try `container.name : "nginx"` then clear filters.

---

## Issue 3 – Filebeat collecting logs but Kibana shows no data
Verification:
1. Filebeat logs – look for `"published events"` / `events.acked` > 0 in monitoring lines:
   `kubectl -n kube-system logs ds/filebeat | grep -i -E "error|acked|monitoring"`
2. Elasticsearch indices – `curl localhost:9200/_cat/indices?v` (docs.count increasing?)
3. Create the **Data View** (`Stack Management → Data Views`).
4. **Refresh / widen time range** (Last 24h or 7d); check clock skew between host and containers.
5. If index exists but search is empty: `curl "localhost:9200/k8s-logs-*/_count?pretty"`.

---

## Issue 4 – `host.docker.internal` not working in Kind
**Explanation:** Kind nodes are themselves Docker containers (`kind-control-plane`, `kind-worker`) on the `kind` Docker network; Pods run inside those containers. Docker Desktop injects `host.docker.internal`, but Linux Docker (Ubuntu) and the Kind node containers don't resolve it by default.
**Solution:** use the actual host IP (LAN IP such as `192.168.x.x`, or the `kind` network gateway):
```bash
docker network inspect kind -f '{{range .IPAM.Config}}{{.Gateway}} {{end}}'   # e.g. 172.18.0.1
hostname -I
```
Test from inside the cluster:
```bash
kubectl run -it --rm t --image=curlimages/curl -- curl -s http://192.168.x.x:9200
```
Alternative: put Elasticsearch on the `kind` Docker network and use its container name/IP: `docker network connect kind elasticsearch`.

---

## Other common problems
| Problem | Fix |
|---|---|
| ES container exits: `max virtual memory areas vm.max_map_count too low` | `sudo sysctl -w vm.max_map_count=262144` |
| ES OOM / exit 137 | Lower `ES_JAVA_OPTS`, give Docker more RAM |
| Kibana "Kibana server is not ready yet" | ES not up yet; `docker logs kibana`; wait/check `ELASTICSEARCH_HOSTS` |
| 8.x `security_exception` / 401 | Security enabled: add username/password/API key, or (lab only) `xpack.security.enabled=false` |
| `x509: certificate signed by unknown authority` | Provide `ssl.certificate_authorities` |
| Filebeat `exiting: ... setup.template.name and setup.template.pattern have to be set if index name is modified` | Add `setup.template.*` or use data streams |
| `filestream input ... must have unique ID` | Give each input a unique `id` |
| Filebeat pod CrashLoop: `data path already locked` | Duplicate Filebeat using same data dir; check hostPath |
| Duplicate logs | Registry lost (non-persistent data dir) – mount hostPath for `/usr/share/filebeat/data` |
| Missing `kubernetes.*` fields | RBAC missing or `NODE_NAME` not set; check ClusterRoleBinding |
| Logs from only one node | DaemonSet not scheduled on control-plane (add tolerations) |
| Cluster yellow (single node) | `PUT _all/_settings {"index":{"number_of_replicas":0}}` |
| Indices read-only | Disk >95%: free space then `PUT _all/_settings {"index.blocks.read_only_allow_delete": null}` |
| 429 / bulk rejected | Backpressure – scale ES, lower Filebeat `bulk_max_size`/workers |

## Command quick reference
```bash
docker ps
kubectl get pods -A
kubectl get svc
kubectl logs <pod> [-c container] [-f] [--previous]
kubectl describe pod <pod>
kubectl exec -it <pod> -- sh
curl localhost:9200
curl localhost:5601/api/status
docker logs nginx
filebeat test config -c filebeat.yml
filebeat test output -c filebeat.yml
```
