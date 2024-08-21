# Redis High Availability with Prometheus and Grafana Monitoring

This setup deploys Redis in a highly available configuration on Kubernetes, with metrics exposed to Prometheus and visualized in Grafana.

## Prerequisites

- **Kubernetes Cluster**: Ensure you have access to a Kubernetes cluster.
- **kubectl**: Kubernetes command-line tool installed and configured.
- **Helm**: Helm package manager installed.

## Steps to Deploy

### 1. Deploy Redis with Metrics Exporter

Apply the Redis StatefulSet and Service manifests to deploy Redis with metrics exporting enabled.

```bash
kubectl apply -f redis-statefulset.yaml
kubectl apply -f redis-service.yaml
```

Another manifest is a cronjob that will insert keys to redis every 1 min (So metrics will keep genereting )
```bash
kubectl apply -f redis-cronjob.yaml
```


### 2. Install Prometheus and Grafana using Helm

#### Add Helm Repositories

Add the required Helm chart repositories:

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

#### Install Prometheus

Install Prometheus in the `monitoring` namespace:

```bash
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring \
  --create-namespace
```

#### Install Grafana

Install Grafana in the `monitoring` namespace and expose it via a LoadBalancer:

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set adminUser=admin \
  --set adminPassword=admin \
  --set service.type=LoadBalancer
```

### 3. Accessing Grafana

#### Retrieve Grafana Service Details

To access Grafana, :

```bash
kubectl port-forward -n monitoring svc/grafana 3000:80
```

- **Default Credentials**: 
  - **Username**: `admin`
  - **Password**: `admin`

### 4. Configure Grafana

#### Add Prometheus as a Data Source

1. Navigate to **Configuration > Data Sources** in Grafana.
2. Add a new Prometheus data source with the following details:
   - **Name**: `Prometheus`
   - **Type**: `Prometheus`
   - **URL**: `http://prometheus-server.monitoring.svc.cluster.local`
3. Click **Save & Test** to confirm the connection.

#### Import Redis Dashboard

1. Navigate to **Create > Import**.
2. Import a Redis dashboard using Grafana's built-in dashboards:
   - **Dashboard ID**: `763` (Official Redis dashboard)
   - **Load** and select the Prometheus data source.
3. Another option is using the general [grafana marketplace](https://grafana.com/grafana/dashboards/?search=redis&dataSource=prometheus)

