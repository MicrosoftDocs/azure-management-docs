---
title: Agents and Tools with Foundry Local Deployment Overview
description: Learn about deploying Agents and Tools with Foundry Local with Azure Arc, including prerequisites, configuration options, and steps for secure, scalable AI at the edge.
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/08/2026
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As an IT administrator or cloud architect, I want to learn about deploying and configuring Agents and Tools with Foundry Local with Azure Arc so that I can enable a secure, scalable AI-powered chat solution using my organization's data at the edge.
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
---

# Deployment overview for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local brings agentic AI capabilities to your edge and hybrid environments. The `indexedSourcesEnabled` setting controls whether the local indexed-source stack is installed with the core Agent Service workloads. This article describes the Azure Arc deployment process, components, and workflow.

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Deploy Agents and Tools with Foundry Local](quickstart-edge-rag.md).

## Agents and Tools with Foundry Local architecture and components

The following diagram shows the key components and architecture you have after you prepare your environment and deploy Agents and Tools with Foundry Local. It includes Azure resources, on-premises infrastructure, and integration points.

<!-- Art Library Source# ConceptArt-0-000-2175 -->
:::image type="content" source="media/deploy-overview/agentic-retrieval-deployment-components.svg" alt-text="Diagram that shows the Agents and Tools with Foundry Local deployment architecture, including Azure resources, on-premises infrastructure, and integration points." border="false":::

The resources and components in the diagram form the core infrastructure for Agents and Tools with Foundry Local:

- **Azure resources:**
  Microsoft Entra ID and Azure Arc provide identity, access, and hybrid management.

- **On-premises infrastructure:**
  These components provide the compute, networking, load balancing, and storage that you need to run and access the extension locally:

  - Azure Local*
  - Azure Kubernetes Service (AKS) cluster on Azure Local*
  - MetalLB
  - AKS node pool
  - Optional data sources, such as a network file share (NFS) server
  - Driver machine (local management host)
  
  \* Agents and Tools with Foundry Local is validated on Azure Local.

- **Core Agent Service** (deployed in every current mode):
  - Agents management — agents, agent versions, connections, and knowledge bases
  - Agents execution — Responses and Conversations APIs, SSE streaming
  - Knowledge base execution — knowledge base retrieval for agents
  - Knowledge Sources service (port 3005) — knowledge source registration and validation
  - Local chat UI (`/chat`) — end-user conversational interface

- **Local indexed-source stack** (when `indexedSourcesEnabled=true`):
  - Ingestion API (port 8000) — document parsing, chunking, embedding
  - Inference API (port 3001) — RAG chat completions
  - Collections API (port 3002) — vector data management
  - Indexed Sources MCP (port 8080) — built-in search tools over Model Context Protocol
  - Embedding and hybrid search models (UAE-Large-V1, BGE-M3, CLIP ViT-L/14) — running on CPU or 2 GPUs (when gpu_enabled=true)
  - Docling parser — running on CPU
  - Milvus vector database + PostgreSQL metadata store

- **Language model endpoint**
  - Foundry Local on Azure Local (recommended)
  - External Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry

  Use a driver machine (local management host) to simplify management of the Azure Arc-enabled Kubernetes cluster on Azure Local. For more information, see  [Prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local](prepare-aks-cluster.md) and [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).

This setup lets you run a secure, scalable, AI-powered chat solution that uses your own data at the edge.

## Key configuration options

When you deploy Agents and Tools with Foundry Local, set several configuration options to tailor the solution to your environment and requirements.

