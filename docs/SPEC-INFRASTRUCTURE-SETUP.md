# SPEC: Infrastructure Setup

## Overview

Terraform-based infrastructure provisioning for Contabo VPS.

## Prerequisites

```bash
terraform >= 1.5.0
```

## Module Structure

```
terraform/
├── modules/
│   ├── contabo-vm/         # VPS provisioning
│   ├── k3s-cluster/        # K3s installation
│   └── dns-failover/       # CoreDNS + k8gb
├── environments/
│   ├── contabo-europe/     # Primary
│   └── contabo-india/      # Future
└── docs/
```

## Quick Start

```bash
cd environments/contabo-europe

# Bootstrap wizard handles credentials interactively
# Creates terraform.tfvars (not committed to Git)

# Initialize and apply
terraform init
terraform plan
terraform apply

# terraform.tfvars is never committed
```

## Resources Created

| Resource | Count | Specs |
|----------|-------|-------|
| Contabo VPS 10 | 3 | 4 vCPU, 8GB RAM, 200GB SSD |
| K3s Server | 3 | Control Plane + etcd |
| Public IPv4 | 3 | Static |

## K3s Cluster Configuration

### Disabled Components

| Component | Reason |
|-----------|--------|
| traefik | Gateway API (Cilium) handles ingress |
| servicelb | DNS-based failover |
| local-storage | App-level replication |
| flannel | Cilium CNI |

### Optimization Parameters

| Parameter | Value | Purpose |
|-----------|-------|---------|
| node-monitor-period | 5s | Faster health detection |
| node-monitor-grace-period | 20s | Faster failover |
| default-watch-cache-size | 50 | Memory optimization |
| quota-backend-bytes | 1GB | etcd limit |
| max-pods | 50 | Per-node limit |

## Secrets Management

**No SOPS:** All secrets handled via interactive bootstrap.

1. Bootstrap wizard prompts for credentials
2. Creates terraform.tfvars locally (gitignored)
3. Provisions infrastructure
4. Initializes Vault
5. ESO manages K8s secrets

Operator saves Vault unseal keys offline - the only manual backup required.

## Post-Bootstrap

After Terraform provisioning:

1. Install Cilium CNI
2. Bootstrap Flux (from Gitea)
3. Flux deploys remaining components

## Related

- [ADR-CONTABO-VPS-INFRASTRUCTURE](./ADR-CONTABO-VPS-INFRASTRUCTURE.md)
- [SPEC-CLOUD-COST-COMPARISON](./SPEC-CLOUD-COST-COMPARISON.md)
