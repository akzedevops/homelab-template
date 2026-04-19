# Security

## CrowdSec

CrowdSec provides collaborative intrusion detection and prevention. The homelab runs:

- **LAPI (Local API)**: Central decision engine deployed as a single pod
- **2 Agents**: Parse Traefik access logs and detect malicious behavior (brute force, scanning, etc.)
- **Traefik Bouncer**: Enforces ban decisions as a ForwardAuth middleware on all IngressRoutes

When a request hits Traefik, the bouncer middleware queries the LAPI. If the source IP is banned, the request is rejected before reaching the backend service.

### Bouncer Middleware Example

The bouncer runs as a ForwardAuth middleware applied to every IngressRoute:

```yaml
# Middleware definition
apiVersion: traefik.io/v1alpha1
kind: Middleware
metadata:
  name: crowdsec-bouncer
  namespace: traefik
spec:
  forwardAuth:
    address: http://crowdsec-bouncer.crowdsec.svc.cluster.local:8080/api/v1/forwardAuth
    trustForwardHeader: true
```

```yaml
# Applied on every IngressRoute
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: example-app
  annotations:
    external-dns.alpha.kubernetes.io/target: example.com
spec:
  entryPoints:
    - websecure
  routes:
    - match: Host(`example.example.com`)
      kind: Rule
      middlewares:
        - name: crowdsec-bouncer
          namespace: traefik
      services:
        - name: example-app
          port: 8080
  tls: {}
```

## Falco

Falco monitors runtime syscalls via eBPF to detect anomalous container behavior — shell spawns, unexpected network connections, file access violations, etc.

- **Falco**: DaemonSet with eBPF driver (no kernel module needed)
- **Falcosidekick**: Receives Falco alerts and exposes Prometheus metrics
- **ServiceMonitor**: Scrapes falcosidekick metrics into Prometheus for Grafana dashboards

## Kubescape

Kubescape runs vulnerability and compliance scanning against the cluster:

- Scans workloads against NSA/CISA and MITRE ATT&CK frameworks
- Detects misconfigurations (missing resource limits, privileged containers, etc.)
- Image vulnerability scanning
- Deployed as an ArgoCD-managed application

## Cloudflare Tunnel

Cloudflare Tunnel eliminates the need for a public IP or open inbound ports on the cluster:

- **Origin IP hidden**: DNS records are CNAMEs pointing to the tunnel — no A records expose the cluster's IP
- **No inbound attack surface**: `cloudflared` initiates outbound QUIC connections to Cloudflare Edge; attackers cannot bypass Cloudflare to hit the origin directly
- **Real client IPs preserved**: Cloudflare passes the original client IP via `X-Forwarded-For`. Traefik is configured with Cloudflare's IP ranges in `trustedIPs`, so CrowdSec sees and can ban real attacker IPs
- **DDoS protection**: All traffic passes through Cloudflare's network before reaching the cluster

## Sealed Secrets

All Kubernetes secrets are encrypted before committing to Git using Sealed Secrets:

- The `sealed-secrets` controller runs in-cluster and holds the decryption key
- Developers encrypt secrets locally with `kubeseal` against the cluster's public key
- Only `SealedSecret` resources exist in Git — never plain `Secret` manifests
- The controller decrypts them at apply time into regular `Secret` objects

```bash
# Seal a secret
kubectl create secret generic my-secret \
  --from-literal=key=value \
  --dry-run=client -o yaml | \
  kubeseal --format yaml > sealed-my-secret.yaml
```
