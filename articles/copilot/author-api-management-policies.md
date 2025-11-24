---
title: Author API Management policies using Azure Copilot
description: Learn about how Azure Copilot can generate Azure API Management policies based on your requirements.
ms.date: 11/24/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As a developer, I want to use AI-assisted tools to generate and explain API Management policies, so that I can efficiently create complex policies without needing to understand all the underlying code.
---

# Author API Management policies using Azure Copilot 

Azure Copilot can author [Azure API Management policies](/azure/api-management/api-management-howto-policies) based on your requirements. By using Azure Copilot, you can create policies quickly, even if you're not sure what code you need. This can be especially helpful when creating complex policies with many requirements.

To get help authoring API Management policies, start from the **Design** tab of an API you previously imported to your API Management instance. Be sure to use the [code editor view](/azure/api-management/set-edit-policies?tabs=editor#configure-policy-in-the-portal). Ask Azure Copilot to generate policy definitions for you, then copy the results right into the editor, making any desired changes. You can also ask questions to understand the different options or change the provided policy.

When you're working with API Management policies, you can also select a portion of the policy, right-click, and then select **Explain**. This will open Azure Copilot and paste your selection with a prompt to explain how that part of the policy works.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to get help authoring API Management policies. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of policies.

- "Generate a policy to configure rate limiting with 5 requests per second"
- "Generate a policy to remove a 'X-AspNet-Version' header from the response"
- "Explain (selected policy or element) to me"

## Examples

When creating an API Management policy, you can say "**Can you show me how to write a policy expression to filter API responses based on user roles in Azure API Management?**" Azure Copilot generates a policy and explains how it works.

:::image type="content" source="media/author-api-management-policies/api-management-filter-responses.png" alt-text="Screenshot of Azure Copilot generating a policy to filter API responses.":::

When you have questions about policy elements, you can get more information by selecting a section of the policy, right-clicking, and selecting **Explain**.

:::image type="content" source="media/author-api-management-policies/api-management-policy-explain.png" lightbox="media/author-api-management-policies/api-management-policy-explain.png" alt-text="Screenshot of right-clicking a section of an API Management policy to get an explanation from Azure Copilot .":::

Azure Copilot explains how the code works, breaking down each specific section and providing links to learn more.

:::image type="content" source="media/author-api-management-policies/api-management-policy-explanation.png" alt-text="Screenshot of Azure Copilot providing information about a specific API Management policy.":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Azure API Management](/azure/api-management/api-management-key-concepts).
