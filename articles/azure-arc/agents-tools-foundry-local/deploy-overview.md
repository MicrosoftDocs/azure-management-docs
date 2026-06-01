---
title: Agentic Retrieval in Foundry Local Deployment Overview
description: Learn about deploying Agentic Retrieval in Foundry Local with Azure Arc, including prerequisites, configuration options, and steps for secure, scalable AI at the edge.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 06/01/2026
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As an IT administrator or cloud architect, I want to learn about deploying and configuring Agentic Retrieval in Foundry Local with Azure Arc so that I can enable a secure, scalable AI-powered chat solution using my organization's data at the edge.
---

# Deployment overview for Agentic Retrieval in Foundry Local

Agentic Retrieval brings agentic AI capabilities to your edge and hybrid environments. Deploy a secure, scalable, and intelligent AI agent platform that uses your own data. Deploy the full Agentic Retrieval platform, or independently deploy just the Agentic Layer or Knowledge Layer based on your needs. This article provides an overview of the deployment process, key components, and workflow to deploy Agentic Retrieval with Azure Arc.

To try Agentic Retrieval without the need for local hardware, see [Quickstart: Install Agentic Retrieval](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Agentic Retrieval architecture and components

The following diagram shows the key components and architecture you have after you prepare your environment and deploy Agentic Retrieval. It includes Azure resources, on-premises infrastructure, and integration points.

<!-- Art Library Source# ConceptArt-0-000-2175 -->
:::image type="content" source="media/deploy-overview/agentic-retrieval-deployment-components.svg" alt-text="Diagram that shows the Agentic Retrieval deployment architecture, including Azure resources, on-premises infrastructure, and integration points." border="false":::

The resources and components in the diagram form the core infrastructure for Agentic Retrieval:

- **Azure resources:**
  Microsoft Entra ID and Azure Arc provide identity, access, and hybrid management.

- **On-premises infrastructure:**
  These components provide the compute, networking, load balancing, and storage that you need to run the Agentic Retrieval extension and to access Agentic Retrieval locally:

  - Azure Local*
  - Azure Kubernetes (AKS) cluster on Azure Local*
  - MetalLB
  - AKS node pool
  - Network file share (NFS) server
  - Driver machine (local management host)
  
  \* Agentic Retrieval is validated on Azure Local.

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

- **Language model endpoint**
  - Foundry Local on Azure Local (recommended)
  - External Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry

  Use a driver machine (local management host) to simplify management of the Azure Arc-enabled Kubernetes cluster on Azure Local. For more information, see  [Prepare AKS cluster on Azure Local for Agentic Retrieval](prepare-aks-cluster.md) and [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).

This setup lets you run a secure, scalable, AI-powered chat solution that uses your own data at the edge.

## Key configuration options

When you deploy Agentic Retrieval, set several configuration options to tailor the solution to your environment and requirements.

- **Language model endpoint (mandatory):** Agentic Retrieval doesn't bundle language models. You must provide your own LLM endpoint that exposes an OpenAI-compatible chat completions API. The recommended option is a [Foundry Local on Azure Local](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local) endpoint. Or use an external Bring Your Own Model (BYOM) endpoint, such as one deployed in [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) for cloud-hosted models.
- **Deployment mode:** Choose which layers to deploy by using the `layerSelection` parameter:
  - `combined` (default) — full platform with both layers
  - `agentic` — agents, knowledge bases, KT/KS, MCP server only (no GPUs required)
  - `knowledge` — ingestion, inference, collections only (2 GPUs required)
- **SSL and domain configuration:** Set up a Transport Layer Security (TLS) termination certificate and domain so you can securely access the chat endpoint.
- **Access and authentication:** Set up the Microsoft Entra app ID and assign roles to users and groups.
- **Data source configuration:** Make sure your data source, an NFS share, is reachable and contains the required files in supported formats.

If you use Foundry Local managed identity authentication (`foundryClientId`), include the following authorization configuration in your deployment plan.

### Foundry Local inference authentication layers

When you use managed identity authentication (`foundryClientId`), Foundry Local inference requires three role assignment layers. Each layer protects a different hop in the request chain, and all three are required.

#### Request flow

When Agents and Tools calls the Foundry Local endpoint with a managed identity token, the request passes through three authorization checks:

- **Layer 1 - Entra app role**: The Foundry app registration validates that the caller's managed identity has the `FoundryInferenceAccess` app role. Without it, the token is valid but missing the required `roles` claim.
- **Layer 2 - ARM Reader**: The Foundry inference pod queries Azure Resource Manager (ARM) to verify the caller's role assignments. The connected cluster managed identity needs `Reader` on its own cluster resource for this lookup to succeed.
- **Layer 3 - Azure RBAC**: ARM checks that the caller has the required Azure roles (`Cognitive Services OpenAI User` and `Reader`) at the subscription or resource group level.

#### Layer summary

| Layer | Who | Role | Purpose | Without it |
|-------|-----|------|---------|------------|
| **1 - Entra app role** | Agents and Tools managed identity &rarr; Foundry app registration | `FoundryInferenceAccess` app role | App-level gate for calling Foundry Local. | `401` token valid but no app role |
| **2 - ARM Reader** | Connected cluster managed identity &rarr; ARM | `Reader` on the cluster resource | Foundry pod reads ARM role assignments to validate callers. | `401` ARM lookup fails |
| **3 - Azure RBAC** | Agents and Tools + Foundry managed identities &rarr; ARM | `Cognitive Services OpenAI User` + `Reader` | Resource-level authorization for model inference. | `403` ARM denies authorization |

#### Assign each layer after identity creation

Role assignments require principal IDs that exist only after the resources are deployed. Assign each layer as soon as the relevant identity exists:

| Deployment step | What it creates | What you can assign after |
|-----------------|----------------|--------------------------|
| `az connectedk8s connect` | Connected cluster managed identity | **Layer 2**: ARM `Reader` for the cluster managed identity |
| `az k8s-extension create` (inference-operator) | Foundry operator managed identity | **Layer 3**: `Reader` for the Foundry operator managed identity |
| `az k8s-extension create` (Agents and Tools) | Agents and Tools extension managed identity | **Layer 1**: Entra app role and **Layer 3**: `Cognitive Services OpenAI User` and `Reader` |

You can also defer all role assignments until after all extensions are installed and assign them together. For role assignment steps, see [Configure Foundry Local inference authentication for Agentic Retrieval](configure-foundry-inference-authentication.md).

Azure RBAC roles alone aren't enough. The Foundry authentication sidecar validates Microsoft Entra ID app roles, not only Azure RBAC. Without the `FoundryInferenceAccess` app role assignment, managed identity tokens are valid but missing the `roles` claim.

## Deployment process for Agentic Retrieval

The deployment process for Agentic Retrieval consists of the following high-level steps:

| High-level step  | Description |
|-----------------|-----------------------------------------------------------|
| 1. Prepare the environment               | Set up the required Azure and on-premises infrastructure, configure your AKS Arc cluster and node pools, establish networking and storage, and set up authentication and user roles. Review the [requirements](requirements.md) and complete the [prerequisites checklist](complete-prerequisites.md). <br><br>As part of the prerequisites, [set up your language model endpoint (mandatory for all deployments)](prepare-model-endpoint.md). <br><br>If you're using [Microsoft Azure Government](/azure/azure-government/documentation-government-welcome), see [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc) for the deployment variations with Agentic Retrieval. |
| 2. Deploy the Agentic Retrieval extension         | Use the Azure portal or CLI to install the extension on your AKS Arc cluster. Choose your deployment mode (Agentic, Knowledge, or Combined), add your language model endpoint, set up security and access parameters, and connect the extension to your Microsoft Entra ID for authentication. See [Deploy the Agentic Retrieval extension](deploy.md). <br><br>After deployment, configure authentication based on your language model source: [Configure Foundry Local inference authentication for Agentic Retrieval](configure-foundry-inference-authentication.md) when you use `foundryClientId`, or [configure BYOM endpoint authentication for Agentic Retrieval](configure-endpoint-authentication.md) when you use BYOM.|
| 3. Validate the deployment               | After you deploy the extension, check that the Agentic Retrieval extension is installed and running on your cluster and that you have connectivity to the chat endpoint.                                                                |
| 4. Configure knowledge layer         | If you deployed in combined or knowledge mode, configure the knowledge layer: set up collections or use the default collection, add data sources, and test the setup. See [Knowledge layer configuration](knowledge-layer-overview.md), [Add a data source](add-data-source.md), and [Test the end-user query experience](test-end-user-app.md). |
| 5. Configure agents (optional) | If you deployed in combined or agentic mode, create knowledge sources and link them to your default knowledge base to build intelligent assistants. See the [Query your data quickstart](quickstart-edge-rag.md). |
| 6. Monitor and evaluate your deployment  | After deploying Agentic Retrieval, monitor system health, track performance, and evaluate the quality of your AI solution. Use built-in metrics and evaluation tools to observe, assess, and optimize your deployment. See [Evaluate the Agentic Retrieval system](evaluate-solution.md) and [Monitor Agentic Retrieval](observability.md). |

## Related content

- [Requirements for Agentic Retrieval](requirements.md)
- [Deployment Prerequisites Checklist](complete-prerequisites.md)
- [Deploy the Agentic Retrieval extension](deploy.md)
- [Add a data source for Agentic Retrieval](add-data-source.md)
- [Knowledge layer configuration](knowledge-layer-overview.md)