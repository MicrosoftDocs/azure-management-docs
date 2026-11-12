---
title: Disconnected Operations for Agents and Tools with Foundry Local Overview
description: "Learn how Agents and Tools with Foundry Local deployments differ in disconnected operations on Azure Local, including expansion packs, authentication, and architecture."
ms.reviewer: assafakiva
ms.topic: overview #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
#CustomerIntent: As a cloud administrator, I want to understand how Agents and Tools with Foundry Local differs in a disconnected Azure Local environment so that I can plan a secure, offline deployment.
---

# Disconnected operations for Agents and Tools with Foundry Local

You can deploy [Agents and Tools with Foundry Local](../overview.md) on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview) by using a deployment model that largely matches connected scenarios. However, several key differences exist when internet connectivity isn't available.

This article explains how disconnected deployments of Agents and Tools with Foundry Local differ from connected deployments, so you can plan a secure, offline agentic Retrieval-Augmented Generation (RAG) solution at the edge.

[!INCLUDE [disconnected-operations-preview-notice](../includes/disconnected-operations-preview-notice.md)]

## What changes in disconnected deployments

In disconnected environments, extension availability, model and container image sourcing, certificate and networking dependencies, identity, and access flows differ from connected deployments.

- **Extension availability**: Before you can install the Agents and Tools with Foundry Local extension (`microsoft.arc.rag`), you must download the Agents and Tools with Foundry Local expansion pack in a connected environment, transfer it to the disconnected environment, and import it on the Azure Local Disconnected Operations machine. After import, the extension type is registered locally and becomes available for installation. For more information, see [Download and import the Agents and Tools with Foundry Local expansion pack](prepare-disconnected.md).
- **Container image and model artifact source**: The expansion pack imports the extension's container images into the local `edgeartifacts` container registry and publishes model artifacts to the registry. Deployments pull these assets locally instead of from internet-connected registries.
- **Language model endpoint**: The recommended option is a Foundry Local on Azure Local endpoint running on the same disconnected cluster. You can also use an external Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API and is reachable from the disconnected environment.
- **Authentication**: End-user sign-in doesn't use public Microsoft Entra ID endpoints. Instead, you create the Microsoft Entra application registration on the disconnected stamp by using the `az` CLI and the disconnected Microsoft Graph endpoint, and you register the sign-in redirect URIs for your ingress domain.
- **Authorization**: When you use managed identity authentication to Foundry, model calls use the Agents and Tools with Foundry Local extension identity. Assign that identity `Reader` on the connected-cluster resource and **Contributor** on the Foundry extension resource for inference. The connected-cluster and Foundry extension identities also need `Reader` for ARM role validation. Role propagation can take longer than in connected environments.
- **Networking**: The extension requires a policy-enforcing container network interface (CNI) such as Calico, Cilium, or Azure NPM, because Agents and Tools with Foundry Local uses Kubernetes network policies by default. The ingress load-balancer IP isn't baked into the expansion pack. You supply an IP from your cluster's ingress subnet at install time by using MetalLB.

## Agent Service in disconnected deployments

Agent Service works the same way in disconnected deployments. You use the same Agents, Responses, Conversations, Knowledge Bases, and Knowledge Sources APIs. Keep these differences in mind:

- **Model connections**: After installation, create a model connection to a model that runs on or near the disconnected cluster, usually Foundry Local with the `FoundryOnArc` kind. Models in Microsoft Foundry (`FoundryCloud`) need internet access and aren't available. See [Create a model connection](../create-manage-agents.md#create-a-model-connection).
- **Tools**: Agents can only call MCP servers that the disconnected cluster can reach. Microsoft Foundry toolboxes run in Azure, so they aren't available. Knowledge bases and the built-in MCP server work as in connected deployments.
- **Sign-in and roles**: Users sign in with the Microsoft Entra application registration that you create on the disconnected stamp. Azure roles are checked first at the extension scope. Use **Reader** for read-only access and **Contributor** for changes and chat. App roles remain a fallback for most management and classic APIs, but not for Responses or Conversations. Role assignments can take time to take effect. See [Assign Azure roles](deploy-disconnected.md#assign-azure-roles).

For Agent Service concepts, see [Agent Service overview](../agentic-overview.md).

## Architecture summary

The **Agents and Tools with Foundry Local** extension for disconnected operations uses the same Azure Arc-enabled Kubernetes cluster and extension-based deployment model as connected deployments. The key difference is that you import the extension's container images, model artifacts, and dependencies into the disconnected environment through a locally installed expansion pack, rather than pulling them from internet-connected registries.

At a high level:

- The **Agents and Tools with Foundry Local extension** (`microsoft.arc.rag`) deploys the same core Agent Service and chat experience as in connected deployments. The `indexedSourcesEnabled` setting controls whether local ingestion and indexed retrieval are also deployed.
- **Expansion packs** are imported into Azure Local Disconnected Operations and populate the local `edgeartifacts` container registry with the extension's images and model artifacts.
- The extension pulls its images and model artifacts from `edgeartifacts` instead of from the Foundry cloud catalog or public registries.
- A **Foundry Local on Azure Local** endpoint (recommended) or a reachable BYOM endpoint provides the language model on or reachable from the disconnected cluster.
- Applications and end users reach the chat and agent endpoints through the cluster's ingress, secured with a Microsoft Entra app registration created on the disconnected stamp.

For the connected architecture, see [Deployment overview for Agents and Tools with Foundry Local](../deploy-overview.md#agents-and-tools-with-foundry-local-architecture-and-components).

## Next step

> [!div class="nextstepaction"]
> [Download and import the Agents and Tools with Foundry Local expansion pack](prepare-disconnected.md)

## Related content

- [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md)
- [What is Agents and Tools with Foundry Local?](../overview.md)
- [Deployment overview for Agents and Tools with Foundry Local](../deploy-overview.md)
- [What you need for Agents and Tools with Foundry Local](../requirements.md)
