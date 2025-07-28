---

title: Requirements for Edge RAG Preview, Enabled by Azure Arc
description: "Learn how to deploy Edge RAG with this guide on hardware, software, networking, and configuration requirements for success."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/13/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to review the hardware, software, networking, and configuration requirements for deploying Edge RAG, so that I can prepare my infrastructure for a successful deployment and operation.
---

# What you need for Edge RAG Preview, enabled by Azure Arc

This article discusses Azure, machine and storage, networking, and other requirements for Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Requirements

To get started with Edge RAG, you need:

- Azure resources:
  - An [Azure subscription](https://azure.microsoft.com/pricing/details/search/).
  - Permissions to deploy [AKS Arc kubernetes cluster](/azure/aks/hybrid/aks-create-clusters-portal), create [node pools](/azure/aks/hybrid/manage-node-pools), and install [extensions](/azure/azure-arc/kubernetes/extensions-release).
  - Permissions to create: 
    - Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).
    - Add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application.
  - For secure deployments:
    - Transport Layer Security (TLS) termination certificate: 
      - This certificate must be signed by a company-specific Certificate Authority (CA) or a well-known public CA.
    - In the absence of a TLS termination certificate, Edge RAG generates a self-signed certificate to ease deployments. We don't recommend continued usage of the self-signed certificate for security conscious deployments.
- On-premises resources
  - An instance of [Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/introducing-azure-local-cloud-infrastructure-for-distributed-locations-enabled-b/4296017) infrastructure
    - Minimum version: Azure Local release [2411](/azure/azure-local/whats-new).
  - An [AKS Arc cluster](/azure/aks/hybrid/aks-create-clusters-portal) on the Azure Local instance.
    - Use [GPUs](/azure/aks/hybrid/deploy-gpu-node-pool) for better performance.
    - When using GPUs, include at least three [GPU-enabled VMs](/azure/azure-local/manage/gpu-preparation) in the node pool for image and text scenarios.
  - One routable, static IP address.
    - This IP address is used by [MetalLB](/azure/aks/hybrid/deploy-load-balancer-portal) load balancer. If you already have MetalLB configured on the target cluster with a routable IP address, you can skip this.
    - This IP address should be routable from client machines.
  - A Network File System (NFS) v3.0 or v4.1 containing your on-premises documents or images.
    - See how to set up NFS on [Windows Server](/windows-server/storage/nfs/deploy-nfs).
    - See how to set up NFS on [Linux](https://linuxconfig.org/how-to-configure-nfs-on-linux).

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

With a GPU setup each individual file can be up to 30 MB. If you're using a CPU-only setup, each individual file can be up to 5 MB.

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