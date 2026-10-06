# 01 – ELK Overview

## What is ELK?
| Component | Role |
|---|---|
| **E**lasticsearch | Stores and searches logs (distributed search engine) |
| **L**ogstash | Heavy-duty pipeline: parse, transform, route |
| **K**ibana | UI: Discover, dashboards, alerts |
| **Beats** (Filebeat) | Lightweight shippers on each source |

Together with Beats it is often called the **Elastic Stack**.

## Typical flow
```
Source → Filebeat → (Logstash optional) → Elasticsearch → Kibana
```

## Filebeat vs Logstash
| | Filebeat | Logstash |
|---|---|---|
| Footprint | Tiny (Go, ~50MB) | Heavy (JVM) |
| Where | On every node | Central tier |
| Strength | Reliable tailing, modules, K8s metadata | Complex parsing, enrichment, multi-output |
| Use when | Almost always as collector | Need grok-heavy transforms, cloud pulls (CloudWatch), buffering |

## When to use what
- Containers/VMs → **Filebeat** straight to Elasticsearch (simple).
- Complex transforms or non-file sources → add **Logstash**.
- High volume/spiky → add **Kafka** buffer.

## Versions
Lab uses **8.19**. In 8.x security (TLS + auth) is on by default; the lab disables it for simplicity – never do that in production (see [14](14-production-best-practices.md)).

See [diagrams/architecture.md](../diagrams/architecture.md).
