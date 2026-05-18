---

title: Requirements for Agents and Tools with Foundry Local
description: "Learn how to deploy Agents and Tools with Foundry Local with this guide on hardware, software, networking, and configuration requirements for success."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/12/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to review the hardware, software, networking, and configuration requirements for deploying Agents and Tools with Foundry Local, so that I can prepare my infrastructure for a successful deployment and operation.
---

# What you need for Agents and Tools with Foundry Local

This article discusses Azure, machine and storage, networking, and other requirements for Agents and Tools with Foundry Local.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Resource requirements

To get started with Agents and Tools with Foundry Local, you need the following Azure and on-premises resources.

### Azure resources

Before deploying Agents and Tools with Foundry Local, make sure you have the following Azure resources and permissions in place:

|**Resource** | **Description** |
|---|---|
| Azure subscription | An [Azure subscription](https://azure.microsoft.com/pricing/details/search/). |
| Microsoft Entra ID  permissions |- Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).<br>- Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application. <br> <br> As part of the prerequisites tasks, you [configure authentication for Agents and Tools with Foundry Local](prepare-authentication.md).|
| Permissions for AKS enabled by Azure Arc| Permissions to deploy [AKS Arc Kubernetes clusters](/azure/aks/hybrid/aks-create-clusters-portal), create [node pools](/azure/aks/hybrid/manage-node-pools), and install [extensions](/azure/azure-arc/kubernetes/extensions-release).   As part of the prerequisites tasks, see [Verify contributor role for Agents and Tools with Foundry Local](prepare-contributor-permission.md)|
| Transport Layer Security (TLS) termination certificate | A certificate signed by a company-specific certification authority (CA) or a well-known public CA for secure deployments. If you don't provide one, Agents and Tools with Foundry Local generates a self-signed certificate. Don't use a self-signed certificate for production environments. |
| BYOM language model endpoint | An OpenAI-compatible chat completions endpoint for your language model. The recommended model is **GPT-OSS-20B** via [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local) which requires its own GPU. See BYOM hardware requirements. Also supported: [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) for cloud-hosted models. See [Create a BYOM endpoint](prepare-model-endpoint.md). |

### On-premises resources

Agents and Tools with Foundry Local deployment supports the following on-premises resources in your environment:

