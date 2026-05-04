---

title: Requirements for Agentic RAG Preview, Enabled by Azure Arc
description: "Learn how to deploy Agentic RAG with this guide on hardware, software, networking, and configuration requirements for success."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 04/30/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to review the hardware, software, networking, and configuration requirements for deploying Agentic RAG, so that I can prepare my infrastructure for a successful deployment and operation.
---

# What you need for Agentic RAG Preview, enabled by Azure Arc

This article discusses Azure, machine and storage, networking, and other requirements for Agentic RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Resource requirements

To get started with Agentic RAG, you need the following Azure and on-premises resources.

### Azure resources

Before deploying Agentic RAG, make sure you have the following Azure resources and permissions in place:

| **Resource** | **Description** |
|---|---|
| Azure subscription | An [Azure subscription](https://azure.microsoft.com/pricing/details/search/). |
| Microsoft Entra ID  permissions |- Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).<br>- Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application. <br> <br> As part of the prerequisites tasks, you [configure authentication for Agentic RAG Preview enabled by Azure Arc](prepare-authentication.md).|
| Permissions for AKS enabled by Azure Arc| Permissions to deploy [AKS Arc Kubernetes clusters](/azure/aks/hybrid/aks-create-clusters-portal), create [node pools](/azure/aks/hybrid/manage-node-pools), and install [extensions](/azure/azure-arc/kubernetes/extensions-release).   As part of the prerequisites tasks, see [Verify contributor role for Agentic RAG Preview enabled by Azure Arc](prepare-contributor-permission.md).|
| Transport Layer Security (TLS) termination certificate | A certificate signed by a company-specific certification authority (CA) or a well-known public CA for secure deployments. If you don't provide one, Agentic RAG generates a self-signed certificate. We don't recommend using a self-signed certificate for production environments. |
| BYOM language model endpoint | An OpenAI-compatible chat completions endpoint for your language model. The recommended model is **GPT-OSS-20B** via Foundry Local on Arc (requires its own GPU — see BYOM hardware requirements above). Also supported: KAITO, Azure OpenAI, Ollama, or another provider. See [Create a BYOM endpoint](prepare-model-endpoint). |

### On-premises resources

An Agentic RAG deployment is supported where you have the following on-premises resources in your environment:

| **Resource** | **Description** |
|---|---|
| Azure Local infrastructure* | An instance of [Azure Local](/azure/azure-local/overview) infrastructure, minimum version 2504. |
| AKS Arc cluster on Azure Local* | An [AKS Arc cluster](/azure/aks/hybrid/aks-create-clusters-portal) running on the Azure Local instance. Use [GPUs](/azure/aks/hybrid/deploy-gpu-node-pool) for better performance; include at least two [GPU-enabled VMs](/azure/azure-local/manage/gpu-preparation) in the node pool (one for text embedding, one for image processing). Docling (document parser) runs on CPU. The LLM runs externally via your BYOM endpoint. As part of the prerequisites tasks, you [prepare AKS cluster on Azure Local for Agentic RAG Preview enabled by Azure Arc](prepare-aks-cluster.md). |
| Routable, static IP address | One routable, static IP address for the [MetalLB](/azure/aks/hybrid/deploy-load-balancer-portal) load balancer. If MetalLB is already configured with a routable IP, this requirement can be skipped. The IP must be accessible from client machines. <br><br>As part of the prerequisites tasks, setting up MetalLB is included in the following articles:<br><br>- [Install networking and observability components for Agentic RAG Preview enabled by Azure Arc](prepare-networking-observability.md) <br>- [Configure DNS for Agentic RAG Preview enabled by Azure Arc](prepare-dns.md). |
| Network File System (NFS) | An NFS v3.0 or v4.1 containing your on-premises documents or images. Only AUTH_SYS authentication method is supported. Kerberos authentication isn't supported. Requires share path, NFS user ID, and group ID. **Required for combined and knowledge modes only.** Not required for agentic mode. See setup guides for [Windows Server](/windows-server/storage/nfs/deploy-nfs) and [Linux](https://linuxconfig.org/how-to-configure-nfs-on-linux).|
|Windows machine (optional)| Ease the management of the Azure Arc-enabled Kubernetes cluster on Azure Local by configuring a driver machine or local management host.<br><br>As part of the prerequisites tasks, install tools like Azure CLI, kubectl, and Helm to prepare the driver machine. For more information, see: <br><br>- [Prepare AKS cluster on Azure Local for Agentic RAG Preview enabled by Azure Arc](prepare-aks-cluster.md)<br>- [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).|

\* Agentic RAG is validated on Azure Local.

## Minimum VM hardware requirements

The following table lists the minimum hardware requirements for the virtual machines. 

