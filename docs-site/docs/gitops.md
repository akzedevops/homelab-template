# GitOps

Everything in this homelab is deployed and managed through Git — no manual `kubectl apply` in production.

## App of Apps Pattern

ArgoCD uses the **App of Apps** pattern:

1. A single root `Application` (`homelab`) points to the `argocd/` directory
2. That directory contains individual `Application` manifests for each service
3. ArgoCD recursively syncs all child applications automatically

```
homelab (root)
├── argocd
├── traefik
├── monitoring
├── gitea
├── authentik
├── nextcloud
├── jellyfin
├── vaultwarden
├── n8n
├── ...24 apps total
```

!!! info "manifest-generate-paths"
    Each ArgoCD Application uses the `argocd.argoproj.io/manifest-generate-paths` annotation to scope change detection. This prevents unnecessary syncs when unrelated files change in the monorepo.

## Webhook-Driven Sync

Gitea sends a webhook to ArgoCD on every push. This triggers an immediate refresh of affected applications instead of waiting for the default 3-minute polling interval.

## CI Pipeline

Every push to the main branch runs a Gitea Actions workflow:

```yaml
# .gitea/workflows/lint.yaml
steps:
  - name: YAML Lint
    run: yamllint .
  - name: Secret Scan
    run: gitleaks detect --source .
```

This catches YAML syntax errors and prevents accidental secret commits before ArgoCD syncs.

## Renovate

[Renovate](https://docs.renovatebot.com/) runs as a CronJob every **Saturday at 3:00 AM UTC**, automatically creating PRs for:

- Helm chart version bumps
- Container image tag updates
- GitHub Actions version updates

Configuration lives in `renovate.json` at the repo root. PRs are auto-merged if CI passes and the update is a patch/minor version.
