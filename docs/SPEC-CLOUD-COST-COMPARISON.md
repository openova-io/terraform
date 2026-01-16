# SPEC: Cloud Provider Cost Comparison

## Overview

Cost analysis for 24GB RAM cluster targeting Middle East latency.

## Budget Providers (Under $25/month)

| Provider | Cost/24GB | Latency to UAE | Terraform | Stability |
|----------|-----------|----------------|-----------|-----------|
| **Contabo** | €13.50 | 40-60ms (India) | Official | 7/10 |
| Hetzner | €21 | 60-80ms (Singapore) | Official | 8/10 |
| Netcup | ~€20 | 80-100ms | **None** | 5/10 |
| Hostinger | ~€24 | 70-100ms | Official | 7/10 |

## Mid-Range Providers ($50-150/month)

| Provider | Cost/24GB | Latency to UAE | Notes |
|----------|-----------|----------------|-------|
| Vultr | ~$105 | 40-60ms (Mumbai) | Best global coverage |
| DigitalOcean | ~$95 | 50-70ms (Bangalore) | Excellent docs |
| Linode/Akamai | ~$179 | 40-60ms (Mumbai) | Akamai backing |

## Hyperscalers

| Provider | Cost/24GB | Latency to UAE | Region |
|----------|-----------|----------------|--------|
| AWS | ~$263 | 10-20ms | Bahrain |
| Azure | ~$282 | 5-15ms | UAE (Dubai) |
| GCP | ~$284 | 20-40ms | Qatar (Doha) |
| Oracle | ~$96/$0 | 5-15ms | UAE - **capacity issues** |
| Alibaba | ~$299 | 5-15ms | Dubai |

## Vendor Profiles

### Contabo (Selected)

| Attribute | Value |
|-----------|-------|
| Founded | 2003 (22 years) |
| Revenue | ~$36M |
| Employees | ~200-300 |
| Uptime | 100% Q3-Q4 2025 |
| Data Centers | Germany, UK, USA, Singapore, Japan, Australia, **India** |

### Hetzner (Fallback)

| Attribute | Value |
|-----------|-------|
| Founded | 1997 (28 years) |
| Revenue | €447M |
| Employees | ~300 |
| Data Centers | Germany, Finland, USA, Singapore |
| Note | More mature but no India DC |

## Final Selection

| Priority | Choice | Latency | Cost | Status |
|----------|--------|---------|------|--------|
| Lowest Cost | Contabo India | 40-60ms | €13.50 | **Selected** |
| Fallback | Hetzner Germany | 80-100ms | €21 | Backup |
| Deprecated | Oracle Free | 5-15ms | $0 | Capacity hunting |

## Related

- [ADR-CONTABO-VPS-INFRASTRUCTURE](./ADR-CONTABO-VPS-INFRASTRUCTURE.md)
