---
title: Edge RAG Deployment Overview
description: Learn about deploying Edge RAG with Azure Arc, including prerequisites, configuration options, and steps for secure, scalable AI at the edge.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 08/14/2025
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As an IT administrator or cloud architect, I want to learn about deploying and configuring Edge RAG with Azure Arc so that I can enable a secure, scalable AI-powered chat solution using my organization's data at the edge.
---

# Deployment overview for Edge RAG preview enabled by Azure Arc

Edge RAG brings generative AI capabilities to your edge and hybrid environments, letting you deploy a secure, scalable, and intelligent chat solution that uses your own data. This article gives an overview of the deployment process, key components, and workflow to deploy Edge RAG with Azure Arc.

## Edge RAG architecture and components

The following diagram shows the key components and architecture you get after you prepare your environment and deploy Edge RAG. It includes Azure resources, on-premises infrastructure, and integration points.

:::image type="content" source="media/deploy-overview/deployment-components.png" alt-text="Diagram that shows the Edge RAG deployment architecture, including Azure resources, on-premises infrastructure, and integration points.":::

The resources and components in the diagram form the core infrastructure for Edge RAG:

- **Azure resources:**
  Microsoft Entra ID and Azure Arc provide identity, access, and hybrid management.

- **On-premises infrastructure:**
  These components provide the compute, networking, load balancing, and storage that you need to run the Edge RAG extension:

  - Azure Local
  - Azure Kubernetes cluster on Azure Local
  - MetalLB
  - Network file share (NFS) server

This setup lets you run a secure, scalable, AI-powered chat solution that uses your own data at the edge.

## Key configuration options

When you deploy Edge RAG, you can set several configuration options to tailor the solution to your environment and requirements.

- **Language model selection:** Choose a Microsoft-provided model (like Phi-3.5 or Mistral-7B), or use your own model (BYOM) endpoint.
- **SSL and domain configuration:** Set up a Transport Layer Security (TLS) termination certificate and domain so you can securely access the chat endpoint.
- **Access and authentication:** Set up the Entra app ID and assign roles to users and groups.
- **Data source configuration:** Make sure your NFS server is reachable and has the required files in supported formats.

## Deployment process for Edge RAG

The deployment process for Edge RAG consists of the following high-level steps:

### 1. Prepare the environment

To prepare your environment, set up the required Azure and on-premises infrastructure, configure your AKS Arc cluster and node pools, establish networking and storage, and set up authentication and user roles. To get started, review the [requirements for Edge RAG](requirements.md) and complete the [prerequisites checklist](complete-prerequisites.md).

### 3. Deploy the Edge RAG extension

To deploy, use the Azure portal or CLI to install the extension on your AKS Arc cluster. Select and configure your preferred language model, set up security and access parameters, and connect the extension to your Microsoft Entra ID for authentication. This step enables the core AI capabilities for your Edge RAG solution. For more information, see [Deploy the Edge RAG extension](deploy.md).

### 4. Validate the deployment

After you deploy the extension, check that the Edge RAG extension is installed and running on your cluster and that you have connectivity to the chat endpoint.

### 4. Configure chat solution

Before making the chat solution available to your organization, configure its data source, user experience, and test the setup to make sure it meets your requirements.

- [Add a data source for Edge RAG](add-data-source.md): Connect your NFS server and ingest documents and images.
- [Configure the chat solution](build-chat-solution-overview.md): Set up chat parameters, prompts, and user experience.
- [Test the chat solution](test-end-user-app.md): Use the out-of-the-box chat application for end user testing or for end users to get started quickly.

### 5. Monitor and evaluate your deployment

After deploying Edge RAG, it’s important to monitor system health, track performance, and evaluate the quality of your AI solution. Monitoring helps you make sure the deployment is running smoothly, while evaluation tools let you measure how well your models and data sources are meeting your organization’s needs. Use the following resources to observe, assess, and optimize your Edge RAG deployment.

- [Evaluate the Edge RAG system](evaluate-solution.md): Evaluate the system, models, and datasets within Edge RAG.
- [Monitor Edge RAG](observability.md): Review the monitoring and observability setup to make sure metrics and logs are available. Then use built-in metrics and evaluation tools to monitor performance and quality.

## Related content

- [Requirements for Edge RAG](requirements.md)
- [Deployment Prerequisites Checklist](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
- [Add a data source for Edge RAG](add-data-source.md)
- [Configure the chat solution](build-chat-solution-overview.md)