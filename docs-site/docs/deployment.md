# Deployment Guide

## Prerequisites

- DigitalOcean account with API token
- `kubectl` and `doctl` CLI installed
- Cloudflare account managing `example.com`

## Steps

1. **Provision DOKS cluster**
   ```bash
   doctl kubernetes cluster create homelab \
     --region nyc1 \
     --node-pool "name=default;size=s-4vcpu-8gb;count=3" \
     --version 1.35.1-do.0
   ```

2. **Install ArgoCD**
   ```bash
   kubectl create namespace argocd
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

3. **Apply the root App of Apps**
   ```bash
   kubectl apply -f argocd/homelab.yaml
   ```
   This deploys all 24 applications automatically.

4. **Create required secrets** (or apply sealed secrets)
   - Cloudflare API token (cert-manager, external-dns)
   - DO Spaces access keys (CSI-S3)
   - Authentik secret key
   - Database credentials

!!! warning
    Never commit plaintext secrets. Use `kubeseal` to encrypt them as SealedSecrets before pushing to Git.