- **Language model endpoint for indexed sources:** When `indexedSourcesEnabled=true`, provide an OpenAI-compatible endpoint for the local retrieval and inference workloads. When indexed sources are disabled, configure models through Agent Service model connections after deployment.
- **Indexed sources:** On the **Components** tab in the Azure portal, select **Enabled** or **Disabled** in the required **Enable indexed sources** dropdown (the portal template defaults `indexedSourcesEnabled` to `false`, whereas Azure CLI defaults to `true`). When enabled, select **Configure** to choose the **CPU (recommended)** or **GPU** processing mode and optional NFS Kerberos settings, and then select **Save**. Choose **Disabled** (or `indexedSourcesEnabled=false` in Azure CLI) to deploy the core Agent Service without the local indexed-source stack. Deployments that need `foundryClientId` or `byom.*` settings must set them with Azure CLI.
- **Networking:** Use the built-in Envoy Gateway, or set `networking.mode=none` and `networking.envoy.enabled=false` for a customer-managed ingress controller.
- **SSL and domain configuration:** On the **Connection + Access** tab (or in Azure CLI), select **Configure** for **Extension address and HTTPS**, enter the extension hostname and Kubernetes TLS secret name so you can securely access the gateway and chat endpoints, and select **Save**.
- **Access and authentication:** On the **Connection + Access** tab (or in Azure CLI), select **Configure** for **User sign-in with Microsoft Entra ID** (and **Trusted services** if models or tools use managed identity), enter the Microsoft Entra application (client) ID, directory (tenant) ID, and any approved service URLs, and select **Save**. After deployment, assign Azure roles to callers and **Reader** to the extension's managed identity at the extension scope.
- **Data source configuration:** Configure only the source-specific network, identity, and credential settings required by the data sources that you use.

### Required roles for API access

Azure role-based access control (Azure RBAC) is the primary model for API access. An administrator with role-assignment permission assigns these roles at the extension resource scope:

| Who | Connected deployment | Disconnected deployment |
|---|---|---|
| Extension's system-assigned managed identity | **Reader**, to read role definitions, assignments, and deny assignments | **Reader**, for the same checks against the stamp's local Azure Resource Manager endpoint |
| People who read resource settings | **Reader** | **Reader** |
| People who create, change, or delete resources | **Cognitive Services Contributor** | **Contributor** |
| People and apps that run agents or use chat | **Cognitive Services OpenAI User**, plus **Reader** to list agents | **Contributor** |

