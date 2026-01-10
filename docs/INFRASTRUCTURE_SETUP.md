# Talent Mesh Infrastructure Setup

## Overview

This guide covers the complete Infrastructure as Code (IaC) setup for Talent Mesh, including:
- Contabo VPS provisioning
- Terraform configuration and deployment
- SOPS secrets encryption
- K3s cluster bootstrapping (3-node HA with etcd quorum)
- External Secrets Operator setup
- DNS-based failover configuration

> **Architecture Decision:** See [ADR-014: Contabo VPS Infrastructure](../09-adrs/ADR-014-CONTABO-VPS-INFRASTRUCTURE.md) and [ADR-039: Secrets Management - Infisical](../09-adrs/ADR-039-SECRETS-MANAGEMENT-INFISICAL.md)
>
> **Note:** SOPS is used for Terraform bootstrap secrets only. Runtime secrets use Infisical + ESO.

---

## Prerequisites

### Required Software

```bash
# Terraform
terraform >= 1.5.0

# SOPS (Secrets Operations)
sops >= 3.8.0

# age (encryption)
age >= 1.1.0

# kubectl
kubectl >= 1.28.0

# Helm
helm >= 3.13.0
```

### Installation (macOS)

```bash
brew install terraform sops age kubectl helm
```

### Installation (Linux)

```bash
# Terraform
wget -O- https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform

# SOPS
wget https://github.com/getsops/sops/releases/download/v3.8.1/sops-v3.8.1.linux.amd64 -O /usr/local/bin/sops
chmod +x /usr/local/bin/sops

# age
sudo apt install age

# kubectl
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl

# Helm
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
```

---

## Contabo Account Setup

### MVP Architecture (3 Nodes)

| Node | Role | Specs | Cost |
|------|------|-------|------|
| Node 1 | K3s Server + Worker | VPS 10: 4 vCPU, 8GB RAM, 200GB SSD | €4.50/month |
| Node 2 | K3s Server + Worker | VPS 10: 4 vCPU, 8GB RAM, 200GB SSD | €4.50/month |
| Node 3 | K3s Server + Worker | VPS 10: 4 vCPU, 8GB RAM, 200GB SSD | €4.50/month |
| **Total** | 3-node etcd quorum | 12 vCPU, 24GB RAM, 600GB SSD | **€13.50/month** |

### Step 1: Create Contabo Account

1. Go to https://contabo.com/
2. Create account and verify email
3. Add payment method

### Step 2: Generate API Credentials

1. Go to: https://my.contabo.com/api/v1
2. Navigate to **API** section
3. Click **Create API Credentials**
4. Save:
   - **Client ID**
   - **Client Secret**
   - **API User** (your email)
   - **API Password**

### Step 3: Note Region Options

| Region | Code | Latency to ME |
|--------|------|---------------|
| Germany (Nuremberg) | EU-DE-1 | 80-120ms |
| Germany (Munich) | EU-DE-2 | 80-120ms |
| UK | EU-UK-1 | 100-140ms |
| USA (St. Louis) | US-ST-1 | 200-250ms |
| USA (New York) | US-NY-1 | 180-220ms |
| Singapore | SIN | 80-120ms |
| **India (Mumbai)** | IN-MUM-1 | **40-60ms** |

> **Recommendation:** Start with EU-DE-1 (Germany) for development, migrate to IN-MUM-1 (India) when ME latency is critical.

---

## SOPS Secrets Encryption Setup

### Step 1: Generate Age Keypair

```bash
# Create directory
mkdir -p ~/.config/sops/age

# Generate keypair
age-keygen -o ~/.config/sops/age/keys.txt

# Output shows public key:
# Public key: age1ql3z7hjy54pw3hyww5ayyfg7zqgvc7w3j2elw8zmrj2kg5sfn9aqmcac8p
```

> **CRITICAL:** Backup `~/.config/sops/age/keys.txt` securely! This is the ONLY key needed to decrypt all secrets.

### Step 2: Configure SOPS

The `.sops.yaml` file in the repository root defines encryption rules:

