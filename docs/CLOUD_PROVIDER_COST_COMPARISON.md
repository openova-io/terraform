# Comprehensive Cloud Provider Cost Comparison for 3-Node Self-Managed Kubernetes Cluster

**Target Specs per Node:** 2 vCPU (or equivalent ARM cores), 8GB RAM, 20-40GB boot volume
**Total Cluster:** 3 nodes
**Last Updated:** January 2026

---

## Executive Summary

For a 3-node self-managed Kubernetes cluster with the specified requirements, **Hetzner Cloud** and **OVHcloud** offer the best value, followed by **Vultr** and **Linode**. The hyperscalers (AWS, Azure, GCP) are significantly more expensive but offer superior global reach and managed services.

**Top 3 Recommendations:**
1. **Hetzner CX33 (x86)**: ~$17.50/month for 3 nodes with 4 vCPU/8GB each - best overall value
2. **OVHcloud VPS-1**: ~$12.60/month for 3 nodes with 4 vCPU/8GB each - incredible value with unlimited traffic
3. **Vultr Cloud Compute**: ~$72/month for 3 nodes with 2 vCPU/4GB each (closest match)

---

## Cost Per GB RAM Analysis

**Non-negotiable requirement:** 24GB total RAM across nodes (for K8s + databases + services)
**Negotiable:** Number of nodes, vCPU count, storage size

### Cheapest $/GB RAM by Provider

| Provider | Plan | RAM | Price/mo | **$/GB RAM** | Terraform |
|----------|------|-----|----------|--------------|-----------|
| **Contabo** | Cloud VPS 10 | 8GB | $4.95 | **$0.62** | Official |
| **Contabo** | Cloud VPS 30 | 24GB | ~$13 | **$0.54** | Official |
| **Netcup** | VPS 1000 G11 | 8GB | €6.85 (~$7.35) | **$0.92** | None |
| **Hetzner** | CX32 | 8GB | €6.80 (~$7.30) | **$0.91** | Official |
| **Hetzner** | CAX21 (ARM) | 8GB | €6.49 (~$7.00) | **$0.88** | Official |
| **Hostinger** | KVM 2 | 8GB | $6.99 | **$0.87** | Official |
| **OVHcloud** | VPS Comfort | 8GB | ~$12 | **$1.50** | Official |
| **Scaleway** | DEV1-L | 8GB | ~$33 | **$4.08** | Official |
| **Linode** | Shared 8GB | 8GB | $48 | **$6.00** | Official |
| **Vultr** | Cloud Compute | 4GB | $20 | **$5.00** | Official |
| **DigitalOcean** | Basic | 4GB | $20 | **$5.00** | Official |
| **Oracle Cloud** | A1.Flex | 8GB | ~$22 | **$2.75** | Official |
| **AWS** | t4g.large | 8GB | $49 | **$6.13** | Official |
| **Azure** | D2ps v5 | 8GB | ~$56 | **$7.02** | Official |
| **GCP** | e2-standard-2 | 8GB | $49 | **$6.13** | Official |
| **Alibaba** | ecs.g6.large | 8GB | ~$72 | **$9.00** | Official |

### Cheapest Configurations for 24GB Total RAM

| Rank | Configuration | Total RAM | vCPUs | Monthly Cost | Terraform |
|------|---------------|-----------|-------|--------------|-----------|
| **1** | **Contabo 1x VPS 30** | 24GB | 8 | **~$13** | Official |
| **2** | **Contabo 3x VPS 10** | 24GB | 9 (3x3) | **~$15** | Official |
| **3** | **Netcup 3x VPS 1000** | 24GB | 12 (3x4) | **~$22** | None |
| **4** | **Hetzner 3x CX32** | 24GB | 12 (3x4) | **~$22** | Official |
| **5** | **Hetzner 3x CAX21** | 24GB | 12 (3x4) | **~$21** | Official |
| **6** | **Hostinger 3x KVM 2** | 24GB | 6 (3x2) | **~$21** | Official |
| **7** | **Hetzner 6x CX22** | 24GB | 12 (6x2) | **~$24** | Official |
| **8** | **Hetzner 2x CX42** | 32GB | 16 (2x8) | **~$35** | Official |

### Recommendation: Minimum Cost for 24GB RAM

**Best overall (HA + Terraform):**
```
Hetzner 3x CAX21 (ARM) = ~$21/month
├── 24GB RAM total (3x 8GB)
├── 12 vCPUs total (3x 4 ARM cores)
├── 240GB NVMe total (3x 80GB) - enough for MinIO
├── 60TB egress total (3x 20TB)
├── Official Terraform provider
└── 3 nodes for K8s HA
```

