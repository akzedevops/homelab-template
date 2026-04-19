# Cost

## Monthly Breakdown

| Resource | Spec | Est. Cost |
|----------|------|-----------|
| DOKS Nodes | 3× s-4vcpu-8gb | ~$144/mo |
| Cloudflare Tunnel | Free tier | $0/mo |
| Block Storage | PVCs for databases | Varies |
| DO Spaces | Media + backups | ~$5/mo base |

**Estimated total: ~$160.50/mo** (excluding variable storage)

!!! note "Savings from Cloudflare Tunnel"
    Migrating from a DO Load Balancer ($12/mo) to Cloudflare Tunnel ($0) saved $12/mo while also hiding the origin IP.

!!! tip
    Use the [DigitalOcean Pricing Calculator](https://www.digitalocean.com/pricing) for current estimates. Prices may change.