```yaml
# .sops.yaml
creation_rules:
  # Terraform secrets
  - path_regex: terraform/.*\.tfvars$
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

  # Kubernetes secrets
  - path_regex: k8s/secrets/.*\.yaml$
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

### Step 3: Encrypt Terraform Secrets

```bash
cd terraform/environments/contabo

# Encrypt existing tfvars
sops --encrypt terraform.tfvars > terraform.tfvars.enc

# Verify encryption
cat terraform.tfvars.enc  # Should show encrypted values

# Remove unencrypted file
rm terraform.tfvars
```

### Step 4: Working with Encrypted Secrets

```bash
# Decrypt for terraform apply
sops --decrypt terraform.tfvars.enc > terraform.tfvars
terraform apply
rm terraform.tfvars  # Clean up after

# Or use sops exec-env (no temp file)
sops exec-env terraform.tfvars.enc 'terraform apply'

# Edit encrypted file directly
sops terraform.tfvars.enc  # Opens in $EDITOR, re-encrypts on save
```

---

## SSH Key Setup

```bash
# Generate SSH key for VM access
ssh-keygen -t ed25519 -C "talent-mesh" -f ~/.ssh/talent-mesh -N ""

# View public key (add to terraform.tfvars)
cat ~/.ssh/talent-mesh.pub
```

---

## Terraform Configuration

### Directory Structure

```
terraform/
├── .gitignore                    # Excludes .tfvars, .tfstate, .pem
├── README.md                     # Terraform usage guide
├── modules/
│   ├── contabo-vm/              # Contabo VPS provisioning
│   ├── hetzner-vm/              # Hetzner fallback
│   ├── k3s-cluster/             # K3s installation
│   ├── dns-failover/            # CoreDNS + Health Orchestrator
│   └── istio-ambient/           # Istio Ambient Mode setup
└── environments/
    ├── contabo/
    │   ├── main.tf              # 3-node cluster definition
    │   ├── variables.tf         # Variable declarations
    │   ├── terraform.tfvars.example  # Template (committed)
    │   └── terraform.tfvars.enc      # Encrypted secrets (committed)
    ├── contabo-india/           # Future ME latency optimization
    └── hetzner/                 # Fallback provider
```

### terraform.tfvars Structure

```hcl
# Contabo API Credentials
contabo_client_id     = "your-client-id"
contabo_client_secret = "your-client-secret"
contabo_api_user      = "your-email@example.com"
contabo_api_password  = "your-api-password"

# Cluster Configuration
region      = "EU-DE-1"  # Germany (Nuremberg)
node_count  = 3
product_id  = "V45"      # VPS 10: 4 vCPU, 8GB RAM, 200GB SSD

# SSH Key
ssh_public_key = "ssh-ed25519 AAAA... talent-mesh"

# K3s Configuration
k3s_token = "your-secure-k3s-token"  # Generate with: openssl rand -hex 32
```

### Deploy Infrastructure

```bash
cd terraform/environments/contabo

# Decrypt secrets
sops --decrypt terraform.tfvars.enc > terraform.tfvars

# Initialize Terraform
terraform init

# Preview changes
terraform plan

# Apply (creates 3 VPS instances with K3s)
terraform apply

# Clean up decrypted secrets
rm terraform.tfvars

# Save outputs
terraform output > outputs.txt
```

### Terraform Outputs

After successful apply:

```bash
# Get node IPs
terraform output node_ips

# Get SSH commands
terraform output ssh_commands

# Get kubeconfig command
terraform output kubeconfig_command
```

---

## K3s Cluster Access

### Get Kubeconfig

```bash
# SSH to node1 and get kubeconfig
ssh root@<node1_public_ip> 'cat /etc/rancher/k3s/k3s.yaml' > ~/.kube/talent-mesh.yaml

# Update server URL to use node1 public IP
sed -i "s/127.0.0.1/<node1_public_ip>/g" ~/.kube/talent-mesh.yaml

# Set as current context
export KUBECONFIG=~/.kube/talent-mesh.yaml

