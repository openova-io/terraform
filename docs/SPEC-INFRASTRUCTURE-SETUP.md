# SPEC: Infrastructure Setup

## Overview

Terraform-based infrastructure provisioning for Contabo VPS.

## Prerequisites

```bash
terraform >= 1.5.0
sops >= 3.8.0
age >= 1.1.0
```

## Module Structure

```
terraform/
├── modules/
│   ├── contabo-vm/         # VPS provisioning
│   ├── k3s-cluster/        # K3s installation
│   └── dns-failover/       # CoreDNS + Health Orchestrator
├── environments/
│   ├── contabo-europe/     # Primary
│   └── contabo-india/      # Future
└── docs/
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

# Clean up secrets
rm terraform.tfvars
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
| traefik | Istio handles ingress |
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

Terraform secrets encrypted with SOPS + age:

```bash
# Edit secrets
sops terraform.tfvars.enc

# Encrypt new file
sops --encrypt terraform.tfvars > terraform.tfvars.enc
```

## Post-Bootstrap

After Terraform provisioning:

1. Install Cilium CNI
2. Bootstrap Flux
3. Flux deploys remaining components

## Related

- [ADR-CONTABO-VPS-INFRASTRUCTURE](./ADR-CONTABO-VPS-INFRASTRUCTURE.md)
- [SPEC-CLOUD-COST-COMPARISON](./SPEC-CLOUD-COST-COMPARISON.md)
