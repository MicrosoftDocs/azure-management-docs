---
title: Edge RAG Deployment Overview
description: Learn about deploying Edge RAG with Azure Arc, including prerequisites, configuration options, and steps for secure, scalable AI at the edge.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 08/27/2025
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As an IT administrator or cloud architect, I want to learn about deploying and configuring Edge RAG with Azure Arc so that I can enable a secure, scalable AI-powered chat solution using my organization's data at the edge.
---

# Deployment overview for Edge RAG Preview enabled by Azure Arc

Edge RAG brings generative AI capabilities to your edge and hybrid environments, letting you deploy a secure, scalable, and intelligent chat solution that uses your own data. This article gives an overview of the deployment process, key components, and workflow to deploy Edge RAG with Azure Arc.

To try Edge RAG without the need for local hardware, see [Quickstart: Install Edge RAG Preview enabled by Azure Arc](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Edge RAG architecture and components

The following diagram shows the key components and architecture you have after you prepare your environment and deploy Edge RAG. It includes Azure resources, on-premises infrastructure, and integration points.

<!-- Art Library Source# ConceptArt-0-000-95 -->
:::image type="content" source="media/deploy-overview/deployment-components.svg" alt-text="Diagram that shows the Edge RAG deployment architecture, including Azure resources, on-premises infrastructure, and integration points." border="false":::

The resources and components in the diagram form the core infrastructure for Edge RAG:

- **Azure resources:**
  Microsoft Entra ID and Azure Arc provide identity, access, and hybrid management.

- **On-premises infrastructure:**
  These components provide the compute, networking, load balancing, and storage that you need to run the Edge RAG extension and to access Edge RAG locally:

  - Azure Local
  - Azure Kubernetes (AKS) cluster on Azure Local
  - MetalLB
  - AKS node pool
  - Network file share (NFS) server
  - Driver machine (local management host)

  Use a driver machine (local management host) to simplify management of the Azure Arc-enabled Kubernetes cluster on Azure Local. For more information, see  [Prepare AKS cluster on Azure Local for Edge RAG](prepare-aks-cluster.md) and [Configure machine to manage Azure Arc-Enabled Kubernetes cluster](configure-driver-machine.md).

This setup lets you run a secure, scalable, AI-powered chat solution that uses your own data at the edge.

## Key configuration options

When you deploy Edge RAG, you can set several configuration options to tailor the solution to your environment and requirements.

- **Language model selection:** Choose a Microsoft-provided model (like Phi-3.5 or Mistral-7B), or use your own model (BYOM) endpoint.
- **SSL and domain configuration:** Set up a Transport Layer Security (TLS) termination certificate and domain so you can securely access the chat endpoint.
- **Access and authentication:** Set up the Entra app ID and assign roles to users and groups.
- **Data source configuration:** Make sure your NFS server is reachable and has the required files in supported formats.

## Deployment process for Edge RAG

The deployment process for Edge RAG consists of the following high-level steps:

| High-level step  | Description |
|-----------------|-----------------------------------------------------------|
| 1. Prepare the environment               | Set up the required Azure and on-premises infrastructure, configure your AKS Arc cluster and node pools, establish networking and storage, and set up authentication and user roles. Review the [requirements](requirements.md) and complete the [prerequisites checklist](complete-prerequisites.md). <br><br>As part of the prerequisites, if you plan to use your own language model instead of one of the models provided by Microsoft,  [create an endpoint to use for Edge RAG](prepare-model-endpoint.md). <br><br>If you're using [Microsoft Azure Government](/azure/azure-government/documentation-government-welcome), see [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc) for the deployment variations with Edge RAG. |
| 2. Deploy the Edge RAG extension         | Use the Azure portal or CLI to install the extension on your AKS Arc cluster. Select and configure your preferred language model, set up security and access parameters, and connect the extension to your Microsoft Entra ID for authentication. See [Deploy the Edge RAG extension](deploy.md). <br><br>As part of the deployment step, if you configured Edge RAG to use your own language model instead of a Microsoft provided model, [configure "BYOM" endpoint authentication for Edge RAG](configure-endpoint-authentication.md).|
| 3. Validate the deployment               | After you deploy the extension, check that the Edge RAG extension is installed and running on your cluster and that you have connectivity to the chat endpoint.                                                                |
| 4. Configure chat solution               | Before making the chat solution available to your organization, configure its data source, user experience, and test the setup to make sure it meets your requirements. See [Add a data source for Edge RAG](add-data-source.md), [Configure the chat solution](build-chat-solution-overview.md), and [Test the chat solution](test-end-user-app.md). |
| 5. Monitor and evaluate your deployment  | After deploying Edge RAG, monitor system health, track performance, and evaluate the quality of your AI solution. Use built-in metrics and evaluation tools to observe, assess, and optimize your deployment. See [Evaluate the Edge RAG system](evaluate-solution.md) and [Monitor Edge RAG](observability.md). |

## Related content

- [Requirements for Edge RAG](requirements.md)
- [Deployment Prerequisites Checklist](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
- [Add a data source for Edge RAG](add-data-source.md)
- [Configure the chat solution](build-chat-solution-overview.md)