# Verify access
kubectl get nodes
```

### Expected Output

```
NAME           STATUS   ROLES                       AGE   VERSION
k3s-node-1     Ready    control-plane,etcd,master   5m    v1.28.x+k3s1
k3s-node-2     Ready    control-plane,etcd,master   4m    v1.28.x+k3s1
k3s-node-3     Ready    control-plane,etcd,master   3m    v1.28.x+k3s1
```

All 3 nodes should show `control-plane,etcd,master` roles (HA configuration).

---

## External Secrets Operator Setup

### Install ESO

```bash
# Add Helm repo
helm repo add external-secrets https://charts.external-secrets.io
helm repo update

# Install ESO
helm install external-secrets external-secrets/external-secrets \
  --namespace external-secrets \
  --create-namespace \
  --set installCRDs=true
```

### Create Secrets Namespace

```bash
kubectl create namespace secrets
```

### Deploy ClusterSecretStore

```yaml
# k8s/infrastructure/external-secrets/cluster-secret-store.yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: main-store
spec:
  provider:
    kubernetes:
      remoteNamespace: secrets
      server:
        caProvider:
          type: ConfigMap
          name: kube-root-ca.crt
          namespace: kube-system
          key: ca.crt
      auth:
        serviceAccount:
          name: external-secrets
          namespace: external-secrets
```

```bash
kubectl apply -f k8s/infrastructure/external-secrets/cluster-secret-store.yaml
```

### Create Base Secrets

```bash
# Decrypt and apply base secrets
sops --decrypt k8s/secrets/base-secrets.enc.yaml | kubectl apply -f -
```

---

## Istio Ambient Mode Setup

### Install Istio with Ambient Profile

```bash
# Download istioctl
curl -L https://istio.io/downloadIstio | sh -
cd istio-*
export PATH=$PWD/bin:$PATH

# Install Istio Ambient Mode
istioctl install --set profile=ambient -y

# Verify installation
kubectl get pods -n istio-system
```

### Expected Pods

```
NAME                                    READY   STATUS    RESTARTS   AGE
istio-cni-node-xxxxx                    1/1     Running   0          2m
istiod-xxxxxxxxx-xxxxx                  1/1     Running   0          2m
ztunnel-xxxxx                           1/1     Running   0          2m
ztunnel-yyyyy                           1/1     Running   0          2m
ztunnel-zzzzz                           1/1     Running   0          2m
```

### Label Namespace for Ambient Mesh

```bash
kubectl create namespace talent-mesh
kubectl label namespace talent-mesh istio.io/dataplane-mode=ambient
```

---

## DNS Failover Setup

### Deploy CoreDNS + Health Orchestrator

```bash
# Deploy DNS failover Helm chart
helm install dns-failover ./helm/dns-failover \
  --namespace dns-failover \
  --create-namespace \
  --set nodeIPs="{<node1_ip>,<node2_ip>,<node3_ip>}"
```

### Verify DNS Failover

```bash
# Check Health Orchestrator
kubectl logs -n dns-failover deployment/health-orchestrator

# Test DNS resolution
kubectl run -it --rm debug --image=busybox --restart=Never -- nslookup api.talentmesh.io
```

---

## Complete Setup Checklist

### One-Time Setup

- [ ] Install prerequisites (terraform, sops, age, kubectl, helm)
- [ ] Create Contabo account
- [ ] Generate Contabo API credentials
- [ ] Generate age keypair for SOPS
- [ ] **Backup age private key securely**
- [ ] Generate SSH keypair
- [ ] Configure terraform.tfvars
- [ ] Encrypt terraform.tfvars with SOPS

### Deploy Infrastructure

- [ ] `terraform init`
- [ ] `terraform plan`
- [ ] `terraform apply`
- [ ] Verify 3 VPS instances in Contabo dashboard
- [ ] Get kubeconfig from node1
- [ ] Verify `kubectl get nodes` shows 3 control-plane nodes

### Configure Kubernetes

- [ ] Install External Secrets Operator
- [ ] Create `secrets` namespace
- [ ] Deploy ClusterSecretStore
- [ ] Deploy base secrets (SOPS encrypted)
- [ ] Install Istio Ambient Mode
- [ ] Label namespaces for ambient mesh
- [ ] Deploy DNS failover components
- [ ] Verify ESO is syncing secrets

---

## Troubleshooting

### Terraform Errors

**Contabo Authentication Failed:**
```bash
# Verify API credentials
curl -X POST "https://auth.contabo.com/auth/realms/contabo/protocol/openid-connect/token" \
  -d "client_id=<client_id>" \
  -d "client_secret=<client_secret>" \
  -d "username=<api_user>" \
  -d "password=<api_password>" \
  -d "grant_type=password"