**Cheapest absolute (single node, no HA):**
```
Contabo 1x Cloud VPS 30 = ~$13/month
├── 24GB RAM
├── 8 vCPUs
├── 200GB NVMe
├── 32TB traffic
├── Official Terraform provider
└── ⚠️ Single node = no HA, not recommended for K8s
```

**Cheapest with HA:**
```
Contabo 3x Cloud VPS 10 = ~$15/month
├── 24GB RAM total (3x 8GB)
├── 9 vCPUs total (3x 3 cores)
├── 225GB NVMe total (3x 75GB)
├── 96TB traffic total (3x 32TB)
├── Official Terraform provider
└── 3 nodes for K8s HA
```

---

## Pricing Comparison Table

**Note:** Since most providers don't offer exactly 2 vCPU + 8GB RAM combinations, I've selected the closest available configurations.

### Compute Instances (3 Nodes)

| Provider | Instance Type | vCPU | RAM | Storage | CPU Type | Price/Node/Mo | 3-Node Total |
|----------|---------------|------|-----|---------|----------|---------------|--------------|
| **Contabo** | Cloud VPS 10 | 3 | 8 GB | 75 GB | Intel/AMD (x86) | $4.95 | **~$14.85** |
| **Hetzner** | CX33 | 4 | 8 GB | 80 GB | AMD EPYC (x86) | €5.49 (~$5.83) | **~$17.50** |
| **Hetzner** | CAX21 | 4 | 8 GB | 80 GB | Ampere Altra (ARM) | €6.49 (~$6.90) | **~$20.70** |
| **OVHcloud** | VPS-1 | 4 | 8 GB | 75 GB | Intel (x86) | $4.20 | **~$12.60** |
| **Linode** | Shared 8GB | 4 | 8 GB | 160 GB | AMD EPYC (x86) | $48.00 | **~$144.00** |
| **Vultr** | Cloud Compute | 2 | 4 GB | 80 GB | Intel/AMD (x86) | $20.00 | **~$60.00** |
| **Vultr** | High Performance | 2 | 4 GB | 60 GB | AMD EPYC (x86) | $24.00 | **~$72.00** |
| **DigitalOcean** | Basic Regular | 2 | 4 GB | 80 GB | Intel/AMD (x86) | $20.00 | **~$60.00** |
| **Scaleway** | DEV1-L | 4 | 8 GB | 80 GB | Intel (x86) | €30.66 (~$32.60) | **~$97.80** |
| **Scaleway** | PRO2-XXS | 2 | 8 GB | - | Intel (x86) | €40.15 (~$42.70) | **~$128.10** |
| **AWS** | t4g.large | 2 | 8 GB | 8 GB EBS | Graviton2 (ARM) | ~$49.06 | **~$147.18** |
| **AWS** | t3.large | 2 | 8 GB | 8 GB EBS | Intel (x86) | ~$60.74 | **~$182.22** |
| **Azure** | D2ps v5 | 2 | 8 GB | - | Ampere Altra (ARM) | ~$56.21 | **~$168.63** |
| **GCP** | e2-standard-2 | 2 | 8 GB | 10 GB | Intel/AMD (x86) | ~$48.92 | **~$146.76** |
| **Oracle Cloud** | A1.Flex (2 OCPU) | 2 | 8 GB | 100 GB boot | Ampere Altra (ARM) | ~$22.03 | **~$66.09** |
| **Alibaba Cloud** | ecs.g6.large | 2 | 8 GB | 40 GB | Intel Xeon (x86) | ~$72.00 | **~$216.00** |

*Currency conversion: €1 = ~$1.063 USD (approximate)*

---

## Block Storage Pricing (for CSI/PVCs - 50GB x 3 = 150GB)

| Provider | Type | Price/GB/Month | 150GB Total/Month |
|----------|------|----------------|-------------------|
| **Contabo** | NVMe (included) | Included in VPS | **$0** (use boot disk) |
| **Hetzner** | NVMe SSD | €0.044 (~$0.047) | **~$7.05** |
| **OVHcloud** | SSD | ~$0.04 | **~$6.00** |
| **Linode** | NVMe SSD | $0.10 | **~$15.00** |
| **Vultr** | NVMe SSD | $0.10 | **~$15.00** |
| **DigitalOcean** | SSD | $0.10 | **~$15.00** |
| **Scaleway** | SSD | €0.086 (~$0.091) | **~$13.65** |
| **AWS** | gp3 SSD | $0.08 | **~$12.00** |
| **Azure** | Standard SSD | ~$0.15 (128GB tier) | **~$22.50** |
| **GCP** | pd-ssd | $0.187 | **~$28.05** |
| **Oracle Cloud** | Balanced | ~$0.025 | **~$3.75** |
| **Alibaba Cloud** | ESSD | ~$0.05 | **~$7.50** |

---

## Object Storage Pricing (S3-Compatible, 1TB Storage)

