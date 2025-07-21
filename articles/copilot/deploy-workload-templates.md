---
title: Discover and deploy workload templates from Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure can provide workload templates for your scenario.
ms.date: 04/08/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom: build-2024, linux-related-content, ignite-2024
ms.author: jenhayes
author: JnHs
# Customer intent: "As a cloud developer, I want to discover and deploy workload templates using an AI assistant, so that I can quickly implement solutions following industry best practices without extensive manual setup."
---

# Discover and deploy workload templates from Microsoft Copilot in Azure

Microsoft Copilot in Azure can help you find workload deployment templates that suit your needs by quickly searching through extensive repositories to identify the best matches. It provides recommendations based on the latest best practices and industry standards, helping you effectively deploy your workloads.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to discover and deploy workload templates. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Deploy a computer vision model to scan aerial imagery for changes to buildings and land releases"
- "Deploy nodejs web app with Postgres DB"
- "I want to build a RAG using an LLM on Azure"
- "Do you have a template to help me deploy Azure AI Services?"
- "Move a .NET application I have to Azure"
- "Deploy python function app running on docker"
- "I want to take advantage of OpenAI or other advanced models in a protected environment that is subject to Enterprise Data Protection (EDP) policies so I can use company data in the process."
- "I want to create assistants API in Azure OpenAI"
- "Any workload templates to help with RAG in Python?"
- "I want a template suggestion to create an Ubuntu Virtual Machine and attach an Azure Data Disk."
- "Template suggestion to deploy an AI model on AKS with the AI toolchain operator"
- "I want a template to deploy a Postgres vector database"

## Examples

To create an e-commerce website using Django framework, you can say **"I want to create an e-commerce website using Django framework."** Copilot in Azure shows a recommendation with a link to the GitHub repository that you can use to run the Azure Development CLI (`azd`) template.

> [!TIP]
> For information on using `azd` templates, see [What is the Azure Developer CLI?](/azure/developer/azure-developer-cli/overview?tabs=windows)

:::image type="content" source="media/deploy-workload-templates/example-workload-template.png" alt-text="Screenshot of example of a template recommendation from Microsoft Copilot in Azure.":::

:::image type="content" source="media/deploy-workload-templates/example-github-repository.png" lightbox="media/deploy-workload-templates/example-github-repository.png" alt-text="Screenshot of example of a GitHub repository for the workload suggested by Microsoft Copilot in Azure. ":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn how to [deploy virtual machines effectively using Microsoft Copilot in Azure](deploy-vms-effectively.md).
