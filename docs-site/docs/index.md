# Homelab Docs

Production-grade self-hosted homelab running on **DigitalOcean Kubernetes (DOKS)**, fully GitOps-managed via **ArgoCD**.

## Overview

- **Cluster**: DOKS 1.35.1 — 3 nodes in nyc1
- **GitOps**: ArgoCD App of Apps pattern with **31 managed applications**
- **Public Services**: 13 services exposed via `*.example.com` through **Cloudflare Tunnel**
- **CI/CD**: Gitea Actions + Renovate for automated dependency updates
- **Security**: Sealed Secrets, CrowdSec, Kubescape, Authentik SSO
- **Monitoring**: Prometheus, Grafana, Loki, Hubble

## Quick Links

| Resource | URL |
|----------|-----|
| ArgoCD | [argocd.example.com](https://argocd.example.com) |
| Grafana | [grafana.example.com](https://grafana.example.com) |
| Homepage | [home.example.com](https://home.example.com) |

!!! tip "Getting Started"
    See the [Deployment Guide](deployment.md) to provision the cluster from scratch, or [Architecture](architecture.md) for a full system overview.
<!-- built 2026-04-19T17:02:58Z -->