| Provider | Product | Storage/TB/Mo | Egress/TB | S3-Compatible | 1TB Cost/Mo |
|----------|---------|---------------|-----------|---------------|-------------|
| **Contabo** | Object Storage | €2.49/250GB (~$10/TB) | Included | Yes | **~$10.60** |
| **Hetzner** | Object Storage | €4.99 base (1TB incl) | €1.00/TB | Yes | **~$5.31** |
| **OVHcloud** | Object Storage | ~$12/TB | Free (internal) | Yes | **~$12.00** |
| **Linode** | Object Storage | $5 (250GB) + $0.02/GB | $0.005/GB | Yes | **~$20.00** |
| **Vultr** | Object Storage | $18/TB (Standard) | $0.01/GB | Yes | **~$18.00** |
| **DigitalOcean** | Spaces | $5 (250GB) + $0.02/GB | $0.01/GB | Yes | **~$20.00** |
| **Scaleway** | Object Storage | €0.012/GB (~$12/TB) | Varies | Yes | **~$12.76** |
| **AWS** | S3 Standard | ~$0.023/GB | $0.09/GB | Native | **~$23.55** |
| **Azure** | Blob Hot | ~$0.018/GB | $0.087/GB | Partial | **~$18.43** |
| **GCP** | Cloud Storage | ~$0.020/GB | $0.12/GB | Partial | **~$20.48** |
| **Oracle Cloud** | Object Storage | ~$0.0255/GB | 10TB free | Yes | **~$26.11** |
| **Alibaba Cloud** | OSS Standard | ~$0.02/GB | $0.076/GB | Yes | **~$20.48** |

---

## Load Balancer Pricing

| Provider | Product | Price/Month | Notes |
|----------|---------|-------------|-------|
| **Contabo** | No managed LB | N/A | Use MetalLB |
| **Hetzner** | Load Balancer | €5.39 (~$5.73) | 1-2TB traffic included |
| **OVHcloud** | Load Balancer | ~$12.00 | Via Public Cloud |
| **Linode** | NodeBalancer | $10.00 | Per NodeBalancer |
| **Vultr** | Load Balancer | $10.00 | Bandwidth neutral |
| **DigitalOcean** | Load Balancer | $12.00 | Regional HTTP |
| **Scaleway** | Load Balancer | ~$12.00 | S-size |
| **AWS** | ALB | ~$24.24 | Base + LCU charges |
| **Azure** | Standard LB | ~$18.00 | + data processing |
| **GCP** | HTTP(S) LB | ~$18.00 | + data processing |
| **Oracle Cloud** | Network LB | **FREE** | Layer 4 |
| **Alibaba Cloud** | CLB/ALB | ~$15-20 | Pay-by-LCU (usage-based) |

**Alternative:** Use **MetalLB** on bare-metal/VPS providers (Hetzner, Vultr, Linode, DO, OVH, Contabo) for $0/month. Requires floating IPs or node ports.

---

## Public IP & Networking Costs

| Provider | Public IP Cost | Egress Included | Extra Egress Cost |
|----------|----------------|-----------------|-------------------|
| **Contabo** | Included | 32 TB/node | ~$0.01/GB |
| **Hetzner** | Included | 20 TB/node | €1.00/TB |
| **OVHcloud** | Included | Unlimited | N/A |
| **Linode** | Included | 4-5 TB/node | $0.005/GB |
| **Vultr** | Included | 3-4 TB/node | $0.01/GB |
| **DigitalOcean** | Included | 4-6 TB/node | $0.01/GB |
| **Scaleway** | €0.004/hr (~$3/mo) | Generous | Varies |
| **AWS** | $0.005/hr (~$3.65/mo each) | 100 GB free | $0.09/GB |
| **Azure** | $0.005/hr (~$3.65/mo each) | 100 GB free | $0.087/GB |
| **GCP** | $0.005/hr (~$3.65/mo each) | Standard Tier | $0.12/GB |
| **Oracle Cloud** | Included | 10 TB/month | $0.0085/GB |
| **Alibaba Cloud** | Included with ECS | 1 TB/node | ~$0.076/GB |

---

## Performance Benchmarks (Geekbench 6)

