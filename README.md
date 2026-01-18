# openova/terraform

Infrastructure as Code for openova kubernetes platform.

## Structure

```
terraform/
├── modules/
│   ├── contabo-vm/         # Contabo VPS provisioning
│   ├── k3s-cluster/        # K3s installation
│   └── dns-failover/       # CoreDNS + k8gb
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
```

## Quick Start

```bash
cd environments/contabo-europe

# Bootstrap wizard handles credentials interactively
# No secrets stored in Git - credentials entered at runtime

# Initialize and apply
terraform init
terraform plan -var-file=terraform.tfvars  # Created interactively
terraform apply -var-file=terraform.tfvars
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

**No SOPS:** Secrets are handled via interactive bootstrap:

1. **Bootstrap Wizard** prompts for cloud credentials
2. Creates terraform.tfvars locally (not committed to Git)
3. Provisions infrastructure
4. Initializes Vault with generated unseal keys
5. ESO PushSecrets sync to both regional Vaults

See [ADR-SECRETS-MANAGEMENT](../external-secrets/docs/ADR-SECRETS-MANAGEMENT.md) for full details.

## Related Documentation

- [Infrastructure Setup Guide](./docs/INFRASTRUCTURE_SETUP.md)
- [Cloud Provider Cost Comparison](./docs/CLOUD_PROVIDER_COST_COMPARISON.md)
- [ADR-014: Contabo VPS Infrastructure](./docs/ADR-014-CONTABO-VPS-INFRASTRUCTURE.md)

---

*Part of [OpenOva](https://openova.io)*
