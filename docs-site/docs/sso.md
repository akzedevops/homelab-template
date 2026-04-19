# SSO

## Provider

**Authentik** (`auth.example.com`) provides OIDC/OAuth2 SSO for all integrated services.

## Integrated Services

| Service | Protocol | Fallback |
|---------|----------|----------|
| ArgoCD | OIDC | Local admin |
| Grafana | OAuth2 | Local admin |
| Gitea | OAuth2 | Local admin |
| Traefik Dashboard | ForwardAuth | Basic auth |

All services retain a local admin account as fallback in case Authentik is unavailable.
