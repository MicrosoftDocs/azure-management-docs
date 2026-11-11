---
title: Set Up NFS with Kerberos Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure Active Directory, Kerberos, and NFS with krb5p encryption for on-premises data ingestion with Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#customer intent: As a platform administrator, I want to set up NFS with Kerberos authentication so that I can securely ingest on-premises data with Agents and Tools with Foundry Local.
---

# Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local

This article shows how to configure NFS with Kerberos (`krb5p`) authentication for ingestion workloads in Agents and Tools with Foundry Local.

## Prerequisites

Before you begin, ensure you have:

- An Azure Local cluster deployed and operational.
- An Arc-enabled Kubernetes cluster connected to Azure.
- At least one available worker node for Kerberos workloads.
- Active Directory Domain Services (AD DS), including a key distribution center (KDC).
- An NFS server configured for NFSv4.1 (or later) with `sec=krb5p` export options.
- Required network paths and ports open between worker nodes, domain controllers, DNS/NTP services, and the NFS server. For details, see [Network requirements](connect-file-share-kerberos-reference.md#network-requirements).
- Reviewed the [NFS with Kerberos authentication overview](connect-file-share-kerberos-overview.md).

## Step 1: Verify your Azure Local cluster

Verify cluster connectivity and node readiness before you configure Kerberos.

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

New or replaced nodes don't automatically have Kerberos prerequisites (keytab, `rpc.gssd`, domain join). Use a fixed node pool for Kerberos workloads. If you need autoscaling, see [Add new nodes](connect-file-share-kerberos-reference.md#add-new-nodes).

## Step 2: Join worker nodes to Active Directory

Join all Kubernetes worker nodes that run NFS ingestion to your Active Directory domain.

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

### Alternative: Manually create a machine SPN (without full domain join)

If full domain join isn't possible, create a machine SPN manually on the domain controller.

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

As an unprivileged user, copy the keytab to a temporary path, and then install it by using elevated permissions. Don't copy the keytab directly into `/etc`.

```bash
KEYTAB_TMP=$(mktemp -d)
chmod 700 "$KEYTAB_TMP"
scp <admin_user>@<domain_controller>:/path/<node_name>.keytab "$KEYTAB_TMP/<node_name>.keytab"
chmod 600 "$KEYTAB_TMP/<node_name>.keytab"
sudo install -o root -g root -m 600 "$KEYTAB_TMP/<node_name>.keytab" /etc/krb5.keytab
rm -rf "$KEYTAB_TMP"
```

## Step 3: Install and configure Kerberos client

Each worker node requires Kerberos client tools and a configuration file (`/etc/krb5.conf`) to communicate securely with your Active Directory domain.

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

## Step 4: Create a service principal and deploy the keytab

Create the AD service principal and distribute its keytab to each ingestion worker node.

1. Create the service principal in AD.
1. Deploy the keytab to all worker nodes.
1. Verify keytab-based authentication on each node.

### Create the service principal in AD DS

Run the following commands on a Windows domain controller. The `kadmin`, `addprinc`, and `ktadd` commands apply to an MIT Kerberos administration service and aren't an alternative administration path for AD DS.

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

Domain-joined nodes already have machine and host principals in `/etc/krb5.keytab`. Preserve those entries. Back up the current keytab, copy the new keytab to a temporary path, and use your organization's supported keytab-management process to merge the NFS principal. The following example uses `ktutil`:

```bash
set -euo pipefail

# Securely transfer the exported file from the Windows administration host first.
KEYTAB_STAGE="$HOME/.edgerag-keytabs"
install -d -m 700 "$KEYTAB_STAGE"
trap 'rm -rf "$KEYTAB_STAGE"' EXIT
scp "<windows_admin>@<domain_controller>:C:/path/edgerag-nfs.keytab" \
  "$KEYTAB_STAGE/edgerag-nfs.keytab"
chmod 600 "$KEYTAB_STAGE/edgerag-nfs.keytab"
KERBEROS_SPN="nfs/<service_account>@<YOUR_REALM>"

for NODE in <node_1> <node_2> <node_3>; do
  echo "Deploying keytab to $NODE..."
  ssh "$NODE" 'umask 077; rm -rf /tmp/edgerag-keytab; mkdir -m 700 /tmp/edgerag-keytab'
  scp "$KEYTAB_STAGE/edgerag-nfs.keytab" "$NODE:/tmp/edgerag-keytab/edgerag-nfs.keytab"
  ssh "$NODE" "KERBEROS_SPN='$KERBEROS_SPN' bash -s" <<'REMOTE'
    set -e
    cleanup() {
      rm -rf /tmp/edgerag-keytab
      sudo rm -rf /var/tmp/edgerag-keytab-merge
    }
    trap cleanup EXIT

    chmod 600 /tmp/edgerag-keytab/edgerag-nfs.keytab
    BACKUP="/etc/krb5.keytab.backup.$(date +%Y%m%d%H%M%S)"
    sudo cp -a /etc/krb5.keytab "$BACKUP"
    ORIGINAL_PRINCIPALS=$(sudo klist -k /etc/krb5.keytab | awk 'NR > 3 {print $NF}' | sort -u)
    sudo rm -rf /var/tmp/edgerag-keytab-merge
    sudo install -d -o root -g root -m 700 /var/tmp/edgerag-keytab-merge
    sudo ktutil <<EOF
rkt /etc/krb5.keytab
rkt /tmp/edgerag-keytab/edgerag-nfs.keytab
wkt /var/tmp/edgerag-keytab-merge/krb5.keytab
quit
EOF

    sudo test -s /var/tmp/edgerag-keytab-merge/krb5.keytab
    sudo klist -k /var/tmp/edgerag-keytab-merge/krb5.keytab >/dev/null
    sudo klist -k /var/tmp/edgerag-keytab-merge/krb5.keytab |
      awk 'NR > 3 {print $NF}' |
      grep -Fx "$KERBEROS_SPN"
    if [ -n "$ORIGINAL_PRINCIPALS" ]; then
      while IFS= read -r principal; do
        sudo klist -k /var/tmp/edgerag-keytab-merge/krb5.keytab |
          awk 'NR > 3 {print $NF}' |
          grep -Fx "$principal" >/dev/null
      done <<< "$ORIGINAL_PRINCIPALS"
    fi

    sudo install -o root -g root -m 600 /var/tmp/edgerag-keytab-merge/krb5.keytab /etc/krb5.keytab
    sudo klist -k /etc/krb5.keytab | awk 'NR > 3 {print $NF}' | grep -Fx "$KERBEROS_SPN"
REMOTE
  echo "  Done."
done
```

Adding a principal can affect which identity `rpc.gssd` selects. The `kerberos.spn` extension setting doesn't force that selection. Validate the identity used by a real authenticated mount and read before you label the node.

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

Enter the SPN value (for example, `nfs/edgerag-svc@CONTOSO.COM`) in the **Kerberos service principal name (SPN)** field during installation.

## Step 5: Install NFS client and enable rpc-gssd

`rpc-gssd` is the kernel-level Kerberos daemon that intercepts NFS mount requests and gets tickets by using the keytab. It must run on every worker node.

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

Test an NFS mount with Kerberos to verify that the full authentication chain works.

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

If this mount fails, you can't continue deployment of the extension. Resolve the issue before you continue. For help, see [Troubleshooting](connect-file-share-kerberos-reference.md#troubleshooting).

### NFS server requirements

You must configure your NFS server to accept Kerberos-authenticated clients.

| Requirement | Details |
|---|---|
| NFSv4.1 | Required by this product's Kerberos ingestion path. It mounts with `vers=4.1,sec=krb5p`. |
| `sec=krb5p` in export options | The export must allow `krb5p` security mode. |
| NFS server has its own SPN in AD | For example, `nfs/nfs-server@<YOUR_REALM>`. |
| Proper NFSv4 ID mapping | Should match client configuration. |

Example NFS server export (`/etc/exports`):

```
/exports/data  *.contoso.com(ro,sync,sec=krb5p,no_subtree_check)
```

The extension requires you to specify the NFS server by hostname (not IP address). Kerberos SPN construction by `rpc.gssd` relies on DNS, so using an IP address causes authentication to fail.

## Step 7: Configure DNS and time sync

Kerberos authentication depends on correct DNS resolution and time synchronization between your nodes and domain controllers.

### Verify forward and reverse DNS resolution

Kerberos requires both forward and reverse DNS. Without proper DNS, ticket requests can fail.

```bash
# Forward lookup for the NFS server
nslookup <nfs_server_fqdn>

# Reverse lookup for the NFS server (required for Kerberos)
nslookup <nfs_server_ip>
# Must return the FQDN

# Forward lookup for domain controllers
nslookup <domain_controller_fqdn>

# KDC SRV records
nslookup -type=SRV _kerberos._tcp.<your_domain>
```

### Ensure clock skew is less than 5 minutes

```bash
# Enable NTP
sudo timedatectl set-ntp true

# Verify
timedatectl status
# NTP synchronized: yes

# Check offset
chronyc tracking   # or: ntpq -p
```

## Step 8: Understand node metadata

Don't apply the `edge-rag/kerberos-provisioned=true` label yet. Apply it only after the Step 9 validation script completes a successful `kinit`, authenticated mount, and known-file read.

### Understand the labels and annotation

The extension uses two node labels and one annotation:

| Metadata | Set by | Meaning |
|---|---|---|
| `edge-rag/kerberos-provisioned=true` | You (manual `kubectl label`) | Your assertion that you prepared this node with `krb5.conf`, keytab, and `rpc.gssd`. |
| `edge-rag/kerberos-ready=true` or `false` | DaemonSet (automatic) | Result of the limited file-presence and KDC-reachability checks. The default check interval is 60 seconds. |
| Annotation `edge-rag/kerberos-reason=<REASON>` | DaemonSet (automatic) | If `ready=false`, the specific validator failure reason. |

The `provisioned` label is your assertion and is counted by the preinstall hook. The runtime validator doesn't read it. The `ready` label doesn't prove that key material is valid, `rpc.gssd` is running, `kinit` succeeds, time is synchronized, or an NFS read works. Ingestion pods schedule only on nodes where `kerberos-ready=true`.

## Step 9: Run the validation script

Run this script on each prepared node before you install Agents and Tools with Foundry Local.

```bash
#!/bin/bash
# Agents and Tools with Foundry Local: Kerberos prerequisites validation
set -euo pipefail

DOMAIN_JOINED="${DOMAIN_JOINED:-true}"
: "${KERBEROS_SPN:?Set KERBEROS_SPN to the NFS principal in the keytab.}"
: "${NFS_SERVER:?Set NFS_SERVER to the NFS server FQDN.}"
: "${NFS_EXPORT:?Set NFS_EXPORT to the exported path.}"
: "${NFS_TEST_FILE:?Set NFS_TEST_FILE to a known readable file, relative to the export root.}"

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
if [ "$DOMAIN_JOINED" = "true" ]; then
    check "Domain joined (realm list)" "realm list 2>/dev/null | grep -qi 'configured: kerberos-member'"
    check "SSSD service running" "systemctl is-active sssd"
else
    echo "  [INFO] Keytab-only path selected; domain join and SSSD checks skipped."
fi
check "Machine hostname resolvable" "nslookup $(hostname -f)"
echo ""

echo "2. Kerberos configuration"
check "/etc/krb5.conf exists" "test -f /etc/krb5.conf"
check "/etc/krb5.conf has default_realm" "grep -q 'default_realm' /etc/krb5.conf"
check "/etc/krb5.keytab exists" "test -f /etc/krb5.keytab"
check "/etc/krb5.keytab is non-empty" "test -s /etc/krb5.keytab"
check "Keytab readable (klist -kt)" "klist -kt /etc/krb5.keytab"
check "kinit with keytab succeeds" \
  "kinit -kt /etc/krb5.keytab \"$KERBEROS_SPN\" && kdestroy"
echo ""

echo "3. NFS services"
check "nfs-common/nfs-utils installed" \
  "dpkg -l nfs-common 2>/dev/null | grep -q '^ii' || rpm -q nfs-utils 2>/dev/null"
check "rpc-gssd enabled" "systemctl is-enabled rpc-gssd"
check "rpc-gssd running" "systemctl is-active rpc-gssd"
echo ""

echo "4. DNS resolution"
REALM=$(grep -i '^[[:space:]]*default_realm[[:space:]]*=' /etc/krb5.conf 2>/dev/null | head -1 | awk -F'=' '{print $2}' | tr -d '[:space:]')
DOMAIN=$(echo "$REALM" | tr '[:upper:]' '[:lower:]')
KDC_HOST=$(grep -i '^[[:space:]]*kdc[[:space:]]*=' /etc/krb5.conf 2>/dev/null | head -1 | awk -F'=' '{print $2}' | tr -d '[:space:]' | cut -d: -f1)
check "KDC SRV record resolvable" "nslookup -type=SRV _kerberos._tcp.$DOMAIN"
if [ -n "$KDC_HOST" ]; then
    check "KDC hostname resolvable ($KDC_HOST)" "nslookup \"$KDC_HOST\""
else
    echo "  [INFO] No explicit kdc entry in /etc/krb5.conf; using DNS SRV discovery."
fi
echo ""

echo "5. Time synchronization"
check "NTP synchronized" \
  "timedatectl status 2>/dev/null | grep -qi 'synchronized: yes\|ntp.*active'"
echo ""

echo "6. Authenticated NFS read"
MOUNT_DIR=$(mktemp -d)
if mount -t nfs4 -o sec=krb5p,vers=4.1 "${NFS_SERVER}:${NFS_EXPORT}" "$MOUNT_DIR" \
    && test -r "$MOUNT_DIR/$NFS_TEST_FILE" \
    && head -c 1 "$MOUNT_DIR/$NFS_TEST_FILE" >/dev/null; then
    echo -e "  ${GREEN}[PASS]${NC} Authenticated NFSv4.1 mount and read"
    PASS=$((PASS + 1))
else
    echo -e "  ${RED}[FAIL]${NC} Authenticated NFSv4.1 mount and read"
    FAIL=$((FAIL + 1))
fi
if mountpoint -q "$MOUNT_DIR"; then
    umount "$MOUNT_DIR" || true
fi
rmdir "$MOUNT_DIR" || true
echo ""

echo "============================================================"
echo -e "  Results: ${GREEN}$PASS passed${NC}, ${RED}$FAIL failed${NC}, ${YELLOW}$WARN warnings${NC}"
if [ $FAIL -eq 0 ]; then
    echo -e "  ${GREEN}[PASS] Node is ready for Kerberos installation${NC}"
    echo ""
    echo "  Next step: apply the label (if not already set):"
    echo "    kubectl label node $(hostname) edge-rag/kerberos-provisioned=true"
else
    echo -e "  ${RED}[FAIL] Node has $FAIL failing checks; resolve them before installation${NC}"
    exit 1
fi
echo "============================================================"
```

Save the script as `validate-kerberos-prereqs.sh` and run it on each node:

```bash
chmod +x validate-kerberos-prereqs.sh

export DOMAIN_JOINED="true" # Set to false for the keytab-only path.
export KERBEROS_SPN="nfs/<service_account>@<YOUR_REALM>"
export NFS_SERVER="<nfs_server_fqdn>"
export NFS_EXPORT="<export_path>"
export NFS_TEST_FILE="<relative_path_to_known_readable_file>"

sudo --preserve-env=DOMAIN_JOINED,KERBEROS_SPN,NFS_SERVER,NFS_EXPORT,NFS_TEST_FILE \
  ./validate-kerberos-prereqs.sh
```

After the script succeeds, apply and verify the customer assertion label:

```bash
kubectl label node <node_name> edge-rag/kerberos-provisioned=true
kubectl get nodes -l edge-rag/kerberos-provisioned=true
```

## Deploy the extension with Kerberos enabled

After you complete this Kerberos setup, return to the [deployment prerequisites checklist](complete-prerequisites.md) and finish any remaining prerequisite steps. When you're ready to deploy, use [Deploy the extension for Agents and Tools with Foundry Local](deploy.md) and the Kerberos-specific values in this section.

During deployment:

- In the Azure portal, go to **Components**, set **Enable indexed sources** to **Enabled**, and select **Configure** for **Indexed sources**. Choose the processing mode, select **Use Kerberos authentication for NFS**, enter the **Kerberos service principal name (SPN)** from this article, and save.
- In Azure CLI, set `$enableKerberos = "true"` and `$kerberosSpn` in the CLI script (which set `kerberos.enabled=true` and `kerberos.spn`) in [Deploy the extension for Agents and Tools with Foundry Local](deploy.md?tabs=azure-cli).

If you don't plan to use Kerberos, skip this article and continue with the standard deployment flow in [Deploy the extension for Agents and Tools with Foundry Local](deploy.md).

## Confirm completion

You're ready to continue when:

- Nodes intended for ingestion are labeled `edge-rag/kerberos-provisioned=true`.
- Kerberos client and keytab validation pass on each worker node.
- A test NFS mount succeeds with `sec=krb5p`.
- You applied Kerberos-enabled installation settings successfully.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster](prepare-aks-cluster.md)

## Related content

- [NFS with Kerberos connection reference](connect-file-share-kerberos-reference.md)