| **Mode** | **VM specs & suggested minimum sizes** |
|---|---|
| **GPU** | 2 x GPU-enabled VMs (one for text embedding, one for image processing). Recommended sizes: Standard_NC8_A2 or Standard_NC8_A16. 3 x CPU VMs — Minimum spec: 8 vCPUs, 32 GB — Recommended size: Standard_D8s_v3. Docling (document parser) runs on CPU — no dedicated GPU required.|

For more information, see [Resource limits, VM sizes, and regions for AKS on Windows Server](/azure/aks/hybrid/concepts-support).

### BYOM language model requirement

Agentic RAG does not bundle any language models. You must provide an external LLM endpoint (Bring Your Own Model) that exposes an OpenAI-compatible chat completions API. The LLM runs outside the Agentic RAG deployment — no GPU is consumed by the LLM within the cluster.

The 2 GPUs in the cluster are used for embedding models. Docling runs on CPU:

| Resource | Purpose |
|---|---|
| GPU 1 | Text embedding model (BGE-M3) |
| GPU 2 | Image embedding model (CLIP ViT-L/14) |
| CPU | Document parser (Docling) — no GPU required |

**Recommended BYOM model:** GPT-OSS-20B via Foundry Local on Arc. Also supported: KAITO, Azure OpenAI, Ollama, or any OpenAI-compatible endpoint. See [Create a BYOM endpoint](prepare-model-endpoint).

#### BYOM hardware requirements (GPT-OSS-20B via Foundry Local)

The BYOM endpoint runs separately from Agentic RAG. If you use GPT-OSS-20B with Foundry Local on Arc, the model host requires its own GPU:

| | Minimum | Recommended (production) |
|---|---|---|
| **GPU** | 1 × NVIDIA GPU, ≥ 24 GB VRAM | 1 × NVIDIA GPU, ≥ 48 GB VRAM |
| **CPU** | 8+ vCPUs | 16+ vCPUs |
| **RAM** | 32 GB | 64 GB |
| **Storage** | ≥ 50 GB | ≥ 50–100 GB per replica |

The minimum configuration is suitable for development and low-concurrency scenarios. For production workloads, larger context windows, or higher concurrency, use the recommended configuration. For very large context sizes (deep search, long-context reasoning), 80 GB–class GPUs may be required.

Foundry Local validates GPU compatibility at deployment time and fails with a clear error if resources are insufficient. Test in a non-production environment first.

### Requirements by deployment mode

| Requirement | combined | agentic | knowledge |
|---|---|---|---|
| GPU-enabled VMs (embedding) | 2 | 0 | 2 |
| CPU VMs | 3+ | 3+ | 3+ |
| BYOM GPU (e.g., GPT-OSS-20B) | 1+ (separate from cluster) | 1+ (separate from cluster) | 1+ (separate from cluster) |
| NFS data source | Required | Not required | Required |
| BYOM endpoint | Required | Required | Required |
| Entra ID app registration | Required | Required | Required |

If you plan to use a CPU-only setup, review the files size and chunking limitations. See:
- [Supported document formats and size](#supported-document-formats-and-size)
- [Chunk settings](build-chat-solution-overview.md#chunk-settings)

## Minimum software requirements

The following table lists the supported minimum software requirements for Agentic RAG.

| **Component** | **Minimum requirements** |
|---|---|
| **VM Operating System** |Linux|
| **Azure Local version*** | Azure Local [2504](/azure/azure-local/whats-new) release |
| **Azure CLI** | As shipped with Azure Local. Don't update to the latest version of Azure CLI, and use the one that was originally shipped with Azure Local. |

\* Agentic RAG is validated on Azure Local.

## Network requirements

All current [Azure Local](/azure/azure-local/concepts/firewall-requirements) and [AKS on Azure Local](/azure/aks/hybrid/aks-hci-network-system-requirements) requirements.

## Supported document formats and size

Agentic RAG supports the following capabilities and related file formats:

| **Capability** | **Supported file format** |
|---|---|
| Text extraction | PDF, DOCX, TXT, MHTML, MHT, MD |
| Image ingestion | JPG, JPEG, PNG |

With a GPU setup, each individual file can be up to 30 MB. If you're using a CPU-only setup, each individual file can be up to 5 MB.

Document or image file types not listed, like audio and video files, aren't currently supported.

## Supported data sources

Agentic RAG supports Network File System (NFS) v3.0  and v4.1 with AUTH_SYS authentication as a data source. Kerberos authentication isn't supported.

## Supported regions

If you plan to use the [quickstart](quickstart-edge-rag.md) for evaluation or  development purposes, deploy Azure resources for Agentic RAG in any region supported by Azure Arc enabled Kubernetes. For production deployments, deploy Agentic RAG and required resources in any region supported by Azure Local.

For the most up-to-date list of supported regions by service, see the [Azure products by region table](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table).

## Related content

- [Quickstart: Install Agentic RAG Preview enabled by Azure Arc](quickstart-edge-rag.md)
- [Complete Agentic RAG deployment prerequisites](complete-prerequisites.md)
- [Deployment overview for Agentic RAG](deploy-overview.md)