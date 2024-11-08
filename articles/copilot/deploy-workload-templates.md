---
title: Discover and deploy workload templates from Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure (preview) can provide workload templates for your scenario.
ms.date: 11/07/2024
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom: build-2024, linux-related-content
ms.author: jenhayes
author: JnHs
---

# Discover and deploy workload templates from Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) can help you find workload deployment templates that suit your needs by quickly searching through extensive repositories to identify the best matches. It provides recommendations based on the latest best practices and industry standards, helping you effectively deploy your workloads.

For a subset of these workload templates, Copilot in Azure provides an enhanced deployment experience to help you quickly set up your workload in Azure.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

[!INCLUDE [preview-note](includes/preview-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to discover and deploy workload templates. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Deploy a computer vision model to scan aerial imagery for changes to buildings and land releases"
- "Deploy nodejs web app with Postgres DB"
- "I want to build a RAG using an LLM on Azure"
- "Do you have a template to help me deploy Azure AI Services?"
- "Move a .NET application I have to Azure"
- "Deploy python function app running on docker"
- "I want to take advantage of open AI or other advanced models in a protected environment that is subject to Enterprise Data Protection (EDP) policies so I can use company data in the process."
- "I want to create assistants API in Azure OpenAI"
- "Any workload templates to help with RAG in Python?"

## Examples

To create an ecommerce website using Django framework, you can say **"I want to create an ecommerce website using Django framework."** Copilot in Azure shows a recommendation with a link to the GitHub repository that you can use.

:::image type="content" source="media/deploy-workload-templates/example-workload-template.png" alt-text="Example of a template recommendation from Microsoft Copilot in Azure (preview).":::

:::image type="content" source="media/deploy-workload-templates/example-github-repository.png" lightbox="media/deploy-workload-templates/example-github-repository.png" alt-text="Example of a GitHub repository for the workload suggested by Microsoft Copilot in Azure. ":::

For some workload templates, Copilot in Azure provides an enhanced deployment experience to help you quickly set up your workload in Azure. You have two options: quickly deploy your workload by running all of the steps at once, or choose to learn with step-by-step guidance. For example, try the following prompts:

- **"I want a template suggestion to deploy a Postgres vector database"**:

  :::image type="content" source="media/deploy-workload-templates/deploy-postgres-vector-database.png" lightbox="media/deploy-workload-templates/deploy-postgres-vector-database.png"  alt-text="Screenshot of Microsoft Copilot in Azure providing an interactive deployment experience for a Postgres vector database.":::

- **"Template suggestion to deploy an AI model on AKS with the AI toolchain operator"**

  :::image type="content" source="media/deploy-workload-templates/deploy-ai-model-aks.png" lightbox="media/deploy-workload-templates/deploy-ai-model-aks.png" alt-text="Screenshot of Microsoft Copilot in Azure providing an interactive deployment experience for an AI model on AKS.":::

- **"I want a template suggestion to create an Ubuntu Virtual Machine and attach an Azure Data Disk."**

  :::image type="content" source="media/deploy-workload-templates/deploy-ubuntu-azure-data-disk.png" lightbox="media/deploy-workload-templates/deploy-ubuntu-azure-data-disk.png"alt-text="Screenshot of Microsoft Copilot in Azure providing an interactive deployment experience for an Ubuntu VM with an Azure Data Disk.":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn how to [deploy virtual machines effectively using Microsoft Copilot in Azure](deploy-vms-effectively.md).
