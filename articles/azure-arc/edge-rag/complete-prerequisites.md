---
title: Deployment Prerequisites Checklist for Agentic RAG
description: "Learn how to complete deployment prerequisites for Agentic RAG to ensure a successful setup for your chat solution."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 04/27/2026
ai-usage: ai-generated
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator or AI application developer, I want to complete the deployment prerequisites for Agentic RAG so that I can ensure a successful setup and configuration of the environment for running AI-powered applications."
---

# Deployment prerequisites checklist for Agentic RAG enabled by Azure Arc

Complete the following steps to prepare for your Agentic RAG deployment. 

To try Agentic RAG without the need for local hardware, see [Quickstart: Install Agentic RAG enabled by Azure Arc](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]


## Prerequisites checklist

Use this checklist to prepare your environment before deploying Agentic RAG on Azure Arc. Each step links to detailed instructions or supporting documentation.

| Step | Task | Description |
|------|------|-------------|
| 1 | [Verify your environment meets requirements](requirements.md) | Before you begin, review [What you need for Agentic RAG](requirements.md). |
| 2 | Choose your deployment mode | Decide which deployment mode you need: **combined** (default, full platform), **agentic** (agents only, no local data ingestion), or **knowledge** (data ingestion and RAG only, no agents). Your choice affects which prerequisites apply. For example, agentic mode doesn't require GPUs or a network files system (NFS) data source. |
| 3 | [Verify you have the Contributor role](prepare-contributor-permission.md) | Verify that you have the **Contributor** role at the subscription level. This role is required to register providers and enable features. |
| 4 | [Set up your language model endpoint (BYOM)](prepare-language-model.md) | Agentic RAG requires you to Bring Your Own Model (BYOM). Set up an OpenAI-compatible LLM endpoint before deployment. See [Prepare your language model endpoint](prepare-language-model.md). |
| 5 | [Create a BYOM endpoint](prepare-model-endpoint.md) | Set up an OpenAI API-compatible endpoint for your language model. This step is **mandatory** for all deployments. See [Create a BYOM endpoint](prepare-model-endpoint.md). |
| 6 | [Verify file share access](prepare-file-server.md) | Make sure your documents and images are hosted on a supported and reachable NFS server. **Required for combined and knowledge modes only.** Skip this step if deploying in agentic mode. |
| 7 | [Prepare an Azure Kubernetes (AKS) cluster on Azure Local](prepare-aks-cluster.md) | Create an AKS Arc cluster on your Azure Local instance with a node pool that meets minimum requirements. |
| 8 | [Configure authentication](prepare-authentication.md) | Configure access to Agentic RAG for AI application developers and for end users of the chat endpoint. |
| 9 | [Install networking and observability components](prepare-networking-observability.md) | Deploy required components for network routing, monitoring, and logging. |
| 10 | [Configure DNS](prepare-dns.md) | Make sure DNS is properly configured so that you can reach the local portal domain for Agentic RAG. |

## Related content

- [Deployment overview for Agentic RAG](deploy-overview.md)
- [Deploy the Agentic RAG extension](deploy.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc)