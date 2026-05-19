---
title: NFS with Kerberos Reference and Troubleshooting for Agents and Tools with Foundry Local
description: "Verification steps, troubleshooting guide, network requirements, keytab rotation, and Helm values reference for NFS with Kerberos authentication."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/17/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want to verify, troubleshoot, and maintain NFS Kerberos authentication for Agents and Tools with Foundry Local.
---

# NFS with Kerberos reference and troubleshooting for Agents and Tools with Foundry Local

This article provides verification steps, troubleshooting guidance, network requirements, operational procedures, and Helm values reference for NFS with Kerberos authentication.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Verification

After installation, verify the Kerberos setup is working.

### Check which nodes are Kerberos-ready

```bash
kubectl get nodes -l edge-rag/kerberos-ready=true
# Should list all your prepared nodes
```

### Check for failed nodes and reasons

```bash
kubectl get nodes -l edge-rag/kerberos-ready=false --show-labels \
  | tr ',' '\n' | grep kerberos
# Possible reasons: KRB5_CONF_MISSING, KEYTAB_MISSING, KEYTAB_EMPTY,
#                   KDC_UNREACHABLE, KDC_DNS_UNREACHABLE
```

### Check DaemonSet validator logs

```bash
kubectl logs -n arc-rag -l app=kerberos-validator --tail=20
```

### Test an NFS ingestion

1. Open the developer portal.
1. Create a new NFS data source.
1. Select **Kerberos** as the authentication method (UID/GID fields are hidden).
1. Enter the NFS server hostname and export path.
1. Start ingestion and verify files are processed without errors.

## Troubleshooting

| Symptom | Root cause | Fix |
|---|---|---|
| **Install fails:** "No nodes with kerberos-provisioned label" | No nodes are labeled. | Run `kubectl label node <node_name> edge-rag/kerberos-provisioned=true`. Complete all prerequisite steps first. |
| **Node shows `kerberos-ready=false`** with reason `KRB5_CONF_MISSING` | `/etc/krb5.conf` doesn't exist on the node. | Create the file per [Step 3](connect-file-share-kerberos-setup.md#step-3-install-and-configure-kerberos-client). |
| **Node shows `kerberos-ready=false`** with reason `KEYTAB_MISSING` or `KEYTAB_EMPTY` | `/etc/krb5.keytab` is missing or zero bytes. | Deploy the keytab per [Step 4](connect-file-share-kerberos-setup.md#step-4-create-service-principal-and-deploy-keytab). |
| **Node shows `kerberos-ready=false`** with reason `KDC_UNREACHABLE` | Can't reach the domain controller on port 88. | Check firewall rules and DNS: `nc -zv <domain_controller> 88`. |
| **Node shows `kerberos-ready=false`** with reason `KDC_DNS_UNREACHABLE` | DNS can't resolve KDC hostname. | Check `/etc/resolv.conf` and `nslookup <domain_controller>`. |
| **Ingestion fails:** "No Kerberos-ready nodes available" | All nodes are `kerberos-ready=false`. | Check DaemonSet logs: `kubectl logs -n arc-rag -l app=kerberos-validator`. |
| **NFS mount fails:** `access denied by server` | Reverse DNS missing for NFS server, or NFS export doesn't allow `krb5p`. | Verify: `nslookup <nfs_server_ip>` returns FQDN. Check NFS export config. |
| **NFS mount fails when using IP address** | Kerberos requires hostname, not IP. | Use the NFS server's FQDN (for example, `nfs-server.contoso.com`). |
| **`kinit: Keytab contains no suitable keys`** | SPN mismatch between keytab and request. | Run `klist -kt /etc/krb5.keytab` and verify SPN matches exactly. |
| **`Clock skew too great`** | Node time differs from KDC by more than 5 minutes. | Enable NTP: `sudo timedatectl set-ntp true`. |
| **`rpc.gssd` not running** | Service is stopped or failed to start. | Run `sudo systemctl restart rpc-gssd`. Check `journalctl -u rpc-gssd`. |
| **Pod stuck in `Pending`** | No nodes match the `kerberos-ready=true` affinity. | Check node labels: `kubectl get nodes --show-labels | grep kerberos`. |

## Network requirements

Ensure the following network paths are open from all Kubernetes worker nodes:

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| Worker nodes | Domain controller (KDC) | 88 | TCP/UDP | Kerberos authentication |
| Worker nodes | Domain controller | 464 | TCP/UDP | Kerberos password change |
| Worker nodes | Domain controller | 389 / 636 | TCP | LDAP / LDAPS |
| Worker nodes | NFS file server | 2049 | TCP | NFS file access |
| Worker nodes | NFS file server | 111 | TCP/UDP | RPC portmapper |
| Worker nodes | DNS server | 53 | TCP/UDP | DNS resolution |
| Worker nodes | NTP server | 123 | UDP | Time synchronization |

Test connectivity:

```bash
nc -zv <domain_controller> 88      # Kerberos
nc -zv <nfs_server_fqdn> 2049      # NFS
```

## Keytab rotation

When your AD policy requires password rotation (typically every 90 days):

```bash
# 1. Generate new keytab (while old keytab is still valid)
kadmin -p <admin_user>@<YOUR_REALM> \
  -q "ktadd -k /tmp/edgerag-nfs-new.keytab nfs/<service_account>@<YOUR_REALM>"

# 2. Deploy to all nodes
for NODE in <node_1> <node_2> <node_3>; do
  scp /tmp/edgerag-nfs-new.keytab ${NODE}:/etc/krb5.keytab
  ssh ${NODE} "sudo chmod 600 /etc/krb5.keytab && sudo chown root:root /etc/krb5.keytab"
done

# 3. Verify -  DaemonSet detects the new keytab within 60 seconds
kubectl get nodes -l edge-rag/kerberos-ready=true
```

> [!NOTE]
> After the AD password is reset, existing Kerberos TGTs remain valid for approximately 10 hours. Deploy the new keytab within this window for uninterrupted service. Running pods don't need to restart -  `rpc.gssd` re-reads the keytab on its next `kinit`.

## Add new nodes

When adding a new worker node to the cluster:

```bash
# 1. Join to AD domain
sudo realm join <YOUR_DOMAIN> -U <admin_user>@<YOUR_DOMAIN>

# 2. Install packages
sudo apt install -y krb5-user nfs-common

# 3. Copy krb5.conf from an existing node
scp <existing_node>:/etc/krb5.conf /etc/krb5.conf

# 4. Deploy keytab
scp <admin_machine>:/path/edgerag-nfs.keytab /etc/krb5.keytab
sudo chmod 600 /etc/krb5.keytab

# 5. Enable rpc-gssd
sudo systemctl enable --now rpc-gssd

# 6. Run validation script
sudo ./validate-kerberos-prereqs.sh

# 7. Label the node
kubectl label node <new_node_name> edge-rag/kerberos-provisioned=true

# The DaemonSet automatically validates and labels it kerberos-ready=true
```

## Helm values reference

| Helm key | Type | Default | Description |
|---|---|---|---|
| `kerberos.enabled` | bool | `false` | Master toggle for Kerberos NFS authentication. |
| `kerberos.spn` | string | `""` | Service Principal Name -  required when enabled. Example: `nfs/edgerag-svc@CONTOSO.COM`. |
| `kerberos.minNodes` | int | `1` | Minimum nodes with `kerberos-provisioned=true` label for pre-install validation. |
| `kerberos.checkInterval` | int | `60` | DaemonSet health check polling interval (seconds). |
| `skipKerberosValidations` | bool | `false` | Skip DaemonSet and pre-install hook. For development and test only -  don't use in production. |

## Related content

- [NFS with Kerberos authentication overview](connect-file-share-kerberos-overview.md)
- [Set up Kerberos authentication](connect-file-share-kerberos-setup.md)
- [Configure NFS server](configure-nfs-server.md)