This setup is required even when your model uses an API key. The portal doesn't assign Reader to the extension identity. If the identity can't read role assignments, Responses and Conversations return `503` with `Authorization service unavailable`. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles) and [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

App roles remain a fallback for most management and classic APIs, but not for Responses or Conversations. Inherited Azure assignments at subscription, resource-group, or cluster scope also apply. Prefer extension-scope assignments for least privilege.

If indexed sources or agents also use Foundry Local with managed identity (`foundryClientId`), set up the separate model-access roles in the following section.

### Required roles for Foundry Local inference

If you deploy the Agents and Tools with Foundry Local extension to use a Foundry Local endpoint (`foundryClientId`), assign the following roles before you test chat or run inference traffic. These roles let each part of the request pass security checks, and if one is missing, requests can fail with `403` or `500` errors.
These model-access checks apply only when you use Foundry Local with `foundryClientId`. They are separate from the API-access roles required for every authenticated deployment.

#### Request flow

When Agents and Tools with Foundry Local calls the Foundry Local endpoint with a managed identity token, Foundry Local first validates the token. The token audience must be the Foundry Local application client ID (`foundryClientId`). If the token isn't valid, Foundry Local returns `401`. Then the request passes through two authorization checks:

- **Layer 1 - ARM Reader**: The Foundry inference pod queries Azure Resource Manager (ARM) to verify the caller's role assignments. The connected cluster managed identity needs `Reader` on its own cluster resource for this lookup to succeed.
- **Layer 2 - Azure RBAC**: ARM checks the Foundry operator and Agents and Tools identities. Use `Reader` on the connected cluster and the inference role on the Foundry extension resource.

#### Layer summary

| Layer | Who | Role | Purpose | Without it |
|-------|-----|------|---------|------------|
| **1 - ARM Reader** | Connected cluster managed identity &rarr; ARM | `Reader` on the cluster resource | Foundry pod reads ARM role assignments to validate callers. | `500` with `rbac_check_unavailable` |
| **2 - Azure RBAC** | Agents and Tools + Foundry managed identities &rarr; ARM | `Reader` on the connected cluster and the inference role on the Foundry extension: `Cognitive Services OpenAI User` in connected deployments, or `Contributor` in disconnected deployments | Resource-level authorization for model inference. | `403` with `insufficient_permissions` |

#### Identity-based role assignment timing

Role assignments require principal IDs that exist only after the resources are deployed. Assign each layer as soon as the relevant identity exists:

| Deployment step | What it creates | What you can assign after |
|-----------------|----------------|--------------------------|
| `az connectedk8s connect` | Connected cluster managed identity | **Layer 1**: ARM `Reader` for the cluster managed identity |
| `az k8s-extension create` (inference-operator) | Foundry operator managed identity | **Layer 2**: `Reader` for the Foundry operator managed identity |
| `az k8s-extension create` (Agents and Tools) | Agents and Tools extension managed identity | **Layer 2**: `Reader` on the cluster plus the inference role on the Foundry extension |

You can also defer all role assignments until after all extensions are installed and assign them together. For role assignment steps, see [Configure Foundry Local inference authentication for Agents and Tools with Foundry Local](configure-foundry-inference-authentication.md).

## Deployment process for Agents and Tools with Foundry Local

The deployment process for Agents and Tools with Foundry Local consists of the following high-level steps:

| High-level step  | Description |
|-----------------|-----------------------------------------------------------|
| 1. Prepare the environment               | Set up the required Azure and on-premises infrastructure, configure your AKS Arc cluster and node pools, establish networking and RWX storage, and set up authentication and user roles. Review the [requirements](requirements.md) and complete the [prerequisites checklist](complete-prerequisites.md). If indexed sources are enabled, [set up the deployment-level language model endpoint](prepare-model-endpoint.md). <br><br>If you're using [Microsoft Azure Government](/azure/azure-government/documentation-government-welcome), see [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#agents-and-tools-with-foundry-local) for the deployment variations with Agents and Tools with Foundry Local. |
| 2. Deploy the Agents and Tools with Foundry Local extension         | Use the Azure portal or CLI to install the extension on your AKS Arc cluster. On the **Components** tab, choose whether to enable indexed sources and configure the processing mode and optional NFS Kerberos settings. On the **Connection + Access** tab, configure the extension hostname, Kubernetes TLS secret name, and Microsoft Entra ID settings. Add deployment-level model settings in Azure CLI only when indexed sources are enabled. Set `foundryClientId` whenever indexed sources or agents use Foundry Local on this cluster. See [Deploy the Agents and Tools with Foundry Local extension](deploy.md). If your Azure Local environment uses disconnected operations, see [Agents and Tools with Foundry Local for disconnected operations on Azure Local overview](disconnected-operations/overview-disconnected.md) and [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md) instead. |
| 3. Assign roles and validate the deployment | After deployment, [assign Reader to the extension identity](deploy.md#allow-the-extension-to-check-azure-roles) and [assign Azure roles to callers](prepare-authentication.md#assign-azure-roles). Complete separate model-access grants when needed. Then check the pods and test the gateway and chat endpoints. See [Verify the deployment](deploy.md#verify-the-deployment). |
| 4. Configure Agentic Retrieval         | If indexed sources are enabled, configure Agentic Retrieval: set up collections or use the default collection, add data sources, and test the setup. See [Agentic Retrieval overview](knowledge-layer-overview.md), [Add a data source](add-data-source.md), and [Test the end-user query experience](test-end-user-app.md). |
| 5. Configure agents (optional) | Create a model connection and an agent, then add knowledge bases and MCP servers as tools. See [Quickstart: Create an agent and chat with it](quickstart-create-agent.md). |
| 6. Monitor your deployment | After deploying Agents and Tools with Foundry Local, monitor system health and performance. See [Monitor Agents and Tools with Foundry Local](observability.md). |

## Related content

- [Requirements for Agents and Tools with Foundry Local](requirements.md)
- [Deployment Prerequisites Checklist](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
- [Disconnected operations on Azure Local overview](disconnected-operations/overview-disconnected.md)
- [Add a data source for Agents and Tools with Foundry Local](add-data-source.md)
- [Agentic Retrieval overview](knowledge-layer-overview.md)