| Provider | Instance | CPU Model | Single-Core | Multi-Core (4 core equivalent) |
|----------|----------|-----------|-------------|-------------------------------|
| **Contabo VPS 10** | 3 x86 vCPU | Intel/AMD | ~900* | ~2700* |
| **Hetzner CAX21** | 4 ARM vCPU | Ampere Altra | ~1039 | ~3268 |
| **Hetzner CX33** | 4 x86 vCPU | AMD EPYC Genoa | ~1200* | ~3800* |
| **OVHcloud VPS-1** | 4 x86 vCPU | Intel Xeon | ~950* | ~3000* |
| **AWS t4g.large** | 2 ARM vCPU | Graviton2 | ~1050 | ~2100 |
| **AWS t3.large** | 2 x86 vCPU | Intel Xeon | ~950 | ~1900 |
| **Azure D2ps v5** | 2 ARM vCPU | Ampere Altra | ~1100 | ~2200 |
| **GCP e2-standard-2** | 2 x86 vCPU | Intel/AMD | ~900 | ~1800 |
| **Oracle A1.Flex** | 2 ARM OCPU | Ampere Altra | ~1050 | ~2100 |
| **Linode 8GB** | 4 x86 vCPU | AMD EPYC | ~1100 | ~3500* |
| **Vultr HP** | 2 x86 vCPU | AMD EPYC | ~1050 | ~2100 |
| **DigitalOcean** | 2 x86 vCPU | Intel/AMD | ~950* | ~1900* |
| **Scaleway DEV1-L** | 4 x86 vCPU | Intel Xeon | ~900* | ~2800* |
| **Alibaba Cloud** | 2 x86 vCPU | Intel Xeon | ~950 | ~1900 |

*Estimated based on relative CPU performance data. Actual scores may vary.

---

## Total Monthly Cost Calculation

**Assumptions:**
- 3 compute nodes (closest to 2 vCPU/8GB)
- 150 GB block storage total (for database PVCs)
- 1 TB object storage
- 1 load balancer (or MetalLB = $0)
- 3 public IPs (or 1 if behind LB)
- 500 GB egress/month (typically within free tier)

### With Managed Load Balancer

| Provider | Compute | Storage | Object | LB | IPs | Egress | **TOTAL** |
|----------|---------|---------|--------|-----|-----|--------|-----------|
| **Contabo** | $14.85 | $0 | $10.60 | N/A | Incl | Incl | **~$25.45** (no LB) |
| **OVHcloud** | $12.60 | $6.00 | $12.00 | $12.00 | Incl | Incl | **~$42.60** |
| **Hetzner (CX33)** | $17.50 | $7.05 | $5.31 | $5.73 | Incl | Incl | **~$35.59** |
| **Hetzner (CAX21)** | $20.70 | $7.05 | $5.31 | $5.73 | Incl | Incl | **~$38.79** |
| **Vultr** | $72.00 | $15.00 | $18.00 | $10.00 | Incl | Incl | **~$115.00** |
| **DigitalOcean** | $60.00 | $15.00 | $20.00 | $12.00 | Incl | Incl | **~$107.00** |
| **Linode** | $144.00 | $15.00 | $20.00 | $10.00 | Incl | Incl | **~$189.00** |
| **Scaleway** | $97.80 | $13.65 | $12.76 | $12.00 | $9.00 | Incl | **~$145.21** |
| **Oracle Cloud (Paid)** | $66.09 | $3.75 | $26.11 | FREE | Incl | Incl | **~$95.95** |
| **AWS (t4g.large)** | $147.18 | $12.00 | $23.55 | $24.24 | $10.95 | $45.00 | **~$262.92** |
| **Azure (D2ps v5)** | $168.63 | $22.50 | $18.43 | $18.00 | $10.95 | $43.50 | **~$282.01** |
| **GCP (e2-standard-2)** | $146.76 | $28.05 | $20.48 | $18.00 | $10.95 | $60.00 | **~$284.24** |
| **Alibaba Cloud** | $216.00 | $7.50 | $20.48 | $17.50 | Incl | $38.00 | **~$299.48** |

### With MetalLB (Self-Hosted Load Balancing)

| Provider | Compute | Storage | Object | IPs | Egress | **TOTAL** |
|----------|---------|---------|--------|-----|--------|-----------|
| **Contabo** | $14.85 | $0 | $10.60 | Incl | Incl | **~$25.45** |
| **OVHcloud** | $12.60 | $6.00 | $12.00 | Incl | Incl | **~$30.60** |
| **Hetzner (CX33)** | $17.50 | $7.05 | $5.31 | Incl | Incl | **~$29.86** |
| **Hetzner (CAX21)** | $20.70 | $7.05 | $5.31 | Incl | Incl | **~$33.06** |
| **Linode** | $144.00 | $15.00 | $20.00 | Incl | Incl | **~$179.00** |
| **Vultr** | $72.00 | $15.00 | $18.00 | Incl | Incl | **~$105.00** |
| **DigitalOcean** | $60.00 | $15.00 | $20.00 | Incl | Incl | **~$95.00** |
| **Scaleway** | $97.80 | $13.65 | $12.76 | $9.00 | Incl | **~$133.21** |
| **Oracle Cloud (Paid)** | $66.09 | $3.75 | $26.11 | Incl | Incl | **~$95.95** |
| **Alibaba Cloud** | $216.00 | $7.50 | $20.48 | Incl | $38.00 | **~$281.98** |

