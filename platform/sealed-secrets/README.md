# Sealed Secrets

Place your SealedSecret manifests here. Generate with:

```bash
kubectl create secret generic <name> --from-literal=key=value -n <ns> --dry-run=client -o json | \
  kubeseal --controller-name=sealed-secrets --controller-namespace=kube-system --format=yaml > <name>.yaml
```
