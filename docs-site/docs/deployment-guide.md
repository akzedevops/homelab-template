# Deployment Guide

How to add a new service to the homelab.

## 1. Create the Application Manifests

For Helm-based apps, create a values file. For plain manifests, create a deployment directory.

```
apps/
  my-app/
    values.yaml       # Helm values
```

Or for raw manifests:

```
apps/
  my-app/
    deployment.yaml
    service.yaml
```

## 2. Create the ArgoCD Application

Create an ArgoCD Application manifest in `argocd/`:

```yaml
# argocd/my-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
  namespace: argocd
  annotations:
    argocd.argoproj.io/manifest-generate-paths: /apps/my-app
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: default
  source:
    repoURL: https://gitea.example.com/<org>/<repo>.git
    targetRevision: main
    path: apps/my-app
    # For Helm:
    # helm:
    #   valueFiles:
    #     - values.yaml
  destination:
    server: https://kubernetes.default.svc
    namespace: my-app
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

The `manifest-generate-paths` annotation ensures ArgoCD only re-renders this app when files under `/apps/my-app` change.

## 3. Create the IngressRoute

Add an IngressRoute in `platform/ingress/`:

```yaml
# platform/ingress/my-app.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: my-app
  namespace: my-app
  annotations:
    external-dns.alpha.kubernetes.io/target: example.com
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`my-app.example.com`)
      kind: Rule
      middlewares:
        - name: crowdsec-bouncer
          namespace: traefik
      services:
        - name: my-app
          port: 8080
  tls: {}
```

Key points:
- `tls: {}` triggers cert-manager to issue a Let's Encrypt certificate via the wildcard
- `external-dns` annotation auto-creates the DNS record in Cloudflare
- `crowdsec-bouncer` middleware protects the route from malicious IPs

## 4. Seal Any Secrets

Never commit plain secrets. Use `kubeseal`:

```bash
# Create the secret manifest (don't apply it)
kubectl create secret generic my-app-secret \
  --namespace my-app \
  --from-literal=DATABASE_URL=<connection-string> \
  --dry-run=client -o yaml | \
  kubeseal --format yaml \
  > platform/sealed-secrets/my-app-secret.yaml
```

## 5. Register in App of Apps

Add the new ArgoCD Application to the root `homelab` app so it's managed by the App of Apps pattern. The root app in `argocd/homelab.yaml` points to the `argocd/` directory — any new Application manifest placed there is auto-discovered.

## 6. Commit and Push

```bash
git add .
git commit -m "feat: add my-app"
git push origin main
```

ArgoCD detects the change and automatically syncs:
1. Creates the namespace
2. Deploys the application
3. Creates the IngressRoute (Traefik picks it up)
4. External-DNS creates the Cloudflare DNS record
5. Cert-manager issues the TLS certificate
6. CrowdSec bouncer protects the endpoint
