---
title: Disconnected Operations for Agentic Retrieval in Foundry Local Overview
description: "Learn how Agentic Retrieval in Foundry Local deployments differ in disconnected operations on Azure Local, including expansion packs, authentication, and architecture."
author: cwatson-cat
ms.author: cwatson
ms.topic: overview #Don't change
ms.date: 07/23/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to understand how Agentic Retrieval in Foundry Local differs in a disconnected Azure Local environment so that I can plan a secure, offline deployment.
---

# Disconnected operations for Agentic Retrieval in Foundry Local

You can deploy Agentic Retrieval in Foundry Local, part of [Agents and Tools with Foundry Local](../overview.md), on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview) by using a deployment model that largely matches connected scenarios. However, several key differences exist when internet connectivity isn't available.

This article explains how disconnected deployments of Agentic Retrieval in Foundry Local differ from connected deployments, so you can plan a secure, offline agentic Retrieval-Augmented Generation (RAG) solution at the edge.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## What changes in disconnected deployments

In disconnected environments, extension availability, model and container image sourcing, certificate and networking dependencies, identity, and access flows differ from connected deployments.

- **Extension availability**: Before you can install the Agentic Retrieval extension (`microsoft.arc.rag`), you must download the Agentic Retrieval expansion pack in a connected environment, transfer it to the disconnected environment, and import it on the Azure Local Disconnected Operations machine. After import, the extension type is registered locally and becomes available for installation. For more information, see [Download and import the Agentic Retrieval expansion pack](prepare-disconnected.md).
- **Container image and model artifact source**: The expansion pack imports the extension's container images into the local `edgeartifacts` container registry and publishes model artifacts to the registry. Deployments pull these assets locally instead of from internet-connected registries.
- **Language model endpoint**: The recommended option is a Foundry Local on Azure Local endpoint running on the same disconnected cluster. You can also use an external Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API and is reachable from the disconnected environment.
- **Authentication**: End-user sign-in doesn't use public Microsoft Entra ID endpoints. Instead, you create the Microsoft Entra application registration on the disconnected stamp by using the `az` CLI and the disconnected Microsoft Graph endpoint, and you register the sign-in redirect URIs for your ingress domain.
- **Authorization**: When you use managed identity authentication to Foundry, you grant the cluster managed identity the built-in **Contributor** role on the Foundry extension's ARM scope. Role propagation on disconnected Azure can take longer than in connected environments.
- **Networking**: The extension requires a policy-enforcing container network interface (CNI) such as Calico, Cilium, or Azure NPM, because Agentic Retrieval uses Kubernetes network policies by default. The ingress load-balancer IP isn't baked into the expansion pack. You supply an IP from your cluster's ingress subnet at install time by using MetalLB.

## Architecture summary

Agentic Retrieval in Foundry Local for disconnected operations uses the same Azure Arc-enabled Kubernetes cluster and extension-based deployment model as connected deployments. The key difference is that the extension's container images, model artifacts, and dependencies are imported into the disconnected environment through a locally installed expansion pack, rather than pulled from internet-connected registries.

At a high level:

- The **Agentic Retrieval extension** (`microsoft.arc.rag`) deploys the same agentic layer, knowledge layer, and chat experience as in connected deployments, based on your chosen deployment mode (`combined`, `agentic`, or `knowledge`).
- **Expansion packs** are imported into Azure Local Disconnected Operations and populate the local `edgeartifacts` container registry with the extension's images and model artifacts.
- The extension pulls its images and model artifacts from `edgeartifacts` instead of from the Foundry cloud catalog or public registries.
- A **Foundry Local on Azure Local** endpoint (recommended) or a reachable BYOM endpoint provides the language model on or reachable from the disconnected cluster.
- Applications and end users reach the chat and agent endpoints through the cluster's ingress, secured with a Microsoft Entra app registration created on the disconnected stamp.

For the connected architecture, see [Deployment overview for Agentic Retrieval in Foundry Local](../deploy-overview.md#agentic-retrieval-architecture-and-components).

## Next step

> [!div class="nextstepaction"]
> [Download and import the Agentic Retrieval expansion pack](prepare-disconnected.md)

## Related content

- [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md)
- [What is Agentic Retrieval in Agents and Tools with Foundry Local?](../overview.md)
- [Deployment overview for Agentic Retrieval in Foundry Local](../deploy-overview.md)
- [What you need for Agentic Retrieval](../requirements.md)
