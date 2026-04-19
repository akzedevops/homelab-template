# Services

All services are exposed via Traefik IngressRoutes under `*.example.com` with Let's Encrypt TLS.

| Service | URL | Description | Stack |
|---------|-----|-------------|-------|
| ArgoCD | `argocd.example.com` | GitOps continuous deployment | ArgoCD |
| Grafana | `grafana.example.com` | Monitoring dashboards | Grafana + Prometheus |
| Gitea | `gitea.example.com` | Self-hosted Git hosting | Gitea + PostgreSQL |
| Authentik | `auth.example.com` | SSO / Identity provider | Authentik + PostgreSQL + Redis |
| Traefik | `traefik.example.com` | Reverse proxy dashboard | Traefik |
| Nextcloud | `nextcloud.example.com` | File storage & sync | Nextcloud + PostgreSQL + Redis |
| Jellyfin | `jellyfin.example.com` | Media streaming server | Jellyfin |
| Vaultwarden | `vault.example.com` | Password manager (Bitwarden-compatible) | Vaultwarden + SQLite |
| n8n | `n8n.example.com` | Workflow automation | n8n + PostgreSQL |
| Homepage | `home.example.com` | Service dashboard | Homepage |
| Hubble | `hubble.example.com` | Cilium network observability UI | Hubble UI |
| Docs | `docs.example.com` | Homelab documentation (MkDocs) | MkDocs Material |
| Prometheus | — (internal) | Metrics collection & alerting | kube-prometheus-stack |
| Loki | — (internal) | Log aggregation | Loki |

!!! note
    Prometheus and Loki are internal-only services accessed through Grafana datasources. All other services have public IngressRoutes protected by CrowdSec and (where applicable) Authentik SSO.
