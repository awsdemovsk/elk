# 13 – Monitoring & Alerting

Kibana: **Stack Management → Rules → Create rule**. (Some connectors/features need a paid or trial licence – check your subscription; basic tier supports core rule types with limited connectors, e.g. server log/index.)

## Rule types you will use
- **Log threshold** (Observability) – count of matching logs.
- **Elasticsearch query** – KQL/DSL, fires if hits > N.
- **Custom threshold** – metrics/logs ratio.
- **Metric threshold / Inventory** – CPU, memory, disk (needs Metricbeat/Elastic Agent).

## Connectors
**Email** (SMTP host/port/credentials), **Slack** (incoming webhook URL), plus index, webhook, PagerDuty, Teams.
Slack: create an Incoming Webhook in your workspace → paste into Slack connector.
Email: Gmail/SES/Office365 SMTP; use app passwords/secrets.

## Alert recipes
| Alert | Condition |
|---|---|
| **High error rate** | Log threshold: `log.level:error` count > 50 per 5 min, group by `kubernetes.namespace` |
| **Pod crash** | ES query on `message : ("CrashLoopBackOff" or "OOMKilled" or "Back-off restarting")`, or metric: `kubernetes.container.status.restarts` increases |
| **SSH brute force** | `Failed password` count > 20 per 5 min grouped by `source.ip` |
| **No data** | Rule "no data" on index → shipper broken |
| **Disk usage** | Metric threshold `system.filesystem.used.pct > 0.85` (Metricbeat `system`) |
| **CPU** | `system.cpu.total.norm.pct > 0.9` for 5 min |
| **Memory** | `system.memory.actual.used.pct > 0.9` |
| **ES cluster** | Cluster health ≠ green; node disk >85% |

## Message template example (Slack)
```
🚨 {{rule.name}} fired
Namespace: {{context.group}}
Matches: {{context.matchingDocuments}}
{{context.reason}}
```

## Good alerting hygiene
- Alert on symptoms (error rate, saturation) not every error.
- Group by key (service/namespace) and set throttle (e.g. notify every 30m).
- Include runbook link in message.
- Test connectors with the "Test" button.
- Keep a no-data alert on your own pipeline.
