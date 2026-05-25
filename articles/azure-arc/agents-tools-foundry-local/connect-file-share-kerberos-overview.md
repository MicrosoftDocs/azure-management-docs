---
title: Network File System (NFS) with Kerberos authentication overview for Agents and Tools with Foundry Local
description: "Learn about Network File System (NFS) data source connectivity with Kerberos (krb5p) authentication for disconnected on-premises deployments of Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/25/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#customer intent: As a platform administrator, I want to understand how NFS with Kerberos authentication works with Agents and Tools with Foundry Local so that I can securely connect on-premises file shares in disconnected environments.
---

# Network File System (NFS) with Kerberos authentication overview for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local can ingest documents from an on-premises Network File System (NFS) file share by using Kerberos authentication (`krb5p`). Instead of passing a UID/GID to access files, Agents and Tools with Foundry Local uses your Active Directory infrastructure to authenticate securely, with full encryption of data in transit.

This article applies to Agents and Tools with Foundry Local on Azure Local (Arc-enabled Kubernetes).

This article explains the Kerberos architecture, ingestion flow, prerequisite scope, and portal configuration fields so that you can plan a production-ready deployment.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## How it works

Kerberos configuration lives on the Kubernetes worker node, not inside the pod. The pod reads files from a mounted directory and isn't aware of Kerberos. The node kernel service (`rpc.gssd`) handles ticket acquisition and encrypted NFS communication.

:::image type="content" source="media/connect-file-share-kerberos-overview/kerberos-architecture.png" alt-text="Diagram showing the architecture of Kerberos NFS authentication with Active Directory, Kubernetes worker node, ingestion pod, and NFS file server." lightbox="media/connect-file-share-kerberos-overview/kerberos-architecture.png" border="false":::

### Key design points

The Kerberos NFS architecture is designed around these principles:

- **No passwords stored**: Authentication uses a keytab file on the node.
- **Data encrypted in transit**: `krb5p` provides full NFS payload encryption.
- **Pod is Kerberos-unaware**: The kernel handles everything via `rpc.gssd`.
- **Continuous health monitoring**: A DaemonSet validates each node every 60 seconds and labels it ready or not ready.
- **Node affinity**: Ingestion pods only schedule on nodes that pass validation.

## What happens during ingestion

When you create an NFS data source with Kerberos authentication:

1. **Ingestion API** receives the request with `uid=0, gid=0` (sentinel values for Kerberos mode).
1. **Preflight check** queries for nodes with `edge-rag/kerberos-ready=true` and raises an error if no nodes are found.
1. **PV/PVC creation** creates a static PersistentVolume with `mount_options: ["sec=krb5p", "vers=4.1"]`.
1. **Pod scheduling** uses node affinity to land the pod on a `kerberos-ready=true` node.
1. **Mount**: Kubelet on the node triggers the NFS mount, and `rpc.gssd` intercepts and obtains a Kerberos ticket from the keytab.
1. **File access**: The pod reads `/mnt/data` as a normal directory with no Kerberos code in the pod.

## Prerequisites checklist

Before you install Agents and Tools with Foundry Local with Kerberos enabled, make sure you have:

- A healthy Azure Local Arc-enabled Kubernetes cluster with worker nodes ready for ingestion.
- Active Directory, DNS, and NTP configured for Kerberos workloads.
- Kerberos client and NFS packages installed and validated on each worker node.
- Keytab and SPN configuration completed for both worker nodes and the NFS server.
- Network connectivity from worker nodes to domain controllers, DNS, NTP, and NFS server endpoints.

For full setup steps and command-level validation, see [Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local](connect-file-share-kerberos-setup.md).

## Portal field reference

When you install Agents and Tools with Foundry Local via the Azure portal, the **Data Source Connection / Authentication** section includes the following Kerberos fields:

| Portal field | Helm key | Type | Default | Description |
|---|---|---|---|---|
| **Enable Kerberos** | `kerberos.enabled` | Toggle | `false` | Enables Kerberos authentication for NFS data sources. When enabled, the installer validates that at least one node is labeled `kerberos-provisioned=true`. |
| **Service principal name (SPN)** | `kerberos.spn` | Text (required when enabled) | _(empty)_ | The Kerberos SPN for NFS authentication. Format: `nfs/<service_account>@<REALM>`. Example: `nfs/edgerag-svc@CONTOSO.COM`. Must match the principal in the keytab deployed on your nodes. |

The SPN field is required when Kerberos is enabled. The installation fails with a template error if left empty.

When you set `kerberos.enabled` to `true`, Agents and Tools with Foundry Local:

1. Deploys a Kerberos Validator DaemonSet on every node (health checks every 60 seconds).
1. Runs a preinstall validation hook (checks for labeled nodes).
1. Creates a dedicated `kerberos-ingestion-sa` ServiceAccount with PV/PVC permissions.
1. Uses PVC-based NFS mounts with `sec=krb5p,vers=4.1` (instead of inline NFS volumes).
1. Schedules ingestion pods only on `kerberos-ready=true` nodes.

The system injects the `kerberos.spn` value into the ingestion API pod as the `KERBEROS_SPN` environment variable. It uses this value for logging and validation. The keytab on the node handles Kerberos authentication.

## Frequently asked questions

**Does the pod need any Kerberos libraries or configuration?**
No. The pod is unaware of Kerberos. The `rpc.gssd` service on the node handles all authentication. The pod reads files from a mounted directory.

**Can I use an IP address for the NFS server?**
No. Kerberos requires a hostname for SPN construction. Use the NFS server's FQDN (for example, `nfs-server.contoso.com`).

**What if my NFS server only supports NFSv3?**
NFSv3 doesn't support Kerberos authentication. You must upgrade to NFSv4.1 or later, or use UID/GID authentication instead.

**Can I mix Kerberos and UID/GID data sources?**
Yes. When `kerberos.enabled=true`, you can still create UID/GID-based NFS data sources by providing nonzero UID and GID values. Kerberos is used only when the authentication method is set to **Kerberos** in the UI (which sends UID=0, GID=0 as sentinel values).

**What happens if a node loses its keytab or rpc.gssd stops?**
The DaemonSet validator detects the issue within 60 seconds and sets the node to `kerberos-ready=false`. Ingestion pods aren't scheduled on that node. Existing pods on the node might fail their next NFS mount. Files are redelivered from Redis and processed by a healthy node.

**How do I remove a node from Kerberos workloads?**
Remove the provisioned label: `kubectl label node <node_name> edge-rag/kerberos-provisioned-`. The DaemonSet continues to run, but the node isn't considered for new ingestion workloads once `kerberos-ready` is set to false.

## Related content

- [Set up Kerberos authentication](connect-file-share-kerberos-setup.md)
- [NFS with Kerberos reference](connect-file-share-kerberos-reference.md)
