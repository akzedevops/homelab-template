# Homelab Template

Production-grade self-hosted homelab on DigitalOcean Kubernetes (DOKS). Fully GitOps via ArgoCD, zero-trust networking via Cloudflare Tunnel, comprehensive monitoring with Telegram alerts.

## What's Included

- **34 ArgoCD apps** deployed via App of Apps pattern
- **13 public services** behind Cloudflare Tunnel (no exposed origin IP)
- **SSO** via Authentik (OIDC for ArgoCD, Grafana, Gitea, Traefik, Hubble)
- **Security**: CrowdSec (L7 WAF) + Falco (runtime) + Kubescape (compliance)
- **Monitoring**: Prometheus + Grafana + Loki + Alertmanager → Telegram
- **Auto-updates**: Renovate CronJob (weekly)
- **Docs site**: MkDocs Material served via nginx

## Infrastructure

| Component | Details |
|-----------|---------|
| Cluster | DOKS 1.35.1, 3× `s-4vcpu-8gb`, nyc1 |
| Ingress | Traefik v3 (2 replicas) |
| Tunnel | Cloudflare Tunnel (2 replicas, no LoadBalancer) |
| TLS | Let's Encrypt wildcard via cert-manager + Cloudflare DNS-01 |
| DNS | Cloudflare + External-DNS (auto-creates CNAME → tunnel) |
| CNI | Cilium + Hubble |
| Storage | DO Block Storage + DO Spaces (CSI-S3) |
| Secrets | Sealed Secrets (encrypted in Git) |
| CI/CD | Gitea Actions (YAML lint + secret scan) |

## Services

| Service | Description |
|---------|-------------|
| ArgoCD | GitOps deployment |
| Grafana | Monitoring (6 custom dashboards) |
| Prometheus + Loki | Metrics + logs |
| Gitea | Git hosting + CI runner |
| Authentik | SSO / Identity provider |
| Nextcloud | File storage & sync |
| Jellyfin | Media server |
| Vaultwarden | Password manager |
| n8n | Workflow automation |
| Paperless-ngx | Document management + OCR |
| Homepage | Dashboard |
| Docs | MkDocs documentation site |
| Goldilocks | Resource recommendations |

## Security Stack

```
User → Cloudflare Edge (DDoS) → Tunnel → Traefik → CrowdSec Bouncer → Service
                                                         ↓
                                              Ban IP if malicious
```

- **CrowdSec**: LAPI + agents parsing Traefik logs, auto-bans attackers
- **Falco**: Runtime syscall monitoring via eBPF
- **Kubescape**: Vulnerability + compliance scanning
- **Sealed Secrets**: Zero plain-text secrets in Git

## Repo Structure

```
argocd/                  # 34 ArgoCD Application manifests
platform/
  traefik/               # Traefik Helm values (OIDC plugin, metrics)
  monitoring/            # kube-prometheus-stack + dashboards + alerting
  cloudflared/           # Cloudflare Tunnel deployment
  ingress/               # All IngressRoutes (CrowdSec middleware on each)
  sealed-secrets/        # Encrypted SealedSecret manifests
  backup/                # Weekly DB backup CronJob → Spaces
  authentik/             # Authentik Helm values
  gitea/                 # Gitea Helm values
  ...
apps/
  jellyfin/              # Jellyfin deployment
  nextcloud/             # Nextcloud Helm values
  n8n/                   # n8n Helm values
  paperless/             # Paperless-ngx deployment
  docs/                  # MkDocs docs site (nginx + Spaces)
docs-site/               # MkDocs source files
.gitea/workflows/        # CI pipelines
renovate.json            # Auto-update config
```

## Setup

1. Provision DOKS cluster on DigitalOcean
2. Install ArgoCD: `kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml`
3. Create required sealed secrets (Cloudflare token, Spaces keys, Telegram bot token, etc.)
4. Apply the root app: `kubectl apply -f argocd/homelab.yaml`
5. Everything else deploys automatically via App of Apps

## Quirks & Gotchas

Things that will bite you if you don't know about them:

| Quirk | Details |
|-------|---------|
| **RWO PVC + RollingUpdate = deadlock** | Deployments with ReadWriteOnce PVCs MUST use `strategy.type: Recreate`. Otherwise the new pod can't mount the volume while the old one holds it. |
| **Bitnami PostgreSQL secrets** | Bitnami PG charts auto-generate passwords on first install. On subsequent syncs, ArgoCD sees drift. Use `existingSecret` or add `ignoreDifferences` on `/data`. |
| **CrowdSec bouncer per-route, NOT global** | Never add the bouncer as an entrypoint-level middleware. If the bouncer pod dies, ALL routes return 500. Use per-IngressRoute ForwardAuth instead. |
| **CrowdSec LAPI CPU throttling** | LAPI needs ≥500m CPU limit. At 200m it gets throttled → bouncer timeouts → 403 on all services. |
| **CrowdSec bouncer registration** | If LAPI restarts, bouncer loses registration. Fix: set `BOUNCER_KEY_<name>` env var on LAPI so it auto-registers on startup. |
| **Cloudflared restarts** | cloudflared exits when all 4 QUIC connections drop simultaneously. Add `--retries 10` to prevent this. |
| **Cloudflare Tunnel → Traefik port** | Route to `:443` (websecure), NOT `:80`. Port 80 has an HTTP→HTTPS redirect IngressRoute that causes 301 loops. |
| **External-DNS with tunnel** | Annotations must point to `<tunnel-uuid>.cfargotunnel.com` (CNAME), not an IP. Add `--cloudflare-proxied` flag. |
| **Traefik OIDC plugin name** | Must be `traefik-oidc-auth`, not `oidc-auth`. The plugin registry name matters. |
| **IngressRoutes outside traefik namespace** | Use `tls: {}` (empty object), NOT `tls: { secretName: ... }`. The wildcard cert is the default. |
| **Grafana 13** | Set `unified_storage.enabled: false` or dashboards break. |
| **Loki SingleBinary** | Occasional `scheduler_processor` EOF errors are harmless (memory-related). |
| **ArgoCD shared resources** | If two ArgoCD apps manage the same resource, you get `resourceVersion` conflicts. One app must own each resource. |
| **Alertmanager CronJob alerts** | Completed CronJob pods (Descheduler, Renovate) trigger `PodNotReady`. Exclude with: `unless on(pod,namespace) kube_pod_status_phase{phase="Succeeded"}` |
| **Traefik forwardedHeaders** | With Cloudflare Tunnel, Traefik sees cloudflared pod IPs. Add `trustedIPs: [10.0.0.0/8]` on websecure entrypoint so it reads real IPs from X-Forwarded-For. |
| **Sealed Secrets controller** | Lives in `kube-system`. When sealing: `--controller-name=sealed-secrets --controller-namespace=kube-system`. |
| **Git remote tokens** | Never change the git remote URL if it has an embedded token. Just push/pull. |

## Cost

| Item | Monthly |
|------|---------|
| 3× s-4vcpu-8gb nodes | $144 |
| Block Storage (~115Gi) | $11.50 |
| DO Spaces (250GB) | $5 |
| Cloudflare Tunnel | Free |
| **Total** | **~$160.50** |

## License

MIT
