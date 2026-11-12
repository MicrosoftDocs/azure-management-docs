---
title: NFS with Kerberos authentication overview
description: "Learn how Network File System (NFS) data sources use Kerberos authentication in disconnected deployments of Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#customer intent: As a platform administrator, I want to understand how NFS with Kerberos authentication works with Agents and Tools with Foundry Local so that I can securely connect on-premises file shares in disconnected environments.
---

# NFS with Kerberos authentication overview

Agents and Tools with Foundry Local can ingest documents from an on-premises Network File System (NFS) file share by using Kerberos authentication (`krb5p`). Instead of passing a UID/GID to access files, the extension uses your Active Directory infrastructure to authenticate securely, with full encryption of data in transit.

This article applies to Agents and Tools with Foundry Local on Azure Local (Arc-enabled Kubernetes).

This article explains the Kerberos architecture, ingestion flow, prerequisite scope, and portal configuration fields so that you can plan a production-ready deployment.

## How it works

Kerberos configuration lives on the Kubernetes worker node, not inside the pod. The pod reads files from a mounted directory and isn't aware of Kerberos. The node kernel service (`rpc.gssd`) handles ticket acquisition and encrypted NFS communication.

:::image type="content" source="media/connect-file-share-kerberos-overview/kerberos-architecture.png" alt-text="Diagram showing the architecture of Kerberos NFS authentication with Active Directory, Kubernetes worker node, ingestion pod, and NFS file server." lightbox="media/connect-file-share-kerberos-overview/kerberos-architecture.png" border="false":::

### Key design points

The Kerberos NFS architecture is designed around these principles:

- **No passwords stored**: Authentication uses a keytab file on the node.
- **Data encrypted in transit**: `krb5p` provides full NFS payload encryption.
- **Pod is Kerberos-unaware**: The kernel handles everything via `rpc.gssd`.
- **Limited readiness monitoring**: A DaemonSet checks for `krb5.conf`, a nonempty keytab, and KDC reachability. The default interval is 60 seconds and can be changed.
- **Node affinity**: Ingestion pods only schedule on nodes that pass validation.

## What happens during ingestion

When you create an NFS data source with Kerberos authentication:

1. **Ingestion API** receives the request with `uid=0, gid=0` (sentinel values for Kerberos mode).
1. **Preflight check** queries for nodes with `edge-rag/kerberos-ready=true` and raises an error if it doesn't find any nodes.
1. **PV/PVC creation** creates a static PersistentVolume with `mount_options: ["sec=krb5p", "vers=4.1"]`.
1. **Pod scheduling** uses node affinity to land the pod on a `kerberos-ready=true` node.
1. **Mount**: Kubelet on the node triggers the NFS mount, and `rpc.gssd` intercepts and obtains a Kerberos ticket from the keytab.
1. **File access**: The pod reads `/mnt/nfs/share` as a normal directory with no Kerberos code in the pod.

## Prerequisites summary

Before you install Agents and Tools with Foundry Local with Kerberos enabled, ensure you have:

- A healthy Azure Arc-enabled Kubernetes cluster with worker nodes ready for ingestion.
- Active Directory, DNS, and NTP configured for Kerberos workloads.
- Kerberos client and NFS packages installed and validated on each worker node.
- Keytab and SPN configuration completed for both worker nodes and the NFS server.
- Network connectivity from worker nodes to domain controllers, DNS, NTP, and NFS server endpoints.

For full setup steps and command-level validation, see [Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local](connect-file-share-kerberos-setup.md).

## Portal field reference

In the Azure portal, go to **Components**, set **Enable indexed sources** to **Enabled**, and select **Configure** for **Indexed sources**. Choose the processing mode, and then use these Kerberos fields:

| Portal field | Helm key | Type | Default | Description |
|---|---|---|---|---|
| **Use Kerberos authentication for NFS** | `kerberos.enabled` | Checkbox | Cleared | Enables Kerberos authentication for NFS data sources. When enabled, the installer validates that at least one node is labeled `kerberos-provisioned=true`. |
| **Kerberos service principal name (SPN)** | `kerberos.spn` | Text (required when enabled) | _(empty)_ | The Kerberos SPN for NFS authentication. Format: `service/host@REALM` with an uppercase realm (for example, `nfs/edgerag-svc@CONTOSO.COM`). Must match a principal available to `rpc.gssd` on the worker nodes. |

The SPN field is required when Kerberos is enabled. Kerberos settings are applied only when indexed sources are enabled.

When you set `kerberos.enabled` to `true`, Agents and Tools with Foundry Local:

1. Deploys a Kerberos Validator DaemonSet on every node. The default check interval is 60 seconds.
1. Runs a preinstall validation hook that counts nodes where you set `edge-rag/kerberos-provisioned=true`.
1. Creates a dedicated `kerberos-ingestion-sa` ServiceAccount with PV/PVC permissions.
1. Uses PVC-based NFS mounts with `sec=krb5p,vers=4.1` (instead of inline NFS volumes).
1. Schedules ingestion pods only on `kerberos-ready=true` nodes.

The system injects the `kerberos.spn` value as the `KERBEROS_SPN` environment variable. The `kerberos-validator` DaemonSet uses the `@REALM` suffix of `KERBEROS_SPN` for KDC DNS fallback checks, and `ingestionapi` attaches `KERBEROS_SPN` to Kerberos error diagnostics. The keytab on the node handles Kerberos authentication.

## Frequently asked questions

**Does the pod need any Kerberos libraries or configuration?**
No. The pod is unaware of Kerberos. The `rpc.gssd` service on the node handles all authentication. The pod reads files from a mounted directory.

**Can I use an IP address for the NFS server?**
No. Kerberos requires a hostname for SPN construction. Use the NFS server's FQDN (for example, `nfs-server.contoso.com`).

**What if my NFS server only supports NFSv3?**
This product's Kerberos ingestion path requires NFSv4.1 with `sec=krb5p`. Use an NFSv4.1 export for Kerberos ingestion, or use UID/GID authentication for the supported non-Kerberos path. This requirement is a product constraint, not a general statement about all NFSv3 implementations.

**Can I mix Kerberos and UID/GID data sources?**
Yes. When `kerberos.enabled=true`, you can still create UID/GID-based NFS data sources by providing nonzero UID and GID values. Kerberos is used only when the authentication method is set to **Kerberos** in the UI (which sends UID=0, GID=0 as sentinel values).

**What happens if a node loses its keytab or rpc.gssd stops?**
The validator detects a missing or empty keytab on its next configured check and sets the node to `kerberos-ready=false`. It doesn't check whether `rpc.gssd` is running, run `kinit`, validate key material, check clock synchronization, or test an authenticated NFS mount. A false readiness label prevents new matching pods from scheduling on the node. It doesn't evict or move pods that are already running.

**How do I remove a node from Kerberos workloads?**
The `edge-rag/kerberos-provisioned=true` label is an installation-time customer assertion; removing it doesn't change runtime scheduling. For maintenance, cordon the node to stop new pods from scheduling, then drain or stop affected ingestion workloads through your normal cluster-maintenance process. Readiness-label changes alone don't migrate running pods.

## Related content

- [Set up Kerberos authentication](connect-file-share-kerberos-setup.md)
- [NFS with Kerberos reference](connect-file-share-kerberos-reference.md)
