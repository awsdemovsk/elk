# 07 – Docker Logs

## How Docker stores logs
Default logging driver is `json-file`. Each container's stdout/stderr is written to:
```
/var/lib/docker/containers/<container-id>/<container-id>-json.log
```
Each line is JSON: `{"log":"...","stream":"stdout","time":"..."}`.

## Inspect
```bash
docker ps
docker logs nginx --tail 20 -f
docker inspect nginx | grep -i logpath
docker inspect -f '{{.LogPath}}' nginx
docker inspect -f '{{.HostConfig.LogConfig}}' nginx
sudo ls -lh /var/lib/docker/containers/$(docker inspect -f '{{.Id}}' nginx)/
```

## Log rotation (avoid filling the disk!)
`/etc/docker/daemon.json`:
```json
{ "log-driver": "json-file",
  "log-opts": { "max-size": "50m", "max-file": "3" } }
```
`sudo systemctl restart docker` (applies to new containers).

## How Filebeat collects
1. Read `/var/lib/docker/containers/*/*.log` with the `container` parser (decodes JSON, sets `stream`, `message`).
2. `add_docker_metadata` adds container name/image/labels (needs `/var/run/docker.sock`).
3. Ship to Elasticsearch.

Configs:
- Host install: [`filebeat-docker-container.yml`](../filebeat/docker/filebeat-docker-container.yml)
- Filebeat as container with autodiscover: [`filebeat-docker-autodiscover.yml`](../filebeat/docker/filebeat-docker-autodiscover.yml) + [`docker-compose-filebeat.yml`](../filebeat/docker/docker-compose-filebeat.yml)

Architecture: see [diagrams](../diagrams/architecture.md#2-docker-logging).

## Alternatives
- `--log-driver=gelf` or `syslog` to send directly (loses local `docker logs` on older versions).
- Filebeat is preferred: files stay local, backpressure-safe.

## Troubleshooting
| Symptom | Check |
|---|---|
| No docker logs in ES | Filebeat mounts `/var/lib/docker/containers` read-only? Running as root? |
| No container.name field | docker.sock mounted? |
| Disk full | No rotation configured |
| Filebeat "permission denied" | Run as `user: root` |
