---

title: Requirements for Agents and Tools with Foundry Local
description: "Learn how to deploy Agents and Tools with Foundry Local with this guide on hardware, software, networking, and configuration requirements for success."
ms.reviewer: assafakiva
ms.topic: concept-article #Don't change
ms.date: 11/05/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to review the hardware, software, networking, and configuration requirements for deploying Agents and Tools with Foundry Local, so that I can prepare my infrastructure for a successful deployment and operation.
---

# What you need for Agents and Tools with Foundry Local

This article discusses Azure, machine and storage, networking, and other requirements for Agents and Tools with Foundry Local.

## Resource requirements

To get started with Agents and Tools with Foundry Local, you need the following Azure and on-premises resources.

### Azure resources

Before deploying Agents and Tools with Foundry Local, ensure that you have the following Azure resources and permissions in place:

| **Resource** | **Description** |
| --- | --- |
| Azure subscription | An [Azure subscription](https://azure.microsoft.com/pricing/details/search/). |
| Microsoft Entra ID  permissions | - Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).<br>- Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application. <br> <br> As part of the prerequisites tasks, you [configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md). |
| Permissions for AKS enabled by Azure Arc | Permissions to deploy [AKS Arc Kubernetes clusters](/azure/aks/hybrid/aks-create-clusters-portal), create [node pools](/azure/aks/hybrid/manage-node-pools), and install [extensions](/azure/azure-arc/kubernetes/extensions-release).   As part of the prerequisites tasks, see [Verify contributor role for Agents and Tools with Foundry Local](prepare-contributor-permission.md) |
| Transport Layer Security (TLS) termination certificate | A certificate signed by a company-specific certification authority (CA) or a well-known public CA for secure deployments. If you don't provide one, Agents and Tools with Foundry Local generates a self-signed certificate. Don't use a self-signed certificate for production environments. |
| Language model endpoint | All agents (whether `indexedSourcesEnabled` is `true` or `false`) use Agent Service model connections (`/connections`) that you configure after deployment. Deployment-time `byom.*` settings are required only when `indexedSourcesEnabled=true` and are used only by deprecated developer-portal classic chat. Use an OpenAI-compatible chat completions endpoint such as GPT-OSS-20B through [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local), or a supported cloud-hosted endpoint. Set `foundryClientId` during deployment whenever indexed sources or agents use Foundry Local managed identity on this cluster. |

### On-premises resources

Agents and Tools with Foundry Local deployment supports the following on-premises resources in your environment:

| **Resource** | **Description** |
| --- | --- |
| Azure Local infrastructure* | An instance of [Azure Local](/azure/azure-local/overview) infrastructure, minimum version 2504. |
| AKS Arc cluster on Azure Local* | An [AKS Arc cluster](/azure/aks/hybrid/aks-create-clusters-portal) running on the Azure Local instance. Use [GPUs](/azure/aks/hybrid/deploy-gpu-node-pool) for better performance. When `indexedSourcesEnabled=true` with GPU mode enabled, include at least two [GPU-enabled VMs](/azure/azure-local/manage/gpu-preparation) in the node pool (one for text embedding and one for image processing). Embedding GPU VMs aren't required (`0` embedding GPU VMs) when `indexedSourcesEnabled=false` or in CPU mode. Docling (document parser) runs on CPU. The language model runs in Foundry Local on the cluster or at an external endpoint. As part of the prerequisite tasks, you [prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md). |
| Routable, static IP address | One routable, static IP address for the [MetalLB](/azure/aks/hybrid/deploy-load-balancer-portal) load balancer. If MetalLB is already configured with a routable IP, you can skip this requirement. The IP must be accessible from client machines. <br><br>As part of the prerequisite tasks, setting up MetalLB is included in the following articles:<br><br>- [Install networking and observability components for Agents and Tools with Foundry Local](prepare-networking-observability.md) <br>- [Configure DNS for Agents and Tools with Foundry Local](prepare-dns.md). |
| Network File System (NFS) | Optional. Use NFS v3.0 or v4.1 when you want to ingest documents or images from a network file share. AUTH_SYS requires the share path, user ID, and group ID. Kerberos (`krb5p`) uses an NFS v4.1 service principal. See setup guides for [Windows Server](/windows-server/storage/nfs/deploy-nfs) and [Linux](https://linuxconfig.org/how-to-configure-nfs-on-linux). For Kerberos setup, see [NFS with Kerberos authentication](connect-file-share-kerberos-overview.md). |
| Windows machine (optional) | Ease the management of the Azure Arc-enabled Kubernetes cluster on Azure Local by configuring a driver machine or local management host.<br><br>As part of the prerequisite tasks, install tools like Azure CLI, kubectl, and Helm to prepare the driver machine. For more information, see: <br><br>- [Prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md)<br>- [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md). |

\* Agents and Tools with Foundry Local is validated on Azure Local.

## Minimum VM hardware requirements

The following table lists the minimum hardware requirements for the virtual machines when `indexedSourcesEnabled=true` with GPU mode enabled (`0` embedding GPU VMs are required when `indexedSourcesEnabled=false` or in CPU mode).

| **Mode** | **VM specs and suggested minimum sizes** |
| ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **GPU (`indexedSourcesEnabled=true`)** | 2 x GPU-enabled VMs for GPU-accelerated embeddings (one for text embedding, one for image processing; `0` embedding GPU VMs when `indexedSourcesEnabled=false` or in CPU mode). Recommended sizes: Standard_NC8_A2 or Standard_NC8_A16. 3 x CPU VMs - Minimum spec: 8 vCPUs, 32 GB - Recommended size: Standard_D8s_v3. Docling (document parser) runs on CPU - no dedicated GPU required. |

### Minimum cluster node capacity

The core Agent Service always deploys. When you set `indexedSourcesEnabled=true`, the extension also deploys local ingestion and retrieval workloads. The following table shows the existing minimum worker node guidance:

| `indexedSourcesEnabled` | CPU workers | GPU workers | Total vCPU | Total RAM |
| --- | --- | --- | --- | --- |
| `false` | 3x Standard_D8s_v3 | 0 | 24 | 96 GB |
| `true` (default) | 3x Standard_D8s_v3 | 2 (GPU mode; 0 in CPU mode) | 24+ | 96+ GB |

> [!NOTE]
> On Azure Local (HaaS), scale node pools with `az aksarc nodepool scale --name <pool> --node-count <N>`.

For more information, see [Resource limits, VM sizes, and regions for AKS on Windows Server](/azure/aks/hybrid/concepts-support).

### Language model requirement

All agents (whether `indexedSourcesEnabled` is `true` or `false`) use Agent Service model connections (`/connections`) that you configure after deployment. When `indexedSourcesEnabled=true`, extension deployment also requires `byom.*` endpoint settings for deprecated developer-portal classic chat. Set `foundryClientId` during deployment whenever indexed sources or agents use Foundry Local managed identity on the cluster. The language model runs in Foundry Local on the cluster or at an external endpoint.

When `indexedSourcesEnabled=true` with GPU mode enabled, the two GPUs in the cluster run the embedding models. Docling runs on CPU:

| **Resource** | **Purpose** |
| --- | --- |
| GPU 1 | Text embedding model (BGE-M3) |
| GPU 2 | Image embedding model (CLIP ViT-L/14) |
| CPU | Document parser (Docling) - no GPU required |

**Recommended model:** GPT-OSS-20B via [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local). Also supported: [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) for cloud-hosted models. See [Create an endpoint](prepare-model-endpoint.md).

For the best experience, deploy both the Foundry Local extension and the Agents and Tools with Foundry Local extension on the same Arc-enabled Kubernetes cluster. Foundry Local on Azure Local provides the recommended language model endpoint, while Agents and Tools with Foundry Local provides the agentic RAG platform. Install the Foundry Local extension first, then use its model endpoint URL when you deploy Agents and Tools with Foundry Local. For more information, see [What is Foundry Local on Azure Local?](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local)

#### Hardware requirements (GPT-OSS-20B via Foundry Local)

The language model endpoint runs separately from Agents and Tools with Foundry Local. If you use GPT-OSS-20B with [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local), the model host requires its own GPU:

| **Resource** | **Minimum** | **Recommended (production)** |
| --- | --- | --- |
| **GPU** | 1 × NVIDIA GPU, ≥ 24 GB VRAM | 1 × NVIDIA GPU, ≥  48 GB VRAM |
| **CPU** | 8+ vCPUs | 16+ vCPUs |
| **RAM** | 32 GB | 64 GB |
| **Storage** | ≥ 50 GB | ≥ 50–100 GB per replica |

The minimum configuration is suitable for development and low-concurrency scenarios. For production workloads, larger context windows, or higher concurrency, use the recommended configuration.

Foundry Local validates GPU compatibility at deployment time and fails with a clear error if resources are insufficient. Test in a non-production environment first.

### Requirements by indexed sources setting

| **Requirement** | **`indexedSourcesEnabled=true`** | **`indexedSourcesEnabled=false`** |
| --- | --- | --- |
| GPU-enabled VMs (embedding) | 2 when GPU mode is enabled (`0` in CPU mode) | 0 |
| CPU VMs | 3+ | 3+ |
| Language model compute | Provided by Foundry Local on the cluster or an external endpoint (for Agent Service `/connections` and deployment-time `byom.*`) | Provided by Foundry Local on the cluster or an external endpoint (for Agent Service `/connections`) |
| NFS data source | Optional; required only when you ingest from NFS | Not required |
| Language model endpoint | All agents use `/connections` configured after deployment; deployment-time `byom.*` settings are required only for deprecated developer-portal classic chat (set `foundryClientId` when indexed sources or agents use Foundry Local managed identity on the cluster) | All agents use `/connections` configured after deployment; deployment-time `byom.*` settings aren't required (set `foundryClientId` when agents use Foundry Local managed identity on the cluster) |
| Entra ID app registration | Required | Required |

## Minimum software requirements

The following table lists the supported minimum software requirements for Agents and Tools with Foundry Local.

| **Component** | **Minimum requirements** |
| --- | --- |
| **VM Operating System** | Linux |
| **Azure Local version*** | Azure Local [2504](/azure/azure-local/whats-new) release |
| **Azure CLI** | As shipped with Azure Local. Don't update to the latest version of Azure CLI, and use the one that was originally shipped with Azure Local. |

\* Agents and Tools with Foundry Local is validated on Azure Local.

## Network requirements

See the current [Azure Local](/azure/azure-local/concepts/firewall-requirements) and [AKS on Azure Local](/azure/aks-hybrid-edge/local/hyperconverged/network-system-requirements) requirements.

Authenticated external MCP endpoints must use HTTPS. Register only MCP servers that you operate or trust because the server receives the delegated credential and its tool output is processed as untrusted input. The extension blocks credential forwarding to destinations that aren't trusted.

## Supported document formats and size

The extension supports the following capabilities and related file formats:

| **Capability** | **Supported file format** |
| --- | --- |
| Text and structured-data extraction | PDF, DOCX, PPTX, TXT, HTML, MD, XLSX, CSV |
| Image ingestion | JPG, JPEG, PNG |

Each individual file can be up to 30 MB. The ingestion service skips files that exceed this limit.

The extension doesn't currently support document or image file types that aren't listed, like audio and video files.

## Supported data sources

When `indexedSourcesEnabled=true`, the extension supports the following data sources:

- **NFS** v3.0 and v4.1 with AUTH_SYS authentication (all deployments).
- **NFS** v4.1 with Kerberos (`krb5p`) authentication when Kerberos is enabled and the worker nodes are configured for it. See [NFS with Kerberos authentication](connect-file-share-kerberos-overview.md).
- **SharePoint Server** Subscription Edition with High-Trust Server-to-Server (S2S) authentication in connected and disconnected environments. See [SharePoint Server-to-Server authentication](connect-sharepoint-overview.md).

## Supported regions

If you plan to use the [quickstart](quickstart-edge-rag.md) for evaluation or development purposes, deploy Azure resources for Agents and Tools with Foundry Local in any region supported by Azure Arc enabled Kubernetes. For production deployments, deploy Agents and Tools with Foundry Local and required resources in any region supported by Azure Local.

For the most up-to-date list of supported regions by service, see the [Azure products by region table](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table).

## Related content

- [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md)
- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