---

## Performance per Dollar Ranking

Based on Geekbench 6 multi-core scores (scaled to 4 vCPU equivalent) divided by total monthly cost:

| Rank | Provider | Config | GB6 Multi-Core | Monthly Cost | Score/$ |
|------|----------|--------|----------------|--------------|---------|
| 1 | **Contabo VPS 10** | 3 vCPU x86 | ~2700* | $25.45 | **106.1** |
| 2 | **Hetzner CX33** | 4 vCPU x86 | ~3800 | $29.86 | **127.3** |
| 3 | **OVHcloud VPS-1** | 4 vCPU x86 | ~3000* | $30.60 | **98.0** |
| 4 | **Hetzner CAX21** | 4 ARM vCPU | ~3268 | $33.06 | **98.8** |
| 5 | **Vultr HP** | 2 vCPU x86 | ~2100 | $105.00 | **20.0** |
| 6 | **DigitalOcean** | 2 vCPU x86 | ~1900 | $95.00 | **20.0** |
| 7 | **Oracle Cloud** | 2 ARM OCPU | ~2100 | $95.95 | **21.9** |
| 8 | **Scaleway DEV1-L** | 4 vCPU x86 | ~2800* | $133.21 | **21.0** |
| 9 | **Linode** | 4 vCPU x86 | ~3500 | $179.00 | **19.6** |
| 10 | **AWS t4g.large** | 2 ARM vCPU | ~2100 | $262.92 | **8.0** |
| 11 | **Azure D2ps v5** | 2 ARM vCPU | ~2200 | $282.01 | **7.8** |
| 12 | **Alibaba Cloud** | 2 vCPU x86 | ~1900 | $281.98 | **6.7** |
| 13 | **GCP e2-standard-2** | 2 vCPU x86 | ~1800 | $284.24 | **6.3** |

---

## Provider-Specific Notes & Caveats

### Contabo (EU/Global)
- **Pros:** Cheapest $/GB RAM, India DC (best ME latency in budget tier), 32TB traffic included, 100% uptime in 2025
- **Cons:** Smaller company, mixed support reviews, no managed load balancer
- **Best for:** Budget-conscious deployments targeting Middle East users

### Hetzner (EU)
- **Pros:** Best price/performance, excellent network, 20TB egress included, ARM and x86 options
- **Cons:** EU-only (Germany/Finland), no global presence, limited managed services
- **Best for:** Cost-conscious self-managed K8s deployments in EU

### OVHcloud (EU/Global)
- **Pros:** Incredible value with unlimited traffic, decent global presence
- **Cons:** Smaller community, fewer integrations, Intel-only
- **Best for:** Budget deployments with unpredictable traffic

### Vultr (Global)
- **Pros:** 32 global locations, good high-performance options, NVMe storage
- **Cons:** More expensive than Hetzner/OVH, 2 vCPU plans have 4GB RAM
- **Best for:** Global deployments needing multi-region presence

### DigitalOcean (Global)
- **Pros:** Developer-friendly, excellent docs, managed K8s available
- **Cons:** More expensive than alternatives, 2 vCPU = 4GB RAM max
- **Best for:** Startups, developer-focused teams

### Linode/Akamai (Global)
- **Pros:** Mature platform, generous bandwidth, AMD EPYC CPUs
- **Cons:** Shared CPU plans expensive for specs
- **Best for:** Teams wanting established provider with good support

### Scaleway (EU)
- **Pros:** Good ARM options, EU-based, decent storage pricing
- **Cons:** Complex pricing, IPv4 charged separately
- **Best for:** EU deployments needing Kubernetes-specific features

### AWS (Global)
- **Pros:** Largest ecosystem, best global reach, Graviton2 ARM excellent
- **Cons:** Complex pricing, expensive egress, IPv4 charges
- **Best for:** Enterprises needing AWS integration, global scale

### Azure (Global)
- **Pros:** Good ARM options (Ampere), Microsoft integration
- **Cons:** Complex pricing, expensive overall
- **Best for:** Microsoft shops, hybrid cloud scenarios

### GCP (Global)
- **Pros:** Good network, sustained use discounts
- **Cons:** Most expensive egress, complex pricing
- **Best for:** Google ecosystem integration, ML workloads

### Oracle Cloud (Global)
- **Pros:** **FREE Always Free tier** (4 OCPU/24GB ARM), cheap paid tier
- **Cons:** Capacity often unavailable, smaller community
- **Best for:** Zero-cost hobby/test clusters, ARM workloads

### Alibaba Cloud (Global, Asia-focused)
- **Pros:** **Highest free credits** ($300-450 for 12 months), strong Asia presence, competitive pricing in Asia regions
- **Cons:** Most expensive in Western regions, complex pricing, less documentation in English
- **Best for:** Asia-Pacific deployments, startups wanting longest credit runway

