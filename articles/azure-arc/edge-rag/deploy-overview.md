---
title: Agents and Tools with Foundry Local Deployment Overview
description: Learn about deploying Agents and Tools with Foundry Local with Azure Arc, including prerequisites, configuration options, and steps for secure, scalable AI at the edge.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/04/2026
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As an IT administrator or cloud architect, I want to learn about deploying and configuring Agents and Tools with Foundry Local with Azure Arc so that I can enable a secure, scalable AI-powered chat solution using my organization's data at the edge.
---

# Deployment overview for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local brings agentic AI capabilities to your edge and hybrid environments. Deploy a secure, scalable, and intelligent AI agent platform that uses your own data. Deploy the full Agents and Tools with Foundry Local platform, or independently deploy just the Agentic Layer or Knowledge Layer based on your needs. This article provides an overview of the deployment process, key components, and workflow to deploy Agents and Tools with Foundry Local with Azure Arc.

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Agents and Tools with Foundry Local architecture and components

The following diagram shows the key components and architecture you have after you prepare your environment and deploy Agents and Tools with Foundry Local. It includes Azure resources, on-premises infrastructure, and integration points.

<!-- Art Library Source# ConceptArt-0-000-95 -->
:::image type="content" source="media/deploy-overview/deployment-components.svg" alt-text="Diagram that shows the Agents and Tools with Foundry Local deployment architecture, including Azure resources, on-premises infrastructure, and integration points." border="false":::

The resources and components in the diagram form the core infrastructure for Agents and Tools with Foundry Local:

- **Azure resources:**
  Microsoft Entra ID and Azure Arc provide identity, access, and hybrid management.

- **On-premises infrastructure:**
  These components provide the compute, networking, load balancing, and storage that you need to run the Agents and Tools with Foundry Local extension and to access Agents and Tools with Foundry Local locally:

  - Azure Local*
  - Azure Kubernetes (AKS) cluster on Azure Local*
  - MetalLB
  - AKS node pool
  - Network file share (NFS) server
  - Driver machine (local management host)
  
  \* Agents and Tools with Foundry Local is validated on Azure Local.

- **Agentic Layer** (when deployed in combined or agentic mode):
  - Agents Runtime (port 8080) — threads, messages, runs, SSE streaming
  - Knowledge Base Manager (port 8080) — knowledge bases management (GET, PATCH, PUT)
  - Knowledge Sources service (port 3005) — MCP server connection registration
  - MCP Server (port 8080) — 6 built-in search tools over Model Context Protocol

- **Knowledge Layer** (when deployed in combined or knowledge mode):
  - Ingestion API (port 8000) — document parsing, chunking, embedding
  - Inference API (port 3001) — RAG chat completions
  - Collections API (port 3002) — vector data management
  - Embedding models (BGE-M3, CLIP ViT-L/14) — running on 2 GPUs
  - Docling parser — running on CPU
  - Milvus vector database + PostgreSQL metadata store

- **External** (customer-managed):
  - LLM endpoint (BYOM) — Foundry Local on Azure Local or Microsoft Foundry

  Use a driver machine (local management host) to simplify management of the Azure Arc-enabled Kubernetes cluster on Azure Local. For more information, see  [Prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md) and [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).

This setup lets you run a secure, scalable, AI-powered chat solution that uses your own data at the edge.

## Key configuration options

When you deploy Agents and Tools with Foundry Local, set several configuration options to tailor the solution to your environment and requirements.

- **Language model (BYOM - mandatory):** Agents and Tools with Foundry Local doesn't bundle any language models. You must provide your own LLM endpoint (Bring Your Own Model) that exposes an OpenAI-compatible chat completions API. Recommended options: [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local) or [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) for cloud-hosted models.
- **Deployment mode:** Choose which layers to deploy by using the `layerSelection` parameter:
  - `combined` (default) — full platform with both layers
  - `agentic` — agents, knowledge bases, KT/KS, MCP server only (no GPUs required)
  - `knowledge` — ingestion, inference, collections only (2 GPUs required)
- **SSL and domain configuration:** Set up a Transport Layer Security (TLS) termination certificate and domain so you can securely access the chat endpoint.
- **Access and authentication:** Set up the Microsoft Entra app ID and assign roles to users and groups.
- **Data source configuration:** Make sure your data source, an NFS share, is reachable and contains the required files in supported formats.

## Deployment process for Agents and Tools with Foundry Local

The deployment process for Agents and Tools with Foundry Local consists of the following high-level steps:

| High-level step  | Description |
|-----------------|-----------------------------------------------------------|
| 1. Prepare the environment               | Set up the required Azure and on-premises infrastructure, configure your AKS Arc cluster and node pools, establish networking and storage, and set up authentication and user roles. Review the [requirements](requirements.md) and complete the [prerequisites checklist](complete-prerequisites.md). <br><br>As part of the prerequisites, [set up your BYOM language model endpoint (mandatory for all deployments)](prepare-model-endpoint.md). <br><br>If you're using [Microsoft Azure Government](/azure/azure-government/documentation-government-welcome), see [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc) for the deployment variations with Agents and Tools with Foundry Local. |
| 2. Deploy the Agents and Tools with Foundry Local extension         | Use the Azure portal or CLI to install the extension on your AKS Arc cluster. Choose your deployment mode (Agentic, Knowledge, or Combined), configure your BYOM endpoint, set up security and access parameters, and connect the extension to your Microsoft Entra ID for authentication. See [Deploy the Agents and Tools with Foundry Local extension](deploy.md). <br><br>After deployment, [configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md).|
| 3. Validate the deployment               | After you deploy the extension, check that the Agents and Tools with Foundry Local extension is installed and running on your cluster and that you have connectivity to the chat endpoint.                                                                |
| 4. Configure Knowledge Layer               | If you deployed in combined or knowledge mode, configure the Knowledge Layer: set up collections, add data sources, configure the user experience, and test the setup. See [Add a data source for Agents and Tools with Foundry Local](add-data-source.md), [Configure the chat solution](build-chat-solution-overview.md), and [Test the chat solution](test-end-user-app.md). |
| 5. Configure agents (optional) | If you deployed in combined or agentic mode, create knowledge sources and link them to your default knowledge base to build intelligent assistants. See the [Query your data quickstart](quickstart-edge-rag.md). |
| 6. Monitor and evaluate your deployment  | After deploying Agents and Tools with Foundry Local, monitor system health, track performance, and evaluate the quality of your AI solution. Use built-in metrics and evaluation tools to observe, assess, and optimize your deployment. See [Evaluate the Agents and Tools with Foundry Local system](evaluate-solution.md) and [Monitor Agents and Tools with Foundry Local](observability.md). |

## Related content

- [Requirements for Agents and Tools with Foundry Local](requirements.md)
- [Deployment Prerequisites Checklist](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
- [Add a data source for Agents and Tools with Foundry Local](add-data-source.md)
- [Configure the chat solution](build-chat-solution-overview.md)