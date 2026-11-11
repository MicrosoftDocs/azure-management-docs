---
title: NFS with Kerberos reference and troubleshooting
description: "Verification steps, troubleshooting guide, network requirements, keytab rotation, and Helm values reference for NFS with Kerberos authentication."
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#customer intent: As a platform administrator, I want to verify, troubleshoot, and maintain NFS Kerberos authentication for Agents and Tools with Foundry Local so that file share ingestion stays reliable and secure.
---

# NFS with Kerberos reference and troubleshooting

This article provides verification steps, troubleshooting guidance, network requirements, operational procedures, and a Helm values reference for NFS with Kerberos authentication.

## Verification

After installation, verify that the Kerberos setup works.

### Check which nodes are Kerberos-ready

```bash
kubectl get nodes -l edge-rag/kerberos-ready=true
# Should list all prepared nodes
```

### Check for failed nodes and reasons

```bash
kubectl get nodes -l edge-rag/kerberos-ready=false \
  -o go-template='{{range .items}}{{.metadata.name}}{{"\t"}}{{index .metadata.annotations "edge-rag/kerberos-reason"}}{{"\n"}}{{end}}'
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
| **Install fails:** "Error: Not enough Kerberos-provisioned nodes." | No nodes are labeled. | Run `kubectl label node <node_name> edge-rag/kerberos-provisioned=true`. Complete all prerequisite steps first. |
| **Node shows `kerberos-ready=false`** with reason `KRB5_CONF_MISSING` | `/etc/krb5.conf` doesn't exist on the node. | Create the file per [Step 3](connect-file-share-kerberos-setup.md#step-3-install-and-configure-kerberos-client). |
| **Node shows `kerberos-ready=false`** with reason `KEYTAB_MISSING` or `KEYTAB_EMPTY` | `/etc/krb5.keytab` is missing or zero bytes. | Deploy the keytab per [Step 4](connect-file-share-kerberos-setup.md#step-4-create-a-service-principal-and-deploy-the-keytab). |
| **Node shows `kerberos-ready=false`** with reason `KDC_UNREACHABLE` | Can't reach the domain controller on port 88, or `kdc =` in `/etc/krb5.conf` includes a `:88` port suffix. | Ensure `kdc =` in `/etc/krb5.conf` is a hostname or IP address without a `:88` port suffix, and check firewall rules and DNS: `nc -zv <domain_controller> 88`. |
| **Node shows `kerberos-ready=false`** (`KDC_DNS_UNREACHABLE`) | DNS can't resolve the KDC hostname. | Check `/etc/resolv.conf` and `nslookup <domain_controller>`. |
| **Ingestion fails:** "No Kerberos-ready nodes available" | All nodes are `kerberos-ready=false`. | Check DaemonSet logs: `kubectl logs -n arc-rag -l app=kerberos-validator`. |
| **NFS mount fails:** `access denied by server` | Reverse DNS missing for NFS server, or NFS export doesn't allow `krb5p`. | Verify: `nslookup <nfs_server_ip>` returns FQDN. Check NFS export config. |
| **NFS mount fails when using IP address** | Kerberos requires hostname, not IP. | Use the NFS server's FQDN (for example, `nfs-server.contoso.com`). |
| **`kinit: Keytab contains no suitable keys`** | Service principal name (SPN) mismatch between keytab and request. | Run `klist -kt /etc/krb5.keytab` and verify that the SPN matches exactly. |
| **`Clock skew too great`** | Node time differs from KDC by more than 5 minutes. | Enable NTP: `sudo timedatectl set-ntp true`. |
| **NFS authentication fails but the node is `kerberos-ready=true`** | The validator doesn't check `rpc.gssd`, key validity, `kinit`, time synchronization, or an NFS read. | Check `systemctl status rpc-gssd`, validate the keytab with `kinit`, verify time synchronization, and repeat the authenticated mount/read test. |
| **Pod stuck in `Pending`** | No nodes match the `kerberos-ready=true` affinity. | Check node labels: `kubectl get nodes --show-labels \| grep kerberos`. |

## Network requirements

The following paths are required while the product uses NFS with Kerberos:

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| Worker nodes | Domain controller (KDC) | 88 | TCP/UDP | Kerberos authentication |
| Worker nodes | NFS file server | 2049 | TCP | NFS file access |
| Worker nodes | DNS server | 53 | TCP/UDP | DNS resolution |
| Worker nodes | NTP server | 123 | UDP | Time synchronization |

Domain join and administration can require additional paths:

| Purpose | Port | When required |
|---|---|---|
| Kerberos password change | 464 TCP/UDP | When your administration or rotation process changes passwords through this service. |
| LDAP / LDAPS | 389 / 636 TCP | For AD DS domain join, directory lookups, or administration that uses LDAP. |
| RPC portmapper | 111 TCP/UDP | For NFS versions or auxiliary RPC services that use portmapper. It isn't required by the product's fixed NFSv4.1 mount path. |

Test connectivity:

```bash
nc -zv <domain_controller> 88      # Kerberos
nc -zv <nfs_server_fqdn> 2049      # NFS
```

## Keytab rotation

Use a staged rotation. The readiness label only checks that the keytab is nonempty and the KDC is reachable; it doesn't validate the new key version, password, expiry, or authentication.

1. Generate the replacement principal material with your realm's supported administration process. Use `ktpass` or approved AD DS tooling for AD DS. The `kadmin` and `ktadd` commands apply only to an MIT Kerberos administration service.
1. Preserve the existing `/etc/krb5.keytab` and its required machine and host entries. Merge or install the new NFS principal through your approved keytab-management process.
1. Keep the old principal material valid during the overlap required by your realm's ticket-lifetime and key-version policy.
1. On every prepared node, run `klist -kt`, obtain fresh credentials with `kinit -kt`, and complete an NFSv4.1 mount and read with `sec=krb5p`.
1. Confirm new ingestion runs succeed before retiring old key material.

Don't rely on a fixed grace period or the `kerberos-ready=true` label as proof of successful rotation. Whether a running workload must be recreated depends on the node's `rpc.gssd` behavior and your Kerberos policy; validate it in your environment.

## Add new nodes

When adding a new worker node to the cluster:

```bash
# 1. Join the AD domain
sudo realm join <YOUR_DOMAIN> -U <admin_user>@<YOUR_DOMAIN>

