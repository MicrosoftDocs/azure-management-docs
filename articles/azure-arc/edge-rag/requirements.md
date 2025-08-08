---

title: Requirements for Edge RAG Preview, Enabled by Azure Arc
description: "Learn how to deploy Edge RAG with this guide on hardware, software, networking, and configuration requirements for success."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 08/08/2025
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to review the hardware, software, networking, and configuration requirements for deploying Edge RAG, so that I can prepare my infrastructure for a successful deployment and operation.
---

# What you need for Edge RAG Preview, enabled by Azure Arc

This article discusses Azure, machine and storage, networking, and other requirements for Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Azure and on-premises requirements

To get started with Edge RAG, you need the following Azure and on-premises resources.

### Azure resources

Before deploying Edge RAG, make sure you have the following Azure resources and permissions in place:

| **Resource** | **Description** |
|---|---|
| Azure subscription | An [Azure subscription](https://azure.microsoft.com/pricing/details/search/). |
| Permissions for Azure Kubernetes Service (AKS) enabled by Azure Arc| Permissions to deploy [AKS Arc Kubernetes clusters](/azure/aks/hybrid/aks-create-clusters-portal), create [node pools](/azure/aks/hybrid/manage-node-pools), and install [extensions](/azure/azure-arc/kubernetes/extensions-release).   As part of the prerequisites tasks, see [Verify contributor role for Edge RAG Preview enabled by Azure Arc](prepare-contributor-permission.md).|
| Microsoft Entra ID  permissions |- Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).<br>- Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application. <br> <br> As part of the prerequisites tasks, you [configure authentication for Edge RAG Preview enabled by Azure Arc](prepare-authentication.md).|
| Transport Layer Security (TLS) termination certificate | A certificate signed by a company-specific certification authority (CA) or a well-known public CA for secure deployments. If you don't provide one, Edge RAG generates a self-signed certificate. We don't recommend using a self-signed certificate for production environments. |

### On-premises resources

The following on-premises resources are required to deploy Edge RAG in your environment:

| **Resource** | **Description** |
|---|---|
| Azure Local infrastructure | An instance of [Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/introducing-azure-local-cloud-infrastructure-for-distributed-locations-enabled-b/4296017) infrastructure, minimum version [2411](/azure/azure-local/whats-new). |
| AKS Arc cluster on Azure Local | An [AKS Arc cluster](/azure/aks/hybrid/aks-create-clusters-portal) running on the Azure Local instance. Use [GPUs](/azure/aks/hybrid/deploy-gpu-node-pool) for better performance; include at least three [GPU-enabled VMs](/azure/azure-local/manage/gpu-preparation) in the node pool for image and text scenarios. As part of the prerequisites tasks, you [prepare AKS cluster on Azure Local for Edge RAG Preview enabled by Azure Arc](prepare-aks-cluster.md). |
| Routable, static IP address | One routable, static IP address for the [MetalLB](/azure/aks/hybrid/deploy-load-balancer-portal) load balancer. If MetalLB is already configured with a routable IP, this requirement can be skipped. The IP must be accessible from client machines. <br><br>As part of the prerequisites tasks, setting up MetalLB is included in the following articles:<br><br>- [Install networking and observability components for Edge RAG Preview enabled by Azure Arc](prepare-networking-observability.md) <br>- [Configure DNS for Edge RAG Preview enabled by Azure Arc](prepare-dns.md). |
| Network File System (NFS) | An NFS v3.0 or v4.1 containing your on-premises documents or images. See setup guides for [Windows Server](/windows-server/storage/nfs/deploy-nfs) and [Linux](https://linuxconfig.org/how-to-configure-nfs-on-linux). As part of the prerequisites tasks, see [Verify NFS server access for Edge RAG Preview enabled by Azure Arc](prepare-file-server.md).|

## Minimum hardware requirements

The following table lists the minimum hardware requirements for the virtual machines. 

| **Mode** | **VM specs & suggested minimum sizes** |
|---|---|
| **GPU** | 3 x GPU-enabled VMs </br>Recommended sizes (choose one based on GPU):</br>- Standard_NC8_A2<br>- Standard_NC8_A16<br>3 x CPU VMs<br> - Minimum spec: 8 vCPUs, 32 GB<br>- Recommended size: Standard_D8s_v3|

For more information, see [Resource limits, VM sizes, and regions for AKS on Windows Server](/azure/aks/hybrid/concepts-support).

The following table lists the hardware recommendations for each language model available with Edge RAG.

| **Model Name**                        | **GPU Support** | **Minimum VM SKUs**      |
|----------------------------------------|-----------------|--------------------------|
| [Microsoft/Phi-3.5-mini-instruct](https://huggingface.co/microsoft/Phi-3.5-mini-instruct)| Nvidia A2<br>Nvidia A16       | Standard_NC8_A2 <br>Standard_NC8_A16         |
| [mistralai/Mistral-7B-Instruct-v0.2](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)     | Nvidia A2<br>Nvidia A16      | Standard_NC8_A2 <br>Standard_NC8_A16         |


If you plan to use a CPU-only setup, review the files size and chunking limitations. See:
- [Supported document formats and size](#supported-document-formats-and-size)
- [Chunk settings](build-chat-solution-overview.md#chunk-settings)

## Minimum software requirements

The following table lists the minimum software requirements for Edge RAG.

| **Component** | **Minimum requirements** |
|---|---|
| **VM Operating System** |Linux|
| **Azure Local version** | Azure Local [2411](/azure/azure-local/whats-new) release |
| **Azure CLI** | As shipped with Azure Local. Don't update to the latest version of Azure CLI, and use the one that was originally shipped with Azure Local. |

## Network requirements

All current [Azure Local](/azure/azure-local/concepts/firewall-requirements) and [AKS on Azure Local](/azure/aks/hybrid/aks-hci-network-system-requirements) requirements.

## Supported document formats and size

Edge RAG supports the following capabilities and related file formats:

| **Capability** | **Supported file format** |
|---|---|
| Text extraction | PDF, DOCX, TXT, MHTML, MHT, MD |
| Image ingestion | JPG, JPEG, PNG |

With a GPU setup, each individual file can be up to 30 MB. If you're using a CPU-only setup, each individual file can be up to 5 MB.

Document or image file types not listed, like audio and video files, aren't currently supported.

## Supported data sources

Edge RAG supports Network File System (NFS) v3.0  and v4.1 with AUTH_SYS authentication as a data source. Kerberos isn't supported.

## Supported regions

Edge RAG is supported in the following regions:

- westeurope
- eastus2euap
- eastus
- westus2
- australiaeast
- eastus2
- japaneast
- canadacentral
- uksouth
- centralindia
- koreacentral

## Related content

[Complete Edge RAG deployment prerequisites](complete-prerequisites.md)