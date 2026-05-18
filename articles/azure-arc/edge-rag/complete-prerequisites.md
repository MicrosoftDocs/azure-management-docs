---
title: Deployment Prerequisites Checklist for Agents and Tools with Foundry Local
description: "Learn how to complete deployment prerequisites for Agents and Tools with Foundry Local to ensure a successful setup for your chat solution."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/18/2026
ai-usage: ai-generated
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator or AI application developer, I want to complete the deployment prerequisites for Agents and Tools with Foundry Local so that I can ensure a successful setup and configuration of the environment for running AI-powered applications."
---

# Deployment prerequisites checklist for Agents and Tools with Foundry Local

Complete the following steps to prepare for your Agents and Tools with Foundry Local deployment. 

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]


## Prerequisites checklist

Use this checklist to prepare your environment before deploying Agents and Tools with Foundry Local on Azure Arc. Each step links to detailed instructions or supporting documentation.

| Step | Task | Description |
|------|------|-------------|
| 1 | [Verify your environment meets requirements](requirements.md) | Before you begin, review [What you need for Agents and Tools with Foundry Local](requirements.md). |
| 2 | Choose your deployment mode | Decide which deployment mode you need: **combined** (default, full platform), **agentic** (agents only, no local data ingestion), or **knowledge** (data ingestion and RAG only, no agents). Your choice affects which prerequisites apply. For example, agentic mode doesn't require GPUs or a network files system (NFS) data source. |
| 3 | [Verify you have the Contributor role](prepare-contributor-permission.md) | Verify that you have the **Contributor** role at the subscription level. This role is required to register providers and enable features. |
| 4 | [Choose your language model (BYOM)](prepare-language-model.md) | Agents and Tools with Foundry Local requires you to Bring Your Own Model (BYOM). Choose a model and set up an OpenAI-compatible LLM endpoint before deployment. See [Choose your language model](prepare-language-model.md). |
| 5 | [Create a BYOM endpoint](prepare-model-endpoint.md) | Set up an OpenAI API-compatible endpoint for your language model. This step is **mandatory** for all deployments. See [Create a BYOM endpoint](prepare-model-endpoint.md). |
| 6 | [Verify file share access](prepare-file-server.md) | Make sure your documents and images are hosted on a supported and reachable NFS server. **Required for combined and knowledge modes only.** Skip this step if deploying in agentic mode. |
| 7 | [(Optional) Set up NFS with Kerberos authentication](connect-nfs-kerberos-overview.md) | If you use NFS with Kerberos (`krb5p`) authentication for disconnected on-premises deployments, complete the Kerberos setup before deployment. **Optional** — skip if you use AUTH_SYS authentication. See [NFS with Kerberos overview](connect-nfs-kerberos-overview.md). |
| 8 | [Set up SharePoint Server S2S authentication](connect-sharepoint-overview.md) | If you use SharePoint Server as a data source, configure High-Trust Server-to-Server (S2S) authentication before deployment. **Required when using SharePoint data sources.** See [SharePoint S2S overview](connect-sharepoint-overview.md). |
| 9 | [Prepare an Azure Kubernetes (AKS) cluster on Azure Local](prepare-aks-cluster.md) | Create an AKS Arc cluster on your Azure Local instance with a node pool that meets minimum requirements. |
| 10 | [Configure authentication](prepare-authentication.md) | Configure access to Agents and Tools with Foundry Local for AI application developers and for end users of the chat endpoint. |
| 11 | [Install networking and observability components](prepare-networking-observability.md) | Deploy required components for network routing, monitoring, and logging. |
| 12 | [Configure DNS](prepare-dns.md) | Make sure DNS is properly configured so that you can reach the local portal domain for Agents and Tools with Foundry Local. |

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#agents-and-tools-with-foundry-local)