# 2. Install packages
sudo apt install -y krb5-user nfs-common

# 3. Copy krb5.conf through a protected temporary directory
KRB5_TMP=$(mktemp -d)
chmod 700 "$KRB5_TMP"
scp <existing_node>:/etc/krb5.conf "$KRB5_TMP/krb5.conf"
chmod 600 "$KRB5_TMP/krb5.conf"
sudo install -o root -g root -m 644 "$KRB5_TMP/krb5.conf" /etc/krb5.conf
rm -rf "$KRB5_TMP"

# 4. Preserve the domain-join machine/host entries and merge the NFS principal
# Repeat the protected staging and ktutil merge procedure from setup Step 4.

# 5. Enable rpc-gssd
sudo systemctl enable --now rpc-gssd

# 6. Run the validation script with the intended principal and a known readable file
export KERBEROS_SPN="nfs/<service_account>@<YOUR_REALM>"
export NFS_SERVER="<nfs_server_fqdn>"
export NFS_EXPORT="<export_path>"
export NFS_TEST_FILE="<relative_path_to_known_readable_file>"
sudo --preserve-env=DOMAIN_JOINED,KERBEROS_SPN,NFS_SERVER,NFS_EXPORT,NFS_TEST_FILE \
  ./validate-kerberos-prereqs.sh

# 7. Label the node only after the authenticated mount and file read succeed
kubectl label node <new_node_name> edge-rag/kerberos-provisioned=true
```

The DaemonSet can then set `kerberos-ready=true` after its limited file-presence and KDC-reachability checks. That label isn't a substitute for the validation script's `kinit`, mount, and known-file read.

## Helm values reference

| Helm key | Type | Default | Description |
|---|---|---|---|
| `kerberos.enabled` | bool | `false` | Primary toggle for Kerberos NFS authentication. |
| `kerberos.spn` | string | `""` | Service principal name. Required when enabled. Example: `nfs/edgerag-svc@CONTOSO.COM`. |
| `kerberos.minNodes` | int | `1` | Minimum nodes with `kerberos-provisioned=true` label for pre-install validation. |
| `kerberos.checkInterval` | int | `60` | DaemonSet health check polling interval (seconds). |
| `skipKerberosValidations` | bool | `false` | Skip DaemonSet and pre-install hook. For development and testing only. Don't use in production. |

## Related content

- [NFS with Kerberos authentication overview](connect-file-share-kerberos-overview.md)
- [Set up Kerberos authentication](connect-file-share-kerberos-setup.md)
- [Configure NFS server](configure-nfs-server.md)
