# 06 – Nginx Logs

## Containerised nginx (lab)
```bash
docker run -d --name nginx -p 8080:80 nginx:stable
curl localhost:8080
docker logs nginx
```
Nginx image symlinks `access.log → /dev/stdout` and `error.log → /dev/stderr`, so Docker captures them and Filebeat reads the Docker log file.

## Host nginx
Logs live in `/var/log/nginx/`. Use [`filebeat-nginx.yml`](../filebeat/linux/filebeat-nginx.yml) or the module.

## Default access-log line
```
172.17.0.1 - - [06/Oct/2026:10:00:01 +0000] "GET / HTTP/1.1" 200 615 "-" "curl/8.5.0"
```

## Parse into fields
- Easiest: nginx **module** (ECS fields: `source.ip`, `url.original`, `http.response.status_code`, `http.request.method`).
- Container logs: use an ingest pipeline with grok:
```json
PUT _ingest/pipeline/nginx-access
{
  "processors": [
    { "grok": { "field": "message",
      "patterns": ["%{IPORHOST:source.ip} - %{DATA:user} \\[%{HTTPDATE:ts}\\] \"%{WORD:http.request.method} %{DATA:url.original} HTTP/%{NUMBER:http.version}\" %{NUMBER:http.response.status_code:int} %{NUMBER:http.response.body.bytes:int}"],
      "ignore_failure": true } }
  ]
}
```
Then in Filebeat: `output.elasticsearch.pipeline: nginx-access`.

## Generate traffic for dashboards
```bash
for i in $(seq 1 200); do curl -s -o /dev/null localhost:8080/; curl -s -o /dev/null localhost:8080/missing-$((i%5)); done
```
(produces 200s and 404s)

## Verify end-to-end
```bash
docker logs nginx --tail 5
curl "localhost:9200/_cat/indices?v"
```
Then Kibana → Discover → `container.name : "nginx"`.