```

**VPS Creation Failed:**
```bash
# Check Contabo provider issues on GitHub
# Known issues: #36 (multi-server), #38 (import)
# Workaround: Apply with -parallelism=1
terraform apply -parallelism=1
```

### K3s Issues

**Node not joining cluster:**
```bash
# Check K3s logs on joining node
ssh root@<node_ip> 'journalctl -u k3s -f'

# Verify first node is accessible
ssh root@<node_ip> 'curl -k https://<first_node_ip>:6443'
```

**etcd quorum lost:**
```bash
# Check etcd status
kubectl get pods -n kube-system | grep etcd

# At least 2 of 3 nodes must be healthy
# If 2+ nodes are down, cluster is unavailable
```

**kubectl connection refused:**
```bash
# Verify server URL in kubeconfig
grep server ~/.kube/talent-mesh.yaml
# Should show: https://<node1_public_ip>:6443

# Test connectivity
curl -k https://<node1_public_ip>:6443
```

### SOPS Issues

**Cannot decrypt:**
```bash
# Verify age key exists
cat ~/.config/sops/age/keys.txt

# Check SOPS config
cat .sops.yaml

# Manually specify key
SOPS_AGE_KEY_FILE=~/.config/sops/age/keys.txt sops --decrypt file.enc
```

---

## Security Considerations

### Secrets Never in Git (Unencrypted)

| File | Git Status |
|------|------------|
| `terraform.tfvars` | **NEVER** commit |
| `terraform.tfvars.enc` | Safe to commit |
| `*.tfstate` | **NEVER** commit |
| `~/.ssh/talent-mesh` | Outside repo |
| `~/.config/sops/age/keys.txt` | Outside repo |

### Firewall Configuration

Each Contabo VPS should have firewall rules:

```bash
# Allow SSH
ufw allow 22/tcp

# Allow K3s API (only if needed externally)
ufw allow 6443/tcp

# Allow HTTPS (Istio Ingress)
ufw allow 443/tcp

# Allow HTTP (redirect to HTTPS)
ufw allow 80/tcp

# Allow TURN (STUNner)
ufw allow 3478/tcp
ufw allow 3478/udp

# Allow inter-node communication (K3s)
# From other node IPs only
ufw allow from <node2_ip>
ufw allow from <node3_ip>

# Enable firewall
ufw enable
```

### Backup Requirements

| Item | Backup Method | Recovery Impact |
|------|---------------|-----------------|
| Age private key | Password manager + physical | Cannot decrypt any secrets |
| SSH private key | Can regenerate, update tfvars | VM SSH access lost |
| Contabo API credentials | Can regenerate in dashboard | Terraform auth fails |

---

## References

- [ADR-014: Contabo VPS Infrastructure](../09-adrs/ADR-014-CONTABO-VPS-INFRASTRUCTURE.md)
- [ADR-039: Secrets Management - Infisical](../09-adrs/ADR-039-SECRETS-MANAGEMENT-INFISICAL.md)
- [ADR-013: Secrets with SOPS and ESO](../09-adrs/ADR-013-SECRETS-SOPS-ESO.md) (superseded, SOPS still used for bootstrap)
- [Infrastructure Agreements](./INFRASTRUCTURE_AGREEMENTS.md)
- [Contabo Terraform Provider](https://registry.terraform.io/providers/contabo/contabo/latest)
- [K3s Documentation](https://docs.k3s.io/)
- [Istio Ambient Mode](https://istio.io/latest/docs/ambient/)
- [SOPS Documentation](https://github.com/getsops/sops)
- [External Secrets Operator](https://external-secrets.io/)
- [Infisical Documentation](https://infisical.com/docs)

---

*Document Version: 2.1*
*Last Updated: 2026-01-09*
*Owner: Platform Team*
