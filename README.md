# openova/terraform

Infrastructure as Code for openova kubernetes platform.

## Structure

```
terraform/
├── modules/
│   ├── contabo-vm/         # Contabo VPS provisioning
│   ├── k3s-cluster/        # K3s installation
│   └── dns-failover/       # CoreDNS + Health Orchestrator
├── environments/
│   ├── contabo-europe/     # Primary (Germany/Nuremberg)
│   └── contabo-india/      # Future (Mumbai - ME latency)
└── docs/
    ├── ADR-014-CONTABO-VPS-INFRASTRUCTURE.md
    ├── CLOUD_PROVIDER_COST_COMPARISON.md
    └── INFRASTRUCTURE_SETUP.md
```

## Prerequisites

```bash
# Required tools
terraform >= 1.5.0
sops >= 3.8.0
age >= 1.1.0
```

## Quick Start

```bash
cd environments/contabo-europe

# Decrypt secrets
sops --decrypt terraform.tfvars.enc > terraform.tfvars

# Initialize and apply
terraform init
terraform plan
terraform apply

# Clean up
rm terraform.tfvars
```

## Resources Created

| Resource | Count | Specs |
|----------|-------|-------|
| Contabo VPS 10 | 3 | 4 vCPU, 8GB RAM, 200GB SSD |
| K3s Server (Control Plane + etcd) | 3 | HA quorum |
| Public IPv4 | 3 | Static |

## Monthly Cost

| Environment | Nodes | Cost |
|-------------|-------|------|
| contabo-europe | 3x VPS 10 | ~€13.50 (~$15) |

## Secrets Management

Terraform secrets are encrypted with SOPS + age:

```bash
# Edit encrypted secrets
sops terraform.tfvars.enc

# Encrypt new file
sops --encrypt terraform.tfvars > terraform.tfvars.enc
```

## Related Documentation

- [Infrastructure Setup Guide](./docs/INFRASTRUCTURE_SETUP.md)
- [Cloud Provider Cost Comparison](./docs/CLOUD_PROVIDER_COST_COMPARISON.md)
- [ADR-014: Contabo VPS Infrastructure](./docs/ADR-014-CONTABO-VPS-INFRASTRUCTURE.md)

---

*Part of [openova](https://github.com/openova-io)*
