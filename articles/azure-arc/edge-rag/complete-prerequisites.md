---
title: Deployment Prerequisites Checklist for Edge RAG
description: "Learn how to complete deployment prerequisites for Edge RAG to ensure a successful setup for your chat solution."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 10/10/2025
ai-usage: ai-generated
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator or AI application developer, I want to complete the deployment prerequisites for Edge RAG so that I can ensure a successful setup and configuration of the environment for running AI-powered applications."
---

# Deployment prerequisites checklist for Edge RAG Preview enabled by Azure Arc

Complete the following steps to prepare for your Edge RAG deployment. 

To try Edge RAG without the need for local hardware, see [Quickstart: Install Edge RAG Preview enabled by Azure Arc](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]


## Prerequisites checklist

Use this checklist to prepare your environment before deploying Edge RAG on Azure Arc. Each step links to detailed instructions or supporting documentation.

| Step | Task | Description |
|------|------|-------------|
|1 |[Verify your environment meets requirements](requirements.md)|Before you begin, review [What you need for Edge RAG](requirements.md).|
| 2 | [Verify you have the Contributor role](prepare-contributor-permission.md) | Verify that you have the **Contributor** role at the subscription level. This role is required to register providers and enable features. |
| 3 | [Choose a language model](prepare-language-model.md) | Decide whether to use a Microsoft-hosted model or your own. This choice is final after you deploy the Edge RAG extension. |
| 4 | [(Optional) Create a model endpoint](prepare-model-endpoint.md) | If using your own model, set up an OpenAI API compatible endpoint to use with Edge RAG. |
| 5 | [Verify network file system (NFS) Server](prepare-file-server.md) | Make sure your documents and images are hosted on a supported and reachable NFS server. |
| 6 | [Prepare AKS cluster on Azure Local](prepare-aks-cluster.md) | Create an AKS Arc cluster on your Azure Local instance with a node pool that meets minimum requirements. |
| 7 | [Configure authentication](prepare-authentication.md) | Configure access to the Edge RAG for AI application developers and for end users of the chat endpoint. |
| 8 | [Install networking and observability components](prepare-networking-observability.md) | Deploy required components for network routing, monitoring, and logging. |
| 9 | [Configure DNS](prepare-dns.md) | Make sure DNS is properly configured so that you can reach the local portal domain for Edge RAG. |

## Related content

- [Deployment overview for Edge RAG](deploy-overview.md)
- [Deploy the Edge RAG extension](deploy.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc)