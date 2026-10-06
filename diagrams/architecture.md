# Architecture Diagrams (Mermaid – renders natively on GitHub)

## 1. ELK Core
```mermaid
flowchart LR
  A[Servers / Apps / Containers] --> B[Filebeat]
  B -->|optional| L[Logstash]
  B --> E[(Elasticsearch)]
  L --> E
  E --> K[Kibana Dashboards & Alerts]
```

## 2. Docker Logging
```mermaid
flowchart LR
  C1[nginx container] -->|stdout/stderr| D[Docker json-file driver]
  D --> F["/var/lib/docker/containers/<id>/<id>-json.log"]
  F --> FB[Filebeat]
  FB --> ES[(Elasticsearch)] --> KB[Kibana]
```

## 3. Kubernetes (Filebeat DaemonSet)
```mermaid
flowchart TB
  subgraph Node1
    P1[Pods] --> L1["/var/log/pods → /var/log/containers"]
    L1 --> FB1[Filebeat Pod]
  end
  subgraph Node2
    P2[Pods] --> L2["/var/log/pods → /var/log/containers"]
    L2 --> FB2[Filebeat Pod]
  end
  FB1 -->|add_kubernetes_metadata| API[K8s API]
  FB2 --> API
  FB1 --> ES[(Elasticsearch)]
  FB2 --> ES --> KB[Kibana]
```

## 4. Kind Lab Network (why host.docker.internal fails)
```mermaid
flowchart LR
  subgraph Host[Ubuntu laptop 192.168.x.x]
    ES[(Elasticsearch :9200)]
    KB[Kibana :5601]
    subgraph Docker network kind
      CP[kind-control-plane]
      W[kind-worker → Filebeat pod]
    end
  end
  W -- "host IP:9200 ✔" --> ES
  W -. "host.docker.internal ✘" .-> ES
```

## 5. AWS
```mermaid
flowchart LR
  EC2[EC2 + Filebeat] --> ES[(Elasticsearch)]
  EKS[EKS + Filebeat DaemonSet] --> ES
  ALB[ALB logs → S3 → SQS] --> FBA[Filebeat aws module] --> ES
  CW[CloudWatch Logs] --> LS[Logstash] --> ES
  ES --> KB[Kibana]
```

## 6. Azure
```mermaid
flowchart LR
  R[AKS / App Service / VM / Container Apps] --> DS[Diagnostic Settings]
  DS --> LAW[Log Analytics Workspace]
  DS --> EH[Event Hub]
  EH --> FB[Filebeat azure module / Logstash] --> ES[(Elasticsearch)] --> KB[Kibana]
```

## 7. GCP
```mermaid
flowchart LR
  S[GKE / GCE / Cloud Run] --> CL[Cloud Logging]
  CL --> LR[Log Router Sink] --> PS[Pub/Sub]
  PS --> FB[Filebeat gcp module / Logstash] --> ES[(Elasticsearch)] --> KB[Kibana]
```

## 8. Production
```mermaid
flowchart LR
  Beats[Filebeat fleet] --> LB[Logstash / Kafka buffer]
  LB --> M[3 Master nodes]
  LB --> H[Hot data nodes] --> W[Warm] --> C[Cold/Frozen → S3 snapshots]
  H --> KBLB[Kibana x2 behind LB]
```