| **Resource** | **Description** |
|---|---|
| Azure Local infrastructure* | An instance of [Azure Local](/azure/azure-local/overview) infrastructure, minimum version 2504. |
| AKS Arc cluster on Azure Local* | An [AKS Arc cluster](/azure/aks/hybrid/aks-create-clusters-portal) running on the Azure Local instance. Use [GPUs](/azure/aks/hybrid/deploy-gpu-node-pool) for better performance. Include at least two [GPU-enabled VMs](/azure/azure-local/manage/gpu-preparation) in the node pool - one for text embedding and one for image processing. Docling (document parser) runs on CPU. The LLM runs externally via your BYOM endpoint. As part of the prerequisite tasks, you [prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md). |
| Routable, static IP address | One routable, static IP address for the [MetalLB](/azure/aks/hybrid/deploy-load-balancer-portal) load balancer. If MetalLB is already configured with a routable IP, you can skip this requirement. The IP must be accessible from client machines. <br><br>As part of the prerequisite tasks, setting up MetalLB is included in the following articles:<br><br>- [Install networking and observability components for Agents and Tools with Foundry Local](prepare-networking-observability.md) <br>- [Configure DNS for Agents and Tools with Foundry Local](prepare-dns.md). |
| Network File System (NFS) | An NFS v3.0 or v4.1 containing your on-premises documents or images. AUTH_SYS authentication is supported for all deployments. For disconnected on-premises deployments, Kerberos (`krb5p`) authentication and SharePoint Server (S2S) are also supported as data sources. Requires share path, NFS user ID, and group ID (for AUTH_SYS) or Kerberos service principal (for `krb5p`). **Required for combined and knowledge modes only.** Not required for agentic mode. See setup guides for [Windows Server](/windows-server/storage/nfs/deploy-nfs) and [Linux](https://linuxconfig.org/how-to-configure-nfs-on-linux). For Kerberos setup, see [NFS with Kerberos authentication](connect-nfs-kerberos-overview.md). For SharePoint, see [SharePoint Server S2S authentication](connect-sharepoint-overview.md).|
|Windows machine (optional)| Ease the management of the Azure Arc-enabled Kubernetes cluster on Azure Local by configuring a driver machine or local management host.<br><br>As part of the prerequisite tasks, install tools like Azure CLI, kubectl, and Helm to prepare the driver machine. For more information, see: <br><br>- [Prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md)<br>- [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).|

\* Agents and Tools with Foundry Local is validated on Azure Local.

## Minimum VM hardware requirements

The following table lists the minimum hardware requirements for the virtual machines. 

| **Mode** | **VM specs and suggested minimum sizes** |
|---|---|
| **GPU** | 2 x GPU-enabled VMs (one for text embedding, one for image processing). Recommended sizes: Standard_NC8_A2 or Standard_NC8_A16. 3 x CPU VMs - Minimum spec: 8 vCPUs, 32 GB - Recommended size: Standard_D8s_v3. Docling (document parser) runs on CPU - no dedicated GPU required.|

For more information, see [Resource limits, VM sizes, and regions for AKS on Windows Server](/azure/aks/hybrid/concepts-support).

### BYOM language model requirement

Agents and Tools with Foundry Local doesn't bundle language models. You must provide an external LLM endpoint (Bring Your Own Model) that exposes an OpenAI-compatible chat completions API. The LLM runs outside the Agents and Tools with Foundry Local deployment - the LLM within the cluster consumes no GPU.

The two GPUs in the cluster are used for embedding models. Docling runs on CPU:

|**Resource** | **Purpose** |
|---|---|
| GPU 1 | Text embedding model (BGE-M3) |
| GPU 2 | Image embedding model (CLIP ViT-L/14) |
| CPU | Document parser (Docling) - no GPU required |

**Recommended BYOM model:** GPT-OSS-20B via [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local). Also supported: [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) for cloud-hosted models. See [Create a BYOM endpoint](prepare-model-endpoint.md).

For the best experience, deploy both the Foundry Local extension and the Agents and Tools with Foundry Local extension on the same Arc-enabled Kubernetes cluster. Foundry Local on Azure Local provides the recommended language model endpoint, while Agents and Tools with Foundry Local provides the agentic RAG platform. Install the Foundry Local extension first, then use its model endpoint URL when you deploy Agents and Tools with Foundry Local. For more information, see [What is Foundry Local on Azure Local?](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local).

#### BYOM hardware requirements (GPT-OSS-20B via Foundry Local)

The BYOM endpoint runs separately from Agents and Tools with Foundry Local. If you use GPT-OSS-20B with [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local), the model host requires its own GPU:

|**Resource**| **Minimum** | **Recommended (production)** |
|---|---|---|
| **GPU** | 1 × NVIDIA GPU, ≥ 24 GB VRAM | 1 × NVIDIA GPU, ≥  48 GB VRAM |
| **CPU** | 8+ vCPUs | 16+ vCPUs |
| **RAM** | 32 GB | 64 GB |
| **Storage** |≥ 50 GB | ≥ 50–100 GB per replica |

The minimum configuration is suitable for development and low-concurrency scenarios. For production workloads, larger context windows, or higher concurrency, use the recommended configuration.

Foundry Local validates GPU compatibility at deployment time and fails with a clear error if resources are insufficient. Test in a non-production environment first.

### Requirements by deployment mode

|**Requirement**| **Combined** | **Agentic** | **Knowledge** |
|---|---|---|---|
| GPU-enabled VMs (embedding) | 2 | 0 | 2 |
| CPU VMs | 3+ | 3+ | 3+ |
| BYOM GPU (e.g., GPT-OSS-20B) | 1+ (separate from cluster) | 1+ (separate from cluster) | 1+ (separate from cluster) |
| NFS data source | Required | Not required | Required |
| BYOM endpoint | Required | Required | Required |
| Entra ID app registration | Required | Required | Required |

If you plan to use a CPU-only setup, review the file size and chunking limitations. See:
- [Supported document formats and size](#supported-document-formats-and-size)
- [Chunk settings](build-chat-solution-overview.md#chunk-settings)

## Minimum software requirements

The following table lists the supported minimum software requirements for Agents and Tools with Foundry Local.

| **Component** | **Minimum requirements** |
|---|---|
| **VM Operating System** |Linux|
| **Azure Local version*** | Azure Local [2504](/azure/azure-local/whats-new) release |
| **Azure CLI** | As shipped with Azure Local. Don't update to the latest version of Azure CLI, and use the one that was originally shipped with Azure Local. |

\* Agents and Tools with Foundry Local is validated on Azure Local.

## Network requirements

All current [Azure Local](/azure/azure-local/concepts/firewall-requirements) and [AKS on Azure Local](/azure/aks/hybrid/aks-hci-network-system-requirements) requirements.

## Supported document formats and size

Agents and Tools with Foundry Local support the following capabilities and related file formats:

| **Capability** | **Supported file format** |
|---|---|
| Text extraction | PDF, DOCX, TXT, MHTML, MHT, MD |
| Image ingestion | JPG, JPEG, PNG |

With a GPU setup, each individual file can be up to 30 MB. If you're using a CPU-only setup, each individual file can be up to 5 MB.

Document or image file types not listed, like audio and video files, aren't currently supported.

## Supported data sources

Agents and Tools with Foundry Local supports the following data sources:

- **NFS** v3.0 and v4.1 with AUTH_SYS authentication (all deployments).
- **NFS** v4.1 with Kerberos (`krb5p`) authentication (disconnected on-premises deployments only). See [NFS with Kerberos authentication](connect-nfs-kerberos-overview.md).
- **SharePoint Server** Subscription Edition with High-Trust Server-to-Server (S2S) authentication (disconnected on-premises deployments only). See [SharePoint Server S2S authentication](connect-sharepoint-overview.md).

## Supported regions

If you plan to use the [quickstart](quickstart-edge-rag.md) for evaluation or  development purposes, deploy Azure resources for Agents and Tools with Foundry Local in any region supported by Azure Arc enabled Kubernetes. For production deployments, deploy Agents and Tools with Foundry Local and required resources in any region supported by Azure Local.

For the most up-to-date list of supported regions by service, see the [Azure products by region table](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table).

## Related content

- [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md)
- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
