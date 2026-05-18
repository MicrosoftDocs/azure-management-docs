---
title: Set Up NFS with Kerberos Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure Active Directory, Kerberos, and NFS with krb5p encryption for on-premises data ingestion with Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 05/17/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want to set up NFS with Kerberos authentication so that I can securely ingest on-premises data with Agents and Tools with Foundry Local.
---

# Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local

This article walks through the step-by-step setup for NFS with Kerberos (`krb5p`) authentication. Before you begin, review the [prerequisites checklist](connect-nfs-kerberos-overview.md#prerequisites-checklist).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Step 1: Verify your Azure Local cluster

Ensure your Arc-enabled Kubernetes cluster is operational.

```bash
# Verify cluster nodes
kubectl get nodes
# All worker nodes should show STATUS: Ready

# Verify Arc connection
az connectedk8s show \
  --name <cluster_name> \
  --resource-group <resource_group> \
  -o table
```

**Requirements:**

- Azure Local cluster deployed and operational
- Arc-enabled Kubernetes connected to Azure
- At least one worker node available for Kerberos workloads

> [!IMPORTANT]
> New or replaced nodes don't automatically have Kerberos prerequisites (keytab, `rpc.gssd`, domain join). We recommend a fixed node pool for Kerberos workloads. If autoscaling is required, see [Adding new nodes](connect-nfs-kerberos-reference.md#add-new-nodes).

## Step 2: Join worker nodes to Active Directory

All Kubernetes worker nodes that run NFS ingestion must be joined to your AD domain.

### Install required packages

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install -y realmd sssd sssd-tools adcli packagekit

# RHEL / CentOS
sudo yum install -y realmd sssd sssd-tools adcli oddjob oddjob-mkhomedir
```

### Discover and join the domain

```bash
# Discover the domain
sudo realm discover <YOUR_DOMAIN>

# Join the domain (prompts for AD admin credentials)
sudo realm join --verbose <YOUR_DOMAIN> -U <admin_user>@<YOUR_DOMAIN>
```

### Verify domain join

```bash
# Verify membership
realm list
# Expected output includes:
#   configured: kerberos-member

# Verify SSSD is running
sudo systemctl status sssd

# Verify machine keytab was created
sudo klist -kt /etc/krb5.keytab
# Should list host/<hostname>@<YOUR_DOMAIN> entries
```

### Alternative: Manual machine SPN (without full domain join)

If full domain join isn't possible, create a machine SPN manually on the domain controller:

```powershell
# On Windows Domain Controller (PowerShell)
New-ADComputer -Name "<NODE_NAME>" -SamAccountName "<NODE_NAME>$" `
  -Path "OU=Kubernetes,DC=contoso,DC=com"

setspn -A nfs/<node_fqdn> <NODE_NAME>

ktpass /out <node_name>.keytab `
  /princ host/<node_fqdn>@<YOUR_REALM> `
  /mapuser <NODE_NAME>$ /pass * `
  /crypto AES256-SHA1 /ptype KRB5_NT_PRINCIPAL
```

```bash
# On the Linux node -  copy the keytab
sudo scp <admin_user>@<domain_controller>:/path/<node_name>.keytab /etc/krb5.keytab
sudo chmod 600 /etc/krb5.keytab
sudo chown root:root /etc/krb5.keytab
```

## Step 3: Install and configure Kerberos client

### Install packages

```bash
# Ubuntu / Debian
sudo apt update && sudo apt install -y krb5-user libpam-krb5

# RHEL / CentOS
sudo yum install -y krb5-workstation krb5-libs
```

### Configure /etc/krb5.conf

Create or edit `/etc/krb5.conf` on every worker node:

```ini
[libdefaults]
    default_realm = <YOUR_REALM>
    dns_lookup_kdc = true
    dns_lookup_realm = true
    ticket_lifetime = 24h
    renew_lifetime = 7d
    forwardable = true
    rdns = true

[realms]
    <YOUR_REALM> = {
        kdc = <kdc_server_1>
        kdc = <kdc_server_2>
        admin_server = <kdc_server_1>
        default_domain = <your_domain_lowercase>
    }

[domain_realm]
    .<your_domain_lowercase> = <YOUR_REALM>
    <your_domain_lowercase> = <YOUR_REALM>
```

Replace the placeholders with your values:

| Placeholder | Example |
|---|---|
| `<YOUR_REALM>` (uppercase) | `CORP.EXAMPLE.COM` |
| `<kdc_server_1>` | `ad-dc1.corp.example.com` |
| `<kdc_server_2>` | `ad-dc2.corp.example.com` (optional, for HA) |
| `<your_domain_lowercase>` | `corp.example.com` |

### Validate Kerberos configuration

```bash
# Test KDC reachability
kinit <admin_user>@<YOUR_REALM>
# Enter password when prompted

# Verify ticket
klist
# Should show: krbtgt/<YOUR_REALM>@<YOUR_REALM>

# Clean up
kdestroy
```

## Step 4: Create service principal and deploy keytab

Agents and Tools with Foundry Local uses a service account keytab for non-interactive Kerberos authentication. No passwords are stored.

### Create the service principal in AD

**Option A -  Using kadmin (Linux):**

```bash
sudo kadmin -p <admin_user>@<YOUR_REALM> <<EOF
addprinc -randkey nfs/<service_account>@<YOUR_REALM>
ktadd -k /tmp/edgerag-nfs.keytab nfs/<service_account>@<YOUR_REALM>
EOF
```

**Option B -  Using PowerShell (Windows domain controller):**

```powershell
# Create service account
New-ADUser -Name "<service_account_name>" `
  -UserPrincipalName "<service_account_name>@<your_domain>" `
  -PasswordNeverExpires $true `
  -CannotChangePassword $true `
  -Enabled $true `
  -AccountPassword (ConvertTo-SecureString "<temp_password>" -AsPlainText -Force)

# Create SPN
setspn -A nfs/<service_account> <service_account_name>

# Export keytab
ktpass /out edgerag-nfs.keytab `
  /princ nfs/<service_account>@<YOUR_REALM> `
  /mapuser <service_account_name>@<your_domain> `
  /pass * `
  /crypto AES256-SHA1 `
  /ptype KRB5_NT_PRINCIPAL
```

### Deploy keytab to all worker nodes

```bash
for NODE in <node_1> <node_2> <node_3>; do
  echo "Deploying keytab to $NODE..."
  scp /tmp/edgerag-nfs.keytab ${NODE}:/etc/krb5.keytab
  ssh ${NODE} "sudo chmod 600 /etc/krb5.keytab && sudo chown root:root /etc/krb5.keytab"
  echo "  Done."
done
```

### Verify keytab on each node

```bash
# List principals in keytab
sudo klist -kt /etc/krb5.keytab

# Test non-interactive authentication
sudo kinit -kt /etc/krb5.keytab nfs/<service_account>@<YOUR_REALM>
sudo klist
# Should show valid TGT

# Clean up
sudo kdestroy
```

> [!NOTE]
> Note the SPN (for example, `nfs/edgerag-svc@CONTOSO.COM`). You enter this value in the **Service Principal Name** field during installation.

## Step 5: Install NFS client and enable rpc-gssd

`rpc-gssd` is the kernel-level Kerberos daemon that intercepts NFS mount requests and acquires tickets by using the keytab. This must be running on every worker node.

```bash
# Install NFS client
sudo apt install -y nfs-common       # Ubuntu / Debian
sudo yum install -y nfs-utils        # RHEL / CentOS

# Enable and start rpc-gssd
sudo systemctl enable rpc-gssd
sudo systemctl start rpc-gssd

# Verify
sudo systemctl status rpc-gssd
# Should show: active (running)
```

## Step 6: Validate NFS Kerberos mount

Test an actual NFS mount with Kerberos to verify the entire chain works end-to-end:

```bash
# Create a test mount point
sudo mkdir -p /mnt/nfs-kerberos-test

# Mount with Kerberos
sudo mount -t nfs4 -o sec=krb5p,vers=4.1 \
  <nfs_server_fqdn>:<export_path> \
  /mnt/nfs-kerberos-test

# Verify mount options
mount | grep nfs-kerberos-test
# Expected: ... type nfs4 (... sec=krb5p ...)

# Test file access
ls /mnt/nfs-kerberos-test/

# Clean up
sudo umount /mnt/nfs-kerberos-test
sudo rmdir /mnt/nfs-kerberos-test
```

> [!IMPORTANT]
> If this mount fails, Agents and Tools with Foundry Local also fails. Resolve the issue before proceeding. See [Troubleshooting](connect-nfs-kerberos-reference.md#troubleshooting).

### NFS server requirements

Your NFS server must be configured to accept Kerberos-authenticated clients:

| Requirement | Details |
|---|---|
| NFSv4.1 or later | Required for Kerberos (NFSv3 doesn't support it). |
| `sec=krb5p` in export options | The export must allow `krb5p` security mode. |
| NFS server has its own SPN in AD | For example, `nfs/nfs-server@<YOUR_REALM>`. |
| Proper NFSv4 ID mapping | Should match client configuration. |

Example NFS server export (`/etc/exports`):

```
/exports/data  *.contoso.com(ro,sync,sec=krb5p,no_subtree_check)
```

> [!IMPORTANT]
> Agents and Tools with Foundry Local requires the NFS server to be specified by hostname (not IP address). Kerberos SPN construction by `rpc.gssd` relies on DNS -  using an IP address causes authentication to fail.

## Step 7: Configure DNS and time sync

### DNS -  Forward and reverse resolution required

Kerberos requires both forward and reverse DNS. Without proper DNS, ticket requests fail silently.

```bash
# Forward lookup -  NFS server
nslookup <nfs_server_fqdn>

# Reverse lookup -  NFS server (REQUIRED for Kerberos)
nslookup <nfs_server_ip>
# Must return the FQDN

# Forward lookup -  domain controllers
nslookup <domain_controller_fqdn>

# KDC SRV records
nslookup -type=SRV _kerberos._tcp.<your_domain>
```

### Time sync -  Clock skew must be less than 5 minutes

```bash
# Enable NTP
sudo timedatectl set-ntp true

# Verify
timedatectl status
# NTP synchronized: yes

# Check offset
chronyc tracking   # or: ntpq -p
```

## Step 8: Label nodes

Apply the Kerberos provisioning label to every prepared node. The pre-install hook checks for this label -  installation fails if no nodes have it.

```bash
# Label each prepared node
kubectl label node <node_name> edge-rag/kerberos-provisioned=true

# Verify
kubectl get nodes -l edge-rag/kerberos-provisioned=true
```

### Understanding the three-tier label system

Agents and Tools with Foundry Local uses three distinct node labels for Kerberos:

| Label | Set by | Meaning |
|---|---|---|
| `edge-rag/kerberos-provisioned=true` | You (manual `kubectl label`) | Your assertion that you prepared this node with `krb5.conf`, keytab, and `rpc.gssd`. |
| `edge-rag/kerberos-ready=true` or `false` | DaemonSet (automatic, every 60 seconds) | System verification that Kerberos prerequisites are currently healthy on this node. |
| `edge-rag/kerberos-reason=<REASON>` | DaemonSet (automatic) | If `ready=false`, the specific failure reason (for example, `KRB5_CONF_MISSING`, `KEYTAB_MISSING`, `KDC_UNREACHABLE`). |

The `provisioned` label is your assertion. The `ready` label is the system's live verification. Ingestion pods only schedule on nodes where `kerberos-ready=true`.

## Step 9: Run the validation script

Run this script on each prepared node before installing Agents and Tools with Foundry Local:

```bash
#!/bin/bash
# Agents and Tools with Foundry Local -  Kerberos prerequisites validation
set -euo pipefail

RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
NC='\033[0m'

PASS=0
FAIL=0
WARN=0

check() {
    local description="$1"
    local command="$2"
    local required="${3:-true}"

    if eval "$command" &>/dev/null; then
        echo -e "  ${GREEN}[PASS]${NC} $description"
        PASS=$((PASS + 1))
    elif [ "$required" = "true" ]; then
        echo -e "  ${RED}[FAIL]${NC} $description"
        FAIL=$((FAIL + 1))
    else
        echo -e "  ${YELLOW}[WARN]${NC} $description (optional)"
        WARN=$((WARN + 1))
    fi
}

echo "============================================================"
echo "  Kerberos prerequisites validation"
echo "  Node: $(hostname -f)"
echo "  Date: $(date -Iseconds)"
echo "============================================================"
echo ""

echo "1. Domain and identity"
check "Domain joined (realm list)" "realm list 2>/dev/null | grep -qi 'configured: kerberos-member'"
check "SSSD service running" "systemctl is-active sssd"
check "Machine hostname resolvable" "nslookup $(hostname -f)"
echo ""

echo "2. Kerberos configuration"
check "/etc/krb5.conf exists" "test -f /etc/krb5.conf"
check "/etc/krb5.conf has default_realm" "grep -q 'default_realm' /etc/krb5.conf"
check "/etc/krb5.keytab exists" "test -f /etc/krb5.keytab"
check "/etc/krb5.keytab is non-empty" "test -s /etc/krb5.keytab"
check "Keytab readable (klist -kt)" "klist -kt /etc/krb5.keytab"
check "kinit with keytab succeeds" \
  "kinit -kt /etc/krb5.keytab \$(klist -kt /etc/krb5.keytab 2>/dev/null | tail -1 | awk '{print \$NF}') && kdestroy"
echo ""

echo "3. NFS services"
check "nfs-common/nfs-utils installed" \
  "dpkg -l nfs-common 2>/dev/null | grep -q '^ii' || rpm -q nfs-utils 2>/dev/null"
check "rpc-gssd enabled" "systemctl is-enabled rpc-gssd"
check "rpc-gssd running" "systemctl is-active rpc-gssd"
echo ""

echo "4. DNS resolution"
REALM=$(grep 'default_realm' /etc/krb5.conf 2>/dev/null | awk '{print $NF}' | tr -d ' ')
DOMAIN=$(echo "$REALM" | tr '[:upper:]' '[:lower:]')
check "KDC SRV record resolvable" "nslookup -type=SRV _kerberos._tcp.$DOMAIN"
check "KDC hostname resolvable" \
  "grep 'kdc' /etc/krb5.conf | head -1 | awk -F= '{print \$2}' | xargs nslookup"
echo ""

echo "5. Time synchronization"
check "NTP synchronized" \
  "timedatectl status 2>/dev/null | grep -qi 'synchronized: yes\|ntp.*active'"
echo ""

echo "============================================================"
echo -e "  Results: ${GREEN}$PASS passed${NC}, ${RED}$FAIL failed${NC}, ${YELLOW}$WARN warnings${NC}"
if [ $FAIL -eq 0 ]; then
    echo -e "  ${GREEN}[PASS] Node is ready for Kerberos installation${NC}"
    echo ""
    echo "  Next step -  apply the label (if not already set):"
    echo "    kubectl label node $(hostname) edge-rag/kerberos-provisioned=true"
else
    echo -e "  ${RED}[FAIL] Node has $FAIL failing checks -  resolve before installation${NC}"
fi
echo "============================================================"
```

Save as `validate-kerberos-prereqs.sh` and run on each node:

```bash
chmod +x validate-kerberos-prereqs.sh
sudo ./validate-kerberos-prereqs.sh
```

## Install Agents and Tools with Foundry Local with Kerberos

### Azure portal (recommended)

1. In the Azure portal, navigate to your Azure Local cluster.
1. Go to **Extensions** > **Add** > **Agentic RAG**.
1. In the **Data Source Connection / Authentication** section:
   - Toggle **Enable Kerberos** to **On**.
   - Enter the **Service Principal Name (SPN)**: `nfs/<service_account>@<YOUR_REALM>` (must match the principal in the keytab deployed to nodes).
1. Complete the rest of the installation wizard.

### Azure CLI

```azurecli
az k8s-extension create \
  --name <extension_name> \
  --cluster-name <cluster_name> \
  --resource-group <resource_group> \
  --cluster-type connectedClusters \
  --extension-type microsoft.arc.rag \
  --configuration-settings \
    kerberos.enabled=true \
    kerberos.spn="nfs/<service_account>@<YOUR_REALM>" \
    kerberos.minNodes=1
```

| CLI parameter | Description | Default |
|---|---|---|
| `kerberos.enabled` | Enable Kerberos authentication. | `false` |
| `kerberos.spn` | Service Principal Name (required when enabled). | _(empty)_ |
| `kerberos.minNodes` | Minimum nodes with `kerberos-provisioned=true` label required for install to succeed. | `1` |

## Next step

> [!div class="nextstepaction"]
> [Reference and troubleshooting](connect-nfs-kerberos-reference.md)
