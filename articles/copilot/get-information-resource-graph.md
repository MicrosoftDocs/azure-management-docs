---
title: Get resource information using Azure Copilot
description: Learn about scenarios where Azure Copilot can help with Azure Resource Graph.
ms.date: 11/18/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As a cloud administrator, I want to use natural language prompts to generate Azure Resource Graph queries, so that I can efficiently access and manage my Azure resources without needing deep familiarity with query languages.
---

# Get resource information using Azure Copilot

You can ask Azure Copilot questions about your Azure resources and cloud environment. Using the combined power of large language models (LLMs) and [Azure Resource Graph](/azure/governance/resource-graph/overview), Azure Copilot helps you author Azure Resource Graph queries. You provide input using natural language from anywhere in the Azure portal, and Azure Copilot returns a working query that you can use with Azure Resource Graph. Azure Resource Graph also acts as an underpinning mechanism for other scenarios that require real-time access to your resource inventory.

Azure Resource Graph's query language is based on the [Kusto Query Language (KQL)](/azure/data-explorer/kusto/query/) used by Azure Data Explorer. However, you don't need to be familiar with KQL in order to use Azure Copilot to retrieve information about your Azure resources and environment. Experienced query authors can also use Azure Copilot to help streamline their query generation process. Once Azure Copilot generates your query, you can run it in Azure Resource Graph, or copy the text to modify or save the query.

While a high level of accuracy is typical, we strongly advise you to review the generated queries to ensure they meet your expectations.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

>[!Note]
>If you prefer to perform these prompts in your development environment, you can do so using [GitHub Copilot for Azure (Preview)](/azure/developer/github-copilot-azure/introduction), an extension for Visual Studio Code. Specifically, you can [write prompts to get resource information](/azure/developer/github-copilot-azure/learn-examples) and more.

## Sample prompts

Here are a few examples of the kinds of prompts you can use to generate Azure Resource Graph queries. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries.

- "Show me all resources that are noncompliant"
- "List all virtual machines lacking enabled replication resources"
- "List all the updates applied to my Linux virtual machines"
- "List all storage accounts that are accessible from the internet"
- "List all virtual machines that are not running now"
- "Write a query that finds all changes for last 7 days."
- "Help me write an ARG query that looks up all virtual machines scale sets, sorted by creation date descending"
- "What are the public IPs of my VMs?"
- "Show me all my storage accounts in East US?"
- "List all my Resource Groups and its subscription."
- "Write a query that finds all resources that were created yesterday."

## Examples

You can ask Azure Copilot to write queries with prompts like "**Write a query to list my virtual machines with their public interface and public IP.**"

:::image type="content" source="media/get-information-resource-graph/azure-resource-graph-explorer-list-vms.png" alt-text="Screenshot of Azure Copilot responding to a request to list VMs.":::

If the generated query isn't exactly what you want, you can ask Azure Copilot to make changes. In this example, the first prompt is "**Write a KQL query to list my VMs by OS.**" After the query is shown, the additional prompt "**Sorted alphabetically**" results in a revised query that lists the OS alphabetically by name.

:::image type="content" source="media/get-information-resource-graph/azure-resource-graph-query-refine.png" alt-text="Screenshot of Azure Copilot generating and then revising a query to list VMs by OS.":::

You can view the generated query in Azure Resource Graph Explorer by selecting **Run**. For example, you can ask "**What resources were created in the last 24 hours?**" After Azure Copilot generates the query, select **Run** to see the query and results in Azure Resource Graph Explorer.

:::image type="content" source="media/get-information-resource-graph/azure-resource-graph-last-24-hours.png" lightbox="media/get-information-resource-graph/azure-resource-graph-last-24-hours.png" alt-text="Screenshot showing Azure Copilot generating a query and showing results in Azure Resource Graph Explorer.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure Resource Graph](/azure/governance/resource-graph/overview).
