---
title: Evaluate Edge RAG Without Azure Local
description: Learn how to set up and evaluate Edge RAG using recommended Azure resources—no Azure Local device required.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 09/05/2025
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As a Microsoft customer, I want to evaluate Edge RAG using recommended Azure resources, without needing an Azure Local device.
---
# Evaluate Edge RAG Preview without Azure Local

Evaluate Edge RAG Preview, enabled by Azure Arc, and its generative AI chat capabilities by using only Azure resources. No on-premises infrastructure is required. This article gives you the steps to set up a nonproduction evaluation environment for Edge RAG.

This evaluation setup is for testing and learning purposes only. For production deployments, see [Edge RAG Deployment Overview](deploy-overview.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you have:

- An active [Azure subscription](https://azure.microsoft.com/free/).
- Owner or Contributor permissions in the subscription.
- Permissions to create and manage:
  - Azure Kubernetes Service (AKS) clusters and install extensions.
  - Microsoft Entra ID applications, users, and groups.

## Set up your evaluation environment

Complete the following steps to prepare your Azure resources and configure the environment to evaluate Edge RAG.

| Step | Task | Description |
|------|------|-------------|
| 1 | Create a high-performance AKS cluster on Azure Arc | In the Azure portal, create an AKS cluster in a supported region. For the node pool, configure 3 GPU-enabled virtual machines (VMs) that are Standard_NC8_A2 or Standard_NC8_A16. See [How to deploy a Kubernetes cluster using the Azure portal](/azure/aks/hybrid/aks-create-clusters-portal). |
| 2 | Set up a secure data source | Create an Azure Files share in the same region as your AKS cluster. Upload sample documents and images in supported formats (PDF, DOCX, TXT, MHTML, MD, JPG, PNG). Set up private endpoints for storage to restrict access and enhance security. See [Create an NFS Azure file share using the Azure portal](/azure/storage/files/storage-files-quick-create-use-linux). |
| 3 | Configure authentication with Microsoft Entra ID | Create an application registration and app roles in Microsoft Entra ID. Assign users or groups for the evaluation to the app roles. See [Configure authentication for Edge RAG Preview enabled by Azure Arc](/azure/azure-arc/edge-rag/prepare-authentication). |
| 4 | Configure local machine or VM to reach Edge RAG | Configure your local machine or a VM to reach the Edge RAG developer portal. See [Configure DNS for the local portal](prepare-dns.md#configure-dns-for-the-local-portal). |
| 5 | Deploy the Edge RAG extension | Use the Azure portal or Azure CLI to install the Edge RAG extension on your AKS cluster. Select a Microsoft-hosted language model like Phi-3.5 or Mistral-7B. See [Deploy and manage cluster extensions by using Azure CLI](/azure/aks/deploy-extensions-az-cli) |
| 6 | Configure and test the chat solution | After you deploy the Edge RAG extension, open a browser with the domain name you provided at deployment and app registration to try out Edge RAG. Add and configure the data source for your Edge RAG chat solution using the Azure Files NFS and uploaded files. Configure the data queries and model settings to refine the chat experience. Evaluate the AI responses and user experience. See [Add data source for Edge RAG Preview, enabled by Azure Arc](add-data-source.md), [Set up the data query for chat solution](set-up-data-query.md), and [Test the chat solution](test-end-user-app.md). |


## Related content

For production scenarios, review [Edge RAG Deployment Overview](deploy-overview.md) and [Requirements for Edge RAG](requirements.md).