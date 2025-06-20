# Install Prometheus and Grafana 

## 1. Add Prometheus Community Helm repo
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

## 2. Create a namespace (optional but recommended)
kubectl create namespace monitoring

## 3. Install kube-prometheus-stack
helm install prometheus-stack prometheus-community/kube-prometheus-stack \
  --namespace monitoring

This stack includes:
    - Prometheus
    - Alertmanager
    - Grafana
    - Node Exporter
    - Kube-state-metrics

### Accessing Grafana 

```
kubectl port-forward -n monitoring svc/prometheus-stack-grafana 3000:80
```

Then open: http://localhost:3000

Default credentials:

User: admin

Password: You can get it with:

```
kubectl get secret -n monitoring prometheus-stack-grafana \
  -o jsonpath="{.data.admin-password}" | base64 --decode
```

### Dashboards
Grafana comes preloaded with dashboards for:

1. Kubernetes Nodes

2. Pods and Workloads

3. Cluster Resource Usage

## Add Prometheus Scraping for Node.js Services

```
metadata:
  annotations:
    prometheus.io/scrape: "true"
    prometheus.io/path: "/metrics"
    prometheus.io/port: "3000"
```

## Set Up Centralized Logging (ELK Stack via Helm)

### Add Bitnami Helm Repo

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```

### Deploy Elasticsearch
```
helm install elasticsearch bitnami/elasticsearch \
  --namespace logging --create-namespace
```

### Deploy Kibana

```
helm install kibana bitnami/kibana \
  --namespace logging \
  --set elasticsearch.hosts[0]=http://elasticsearch.logging.svc.cluster.local:9200
```


### Deploy Fluent Bit (Log Shipper)
```
helm install fluent-bit bitnami/fluent-bit \
  --namespace logging \
  --set backend.type=es \
  --set backend.es.host=elasticsearch.logging.svc.cluster.local \
  --set backend.es.port=9200
```

### Access Kibana (Port Forward)

```
kubectl port-forward svc/kibana -n logging 5601:5601
```

Open http://localhost:5601