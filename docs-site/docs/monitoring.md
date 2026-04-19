# Monitoring

## Stack

- **Prometheus**: Metrics collection via kube-prometheus-stack Helm chart
- **Grafana**: Dashboards and alerting — accessible at `grafana.example.com`
- **Loki**: Log aggregation — queried through Grafana datasource
- **Hubble**: Cilium network observability — accessible at `hubble.example.com`

## Key Dashboards

- Node resource usage (CPU, memory, disk)
- Pod restart and crash tracking
- Traefik request rates and error codes
- CrowdSec ban activity
