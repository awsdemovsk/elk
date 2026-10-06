# 02 – Elasticsearch Basics (DevOps view)

Only what you need to operate it – not internals.

## Vocabulary
- **Index** – collection of documents (e.g. `k8s-logs-2026.10.06`).
- **Document** – one log event (JSON).
- **Shard** – piece of an index; **replica** – copy for HA.
- **Data stream / ILM** – time-based indices with automated rollover/retention.
- **Data View (Kibana)** – tells Kibana which indices to query.

## Run it (lab)
```bash
docker compose -f lab/docker-compose.yml up -d elasticsearch
curl localhost:9200
```

## Daily-use API calls
```bash
curl localhost:9200/_cluster/health?pretty
curl localhost:9200/_cat/nodes?v
curl localhost:9200/_cat/indices?v          # do indices exist? doc counts growing?
curl localhost:9200/_cat/shards?v
curl localhost:9200/_cat/allocation?v       # disk per node
curl "localhost:9200/k8s-logs-*/_search?size=1&pretty"
curl -X DELETE localhost:9200/old-index-name
```

## Cluster health
- **green** all shards allocated
- **yellow** replicas unassigned (normal on single node)
- **red** primary shard missing → data unavailable, investigate now

## Common ops gotchas
- Single node + replicas=1 → yellow. Set `number_of_replicas: 0` for lab.
- Disk >85% stops allocating; >95% makes indices read-only (flood stage).
- `vm.max_map_count` must be ≥ 262144 on Linux hosts:
  `sudo sysctl -w vm.max_map_count=262144`
- Heap: set to ≤50% RAM, ≤ ~31GB.
