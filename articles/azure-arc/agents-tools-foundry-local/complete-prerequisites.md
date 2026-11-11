---
title: Deployment Prerequisites Checklist for Agents and Tools with Foundry Local
description: "Learn how to complete deployment prerequisites for Agents and Tools with Foundry Local to ensure a successful setup for your chat solution."
ms.reviewer: assafakiva
ms.topic: concept-article #Don't change
ms.date: 11/05/2026
ai-usage: ai-generated
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator or AI application developer, I want to complete the deployment prerequisites for Agents and Tools with Foundry Local so that I can ensure a successful setup and configuration of the environment for running AI-powered applications."
---

# Deployment prerequisites checklist for Agents and Tools with Foundry Local

Complete the following steps to prepare for your Agents and Tools with Foundry Local deployment. If your Azure Local environment uses disconnected operations, see [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md), which covers prerequisites and installation in a single article.

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Deploy Agents and Tools with Foundry Local](quickstart-edge-rag.md).


## Prerequisites checklist

Use this checklist to prepare your environment before deploying Agents and Tools with Foundry Local on Azure Arc. Each step links to detailed instructions or supporting documentation.

| Step | Task | Description |
|------|------|-------------|
| 1 | [Verify your environment meets requirements](requirements.md) | Before you begin, review [What you need for Agents and Tools with Foundry Local](requirements.md). |
| 2 | Choose whether to enable indexed sources | The core Agent Service is always deployed. Keep indexed sources enabled (the default) when you need local ingestion, collections, and Indexed Sources MCP. Disable indexed sources when you only need agents and other configured tools. |
| 3 | [Verify you have the Contributor role](prepare-contributor-permission.md) | Verify that you have the **Contributor** role at the subscription level. This role is required to register providers and enable features. |
| 4 | [Choose your language model](prepare-language-model.md) | If indexed sources are enabled, choose a language model for retrieval-augmented generation. Agent Service models are configured through model connections after deployment. |
| 5 | [Create an endpoint](prepare-model-endpoint.md) | If indexed sources are enabled, set up an OpenAI API-compatible endpoint. If you use Foundry Local, install the Foundry Local on Azure Local extension first and then use its model endpoint URL. When `indexedSourcesEnabled=false`, you don't pass an endpoint during deployment, but agents still need one for their model connections. If agents use Foundry Local on this cluster, set `foundryClientId` during deployment. |
| 6 | [Verify file share access](prepare-file-server.md) | Complete this step only when you plan to ingest documents or images from NFS. Other source types have their own prerequisites. |
| 7 | [(Optional) Set up NFS with Kerberos authentication](connect-file-share-kerberos-overview.md) | If you use NFS with Kerberos (`krb5p`) authentication for disconnected on-premises deployments, complete the Kerberos setup before deployment. **Optional** — skip if you use AUTH_SYS authentication. See [NFS with Kerberos overview](connect-file-share-kerberos-overview.md). |
| 8 | [Set up SharePoint Server-to-Server authentication](connect-sharepoint-overview.md) | If you use SharePoint Server as an indexed ingestion source, configure High-Trust Server-to-Server (S2S) authentication and create the Kubernetes signing Secret that each source selects. |
| 9 | [Prepare an Azure Kubernetes Service (AKS) cluster on Azure Local](prepare-aks-cluster.md) | Create an AKS Arc cluster on your Azure Local instance with a node pool that meets minimum requirements. |
| 10 | [Prepare RWX cache claims for Microsoft 365 Local (optional)](prepare-microsoft-365-local.md#shared-file-cache-requirements) | Only if you use Microsoft 365 Local. Prepare an RWX persistent volume claim for each provider that you use, and set it with `sharepointMcp.cache.claimName` or `exchangeMcp.cache.claimName` in the Azure CLI deployment. |
| 11 | [Configure authentication](prepare-authentication.md) | Register the Microsoft Entra application and prepare fallback app roles. Arrange permission to assign Azure roles after deployment. |
| 12 | [Install networking and observability components](prepare-networking-observability.md) | Prepare routing, monitoring, and logging. Decide whether to use the built-in Envoy Gateway (`networking.mode=envoy`) or your own ingress controller (`networking.mode=none` with `networking.envoy.enabled=false`). |
| 13 | [Configure DNS](prepare-dns.md) | Make sure DNS is properly configured so that you can reach the local portal domain for Agents and Tools with Foundry Local. |

## After deployment

Before you test API calls or chat, [assign Reader to the extension's system-assigned managed identity](deploy.md#allow-the-extension-to-check-azure-roles) at the extension resource scope. The portal doesn't create this assignment. Without it, Responses and Conversations can return `503` with `Authorization service unavailable`.

Also [assign Azure roles to callers](prepare-authentication.md#assign-azure-roles). In connected deployments, use **Reader** for reads, **Cognitive Services Contributor** for changes, and **Cognitive Services OpenAI User** for chat and agent execution. In disconnected deployments, use **Reader** for reads and **Contributor** for changes and chat. These API-access roles are separate from any Foundry Local model-access grants.

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#agents-and-tools-with-foundry-local)