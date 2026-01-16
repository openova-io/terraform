# ADR: Contabo VPS Infrastructure

**Status:** Accepted
**Date:** 2024-06-01

## Context

Need infrastructure for K3s cluster. Previously used Oracle Cloud Always Free but faced severe ARM capacity issues.

## Decision

Use Contabo VPS as primary infrastructure provider.

## Rationale

| Factor | Oracle Always Free | Contabo |
|--------|-------------------|---------|
| Cost | $0 | €13.50/month |
| Availability | Weeks of waiting | Immediate |
| Capacity | ARM shortage | Always available |
| Support | None | Basic |
| Terraform | Official | Official |

**Key Decision Factors:**
- Best $/GB RAM value ($0.54-0.62/GB)
- India DC provides lowest latency to ME among budget providers
- 100% uptime Q3-Q4 2025 (verified)
- Immediate provisioning

## Selected Configuration

| Component | Value |
|-----------|-------|
| Instance Type | VPS 10: 4 vCPU, 8GB RAM, 200GB SSD |
| Node Count | 3 (Control Plane + Worker) |
| Total Resources | 12 vCPU, 24GB RAM, 600GB SSD |
| Primary Region | Europe (Germany) |
| Future Region | India (40-60ms to UAE) |
| Monthly Cost | €13.50 (~$15) |

## Alternatives Considered

| Provider | Cost | Latency to UAE | Why Not |
|----------|------|----------------|---------|
| Oracle Free | $0 | 5-15ms | Capacity issues |
| Hetzner | €21 | 60-80ms | Higher latency |
| AWS/Azure | $250+ | 5-15ms | 15x cost |

## Consequences

**Positive:** Immediate provisioning, consistent availability, good value
**Negative:** Not UAE region (40-60ms vs 5-15ms), smaller company

## Related

- [SPEC-CLOUD-COST-COMPARISON](./SPEC-CLOUD-COST-COMPARISON.md)
- [SPEC-INFRASTRUCTURE-SETUP](./SPEC-INFRASTRUCTURE-SETUP.md)