---

## Recommendations by Use Case

### 1. Budget Production Cluster (< $50/month)
**Hetzner CX33** or **OVHcloud VPS-1**
- Total: ~$30-43/month
- Best value, solid performance
- EU-only acceptable

### 2. Global Production Cluster ($100-150/month)
**Vultr High Performance** or **DigitalOcean Basic**
- Total: ~$95-115/month
- Multiple regions available
- Good developer experience

### 3. Enterprise/Compliance Requirements ($250+/month)
**AWS t4g.large** or **Azure D2ps v5**
- Total: ~$260-285/month
- Full ecosystem integration
- Compliance certifications

### 4. Zero-Cost Development/Testing
**Oracle Cloud Always Free**
- 4 OCPU (8 vCPU equivalent) + 24GB RAM
- Perfect for Talent Mesh MVP
- Capacity hunting required

---

## New Account Gifted Credits

**Strategy:** Start with a provider offering generous credits, exhaust them, then migrate to Hetzner for long-term cost efficiency. Since we're using Kubernetes with Terraform, migration is straightforward.

| Provider | Credits | Validity | Requirements | Notes |
|----------|---------|----------|--------------|-------|
| **Vultr** | $300 | 30 days | Credit card | [Signup](https://www.vultr.com/) - Best for quick testing |
| **Civo** | $250 | 1 month | Credit card within 21 days | [Signup](https://dashboard.civo.com/signup) - K8s-focused, free egress |
| **AWS** | $200 | 6 months | New account (post July 2025) | [Free Tier](https://aws.amazon.com/free/) - $100 on signup + $100 via activities |
| **GCP** | $300 | 90 days | Credit card | [Free Trial](https://cloud.google.com/free) - Best ML/AI integration |
| **Azure** | $200 | 30 days | Credit card | [Free Account](https://azure.microsoft.com/free/) - Plus 12-mo free services |
| **DigitalOcean** | $200 | 60 days | Credit card | Via referral links |
| **Linode** | $100 | 60 days | Credit card | [Signup](https://www.linode.com/) |
| **Scaleway** | €50 | 1 month | Credit card | Basic signup; startups can get up to €36,000 |
| **Oracle Cloud** | $300 | 30 days | Credit card | Plus Always Free tier (permanent) |
| **Alibaba** | $300-450 | 12 months | New account | [Free Trial](https://www.alibabacloud.com/campaign/free-trial) |

### Startup Programs (Higher Credits)

| Provider | Program | Credits | Requirements |
|----------|---------|---------|--------------|
| **AWS Activate** | Startup program | $1,000 - $100,000 | VC-backed or accelerator |
| **Azure for Startups** | Microsoft program | Up to $150,000 | Software startup, <$10K prior Azure credits |
| **GCP for Startups** | Google program | Up to $100,000 | VC-backed startup |
| **Scaleway Startup** | Early Stage | Up to €36,000 | Startup application |
| **OVHcloud Startup** | Scaleup tier | Up to €100,000 | Established startup |

### Credit Duration Analysis

How long can you run a 3-node K8s cluster on each provider's free credits?

| Provider | Free Credits | Monthly Cost | Months Possible | Credit Validity | **Usable Months** |
|----------|--------------|--------------|-----------------|-----------------|-------------------|
| **Alibaba** | $300-450 | ~$299 | 1-1.5 months | 12 months | **1-1.5** |
| **GCP** | $300 | ~$284 | 1.1 months | 90 days (3 mo) | **1** |
| **AWS** | $200 | ~$263 | 0.8 months | 6 months | **0.8** |
| **Azure** | $200 | ~$282 | 0.7 months | 30 days | **0.7** |
| **Vultr** | $300 | ~$105 | 2.9 months | 30 days | **1** |
| **Civo** | $250 | ~$97 | 2.6 months | 1 month | **1** |
| **DigitalOcean** | $200 | ~$95 | 2.1 months | 60 days | **2** |
| **Linode** | $100 | ~$179 | 0.6 months | 60 days | **0.6** |
| **Oracle** | $300 | ~$96 | 3.1 months | 30 days | **1** |
| **Scaleway** | €50 (~$53) | ~$133 | 0.4 months | 1 month | **0.4** |

**Key insight:** Credit validity often expires before you can fully use them. Vultr ($300) sounds great but expires in 30 days - you can only use ~$105 worth.

### Optimal Credit Stacking Order

Stack providers by **usable months** (limited by validity), not total credits:

| Order | Provider | Usable Months | Cumulative Free Time |
|-------|----------|---------------|----------------------|
| 1 | **DigitalOcean** | 2 months | 2 months |
| 2 | **Alibaba** | 1.5 months | 3.5 months |
| 3 | **GCP** | 1 month | 4.5 months |
| 4 | **Vultr** | 1 month | 5.5 months |
| 5 | **Civo** | 1 month | 6.5 months |
| 6 | **Oracle** | 1 month | 7.5 months |
| 7 | **AWS** | 0.8 months | 8.3 months |
| 8 | **Azure** | 0.7 months | 9 months |
| | **Hetzner** | PAID (~$30/mo) | - |

**Maximum free runway: ~9 months** (using all providers sequentially)

**Note:** This requires creating accounts on 8 different providers and migrating between them. A more practical approach is to use 2-3 providers.

### Recommended Credit Strategy for Talent Mesh

**Option A: Practical Approach (3 providers)**
1. **DigitalOcean** ($200, 60 days) - 2 months usable
2. **GCP** ($300, 90 days) - 1 month usable
3. **Alibaba** ($300-450, 12 months) - 1.5 months usable
4. **Hetzner** (~$30/month) - Long-term PAID production

**Total free runway: ~4.5 months**, then ~$30/month on Hetzner.

**Option B: Maximum Free (8 providers, more migration effort)**
1. DigitalOcean → 2. Alibaba → 3. GCP → 4. Vultr → 5. Civo → 6. Oracle → 7. AWS → 8. Azure
4. **Hetzner** (~$30/month) - Long-term PAID production

**Total free runway: ~9 months**, then ~$30/month on Hetzner.

**Option C: Simple (Start paid immediately)**
1. **Hetzner** (~$30/month) - Skip the credit dance, start building

**Trade-off:** Pay $30/month from day 1, but zero migration overhead.

---

## Terraform Provider Availability

All providers in this comparison have official or community Terraform providers, enabling Infrastructure as Code:

| Provider | Terraform Provider | Registry | Maturity |
|----------|-------------------|----------|----------|
| **Contabo** | `contabo/contabo` | [Official](https://registry.terraform.io/providers/contabo/contabo/latest) | Good |
| **Hetzner** | `hetznercloud/hcloud` | [Official](https://registry.terraform.io/providers/hetznercloud/hcloud/latest) | Mature, well-documented |
| **Vultr** | `vultr/vultr` | [Official](https://registry.terraform.io/providers/vultr/vultr/latest) | Mature |
| **DigitalOcean** | `digitalocean/digitalocean` | [Official](https://registry.terraform.io/providers/digitalocean/digitalocean/latest) | Mature, excellent |
| **Linode** | `linode/linode` | [Official](https://registry.terraform.io/providers/linode/linode/latest) | Mature |
| **Scaleway** | `scaleway/scaleway` | [Official](https://registry.terraform.io/providers/scaleway/scaleway/latest) | Good |
| **OVHcloud** | `ovh/ovh` | [Official](https://registry.terraform.io/providers/ovh/ovh/latest) | Good |
| **Civo** | `civo/civo` | [Official](https://registry.terraform.io/providers/civo/civo/latest) | Good, K8s-focused |
| **AWS** | `hashicorp/aws` | [Official](https://registry.terraform.io/providers/hashicorp/aws/latest) | Excellent, most mature |
| **Azure** | `hashicorp/azurerm` | [Official](https://registry.terraform.io/providers/hashicorp/azurerm/latest) | Excellent |
| **GCP** | `hashicorp/google` | [Official](https://registry.terraform.io/providers/hashicorp/google/latest) | Excellent |
| **Oracle Cloud** | `oracle/oci` | [Official](https://registry.terraform.io/providers/oracle/oci/latest) | Good |
| **Alibaba** | `aliyun/alicloud` | [Official](https://registry.terraform.io/providers/aliyun/alicloud/latest) | Good |

### Terraform Provider Features Comparison

| Provider | Compute | Block Storage | Object Storage | Load Balancer | DNS | K8s Cluster |
|----------|---------|---------------|----------------|---------------|-----|-------------|
| **Contabo** | ✓ | ✓ | ✓ | ✗ | ✗ | ✗ (self-managed) |
| **Hetzner** | ✓ | ✓ | ✓ | ✓ | ✓ | ✗ (self-managed) |
| **Vultr** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (VKE) |
| **DigitalOcean** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (DOKS) |
| **Linode** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (LKE) |
| **Civo** | ✓ | ✓ | ✗ | ✗ | ✓ | ✓ (native) |
| **OVHcloud** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (MKS) |
| **AWS** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (EKS) |
| **Azure** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (AKS) |
| **GCP** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (GKE) |
| **Oracle** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (OKE) |
| **Alibaba** | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ (ACK) |

### Migration Strategy with Terraform

Since all providers have Terraform support, migration path is:

```
1. Write provider-agnostic Terraform modules for K8s resources (Helm, kubectl)
2. Write provider-specific modules for infrastructure (VMs, storage, networking)
3. Use workspaces or separate state files per provider
4. Migration = destroy old provider + apply new provider + restore K8s state
```

**Key advantage:** Kubernetes workloads are portable. Only infrastructure provisioning changes between providers.

---

## Sources

### Pricing Pages
- [Contabo VPS Pricing](https://contabo.com/en/vps/)
- [Hetzner Cloud Pricing](https://www.hetzner.com/cloud)
- [Vultr Pricing](https://www.vultr.com/pricing/)
- [DigitalOcean Droplet Pricing](https://www.digitalocean.com/pricing/droplets)
- [Linode/Akamai Pricing](https://www.linode.com/pricing/)
- [Scaleway Virtual Instances Pricing](https://www.scaleway.com/en/pricing/virtual-instances/)
- [OVHcloud VPS Pricing](https://us.ovhcloud.com/vps/)
- [AWS EC2 Pricing](https://aws.amazon.com/ec2/pricing/on-demand/)
- [Azure VM Pricing](https://azure.microsoft.com/en-us/pricing/details/virtual-machines/series/)
- [GCP Compute Engine Pricing](https://cloud.google.com/compute/vm-instance-pricing)
- [Oracle Cloud Compute Pricing](https://www.oracle.com/cloud/compute/pricing/)
- [Alibaba Cloud ECS Pricing](https://www.alibabacloud.com/en/product/ecs-pricing-list/en)

### Benchmark Sources
- [VPSBenchmarks.com](https://www.vpsbenchmarks.com/)
- [Geekbench Browser](https://browser.geekbench.com/)
- [EC2Instances.info](https://instances.vantage.sh/)

### Storage Pricing
- [Hetzner Object Storage](https://www.hetzner.com/storage/object-storage/)
- [Hetzner Block Storage](https://docs.hetzner.com/cloud/volumes/overview/)
- [DigitalOcean Spaces Pricing](https://www.digitalocean.com/pricing/spaces-object-storage)
- [Vultr Object Storage](https://www.vultr.com/products/object-storage/)
- [AWS S3 Pricing](https://aws.amazon.com/s3/pricing/)
- [AWS EBS Pricing](https://aws.amazon.com/ebs/pricing/)

### Load Balancer Pricing
- [Hetzner Load Balancer](https://www.hetzner.com/cloud/load-balancer/)
- [DigitalOcean Load Balancer](https://www.digitalocean.com/pricing/load-balancers)
- [AWS ELB Pricing](https://aws.amazon.com/elasticloadbalancing/pricing/)
- [Oracle Cloud Load Balancer](https://www.oracle.com/cloud/networking/flexible-load-balancer/pricing/)

### Egress/Bandwidth Pricing
- [Cloud Egress Comparison - GetDeploying](https://getdeploying.com/reference/data-egress)
- [AWS IPv4 Pricing](https://aws.amazon.com/blogs/aws/new-aws-public-ipv4-address-charge-public-ip-insights/)

### Free Credits & Trials
- [AWS Free Tier](https://aws.amazon.com/free/)
- [AWS Free Tier 2025 Changes](https://aws.amazon.com/blogs/aws/aws-free-tier-update-new-customers-can-get-started-and-explore-aws-with-up-to-200-in-credits/)
- [GCP Free Trial](https://cloud.google.com/free)
- [Azure Free Account](https://azure.microsoft.com/free/)
- [Civo Signup](https://dashboard.civo.com/signup)
- [Vultr Free Credits](https://www.vultr.com/)
- [Scaleway Startup Program](https://www.scaleway.com/en/startup-program/)
- [Cloud Free Tier Comparison (GitHub)](https://github.com/cloudcommunity/Cloud-Free-Tier-Comparison)

### Terraform Providers
- [Terraform Registry](https://registry.terraform.io/)
- [Contabo Provider](https://registry.terraform.io/providers/contabo/contabo/latest)
- [Hetzner Provider](https://registry.terraform.io/providers/hetznercloud/hcloud/latest)
- [Vultr Provider](https://registry.terraform.io/providers/vultr/vultr/latest)
- [DigitalOcean Provider](https://registry.terraform.io/providers/digitalocean/digitalocean/latest)
- [Civo Provider](https://registry.terraform.io/providers/civo/civo/latest)
- [AWS Provider](https://registry.terraform.io/providers/hashicorp/aws/latest)
- [Azure Provider](https://registry.terraform.io/providers/hashicorp/azurerm/latest)
- [GCP Provider](https://registry.terraform.io/providers/hashicorp/google/latest)
- [Oracle Provider](https://registry.terraform.io/providers/oracle/oci/latest)
- [Alibaba Provider](https://registry.terraform.io/providers/aliyun/alicloud/latest)

---

*Note: Prices are approximate and subject to change. Always verify current pricing on provider websites before making decisions. Currency conversions based on January 2026 rates.*
