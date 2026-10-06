# 04 – Filebeat Basics

## How it works
Inputs (harvesters) tail files → processors enrich → output ships. A **registry** (`data/registry`) tracks offsets so restarts don't duplicate or lose lines.

## Minimal config
```yaml
filebeat.inputs:
  - type: filestream
    id: my-app
    paths: [/var/log/myapp/*.log]
output.elasticsearch:
  hosts: ["http://localhost:9200"]
```
> `filestream` replaces the deprecated `log` input in 8.x. Every filestream input needs a unique `id`.

## Essential commands
```bash
filebeat test config -c filebeat.yml     # syntax check
filebeat test output -c filebeat.yml     # connectivity to Elasticsearch
filebeat -e -c filebeat.yml              # run in foreground, logs to stderr
filebeat modules list
filebeat modules enable nginx system
filebeat setup -e                        # loads templates + prebuilt dashboards
```

## Useful processors
```yaml
processors:
  - add_host_metadata: ~
  - add_docker_metadata: ~
  - add_kubernetes_metadata: ~
  - drop_event:
      when.contains.message: "healthcheck"
  - drop_fields:
      fields: ["agent.ephemeral_id"]
      ignore_missing: true
  - dissect:
      tokenizer: "%{ts} %{level} %{msg}"
      field: message
      target_prefix: app
```

## Index naming (custom index)
```yaml
setup.ilm.enabled: false
setup.template.name: "myapp"
setup.template.pattern: "myapp-*"
output.elasticsearch:
  index: "myapp-%{+yyyy.MM.dd}"
```
(Custom `index` requires the `setup.template.*` lines, otherwise Filebeat refuses to start.)

## Multiline (stack traces)
```yaml
multiline.type: pattern
multiline.pattern: '^[[:space:]]+(at|\.{3})|^Caused by:'
multiline.negate: false
multiline.match: after
```

## Filebeat vs Elastic Agent
Elastic Agent + Fleet is the newer central-management option. Filebeat remains simpler for config-as-code (GitOps) and is what this handbook uses.
