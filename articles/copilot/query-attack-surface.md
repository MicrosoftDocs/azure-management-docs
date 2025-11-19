---
title: Query your attack surface with Defender EASM using Azure Copilot
description: Learn how Azure Copilot can help query Attack Surface Insights from Defender EASM.
ms.date: 11/18/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As a security analyst, I want to use natural language queries with my organization's attack surface data, so that I can easily extract contextual insights and improve my understanding of potential vulnerabilities without needing advanced querying skills.
---

# Query your attack surface with Defender EASM using Azure Copilot

[Microsoft Defender External Attack Surface Management (Defender EASM)](/azure/external-attack-surface-management/overview) scans inventory assets and collects robust contextual metadata that powers Attack Surface Insights. These insights help an organization understand what their attack surface looks like, where the risk resides, and what assets they need to focus on.

> [!IMPORTANT]
> Use of Azure Copilot to query Defender EASM is included with Copilot for Security and requires [security compute units (SCUs)](/copilot/security/get-started-security-copilot#security-compute-units).  You can provision SCUs and increase or decrease them at any time. For more information on SCUs, see [Get started with Microsoft Copilot](/copilot/security/get-started-security-copilot) and [Manage usage of security compute units](/copilot/security/manage-usage).
>
> To use Azure Copilot to query Defender EASM, you or your admin team must be a member of the appropriate role in Copilot for Security and must have access to a Defender EASM resource. For information on supported roles, see [Understand authentication in Microsoft Copilot for Security](/copilot/security/authentication).

With Azure Copilot, you can use natural language to ask questions and better understand your organization's attack surface. Through Defender EASM's extensive querying capabilities, you can extract asset metadata and key asset information, even if you don't have an advanced Defender EASM querying skillset.

When you ask Azure Copilot about your attack surface, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. If the context isn't clear, you'll be prompted to specify which Defender EASM resource to use.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Sample prompts

Here are a few examples of the kinds of prompts you can use to query attack surface data collected by Defender EASM. Modify these prompts based on your real-life scenarios, or try additional prompts to get advice on specific areas.

- "Tell me about Defender EASM high priority attack surface insights."
- "What are my externally facing assets?"
- "Find all the page and host assets in my inventory with the IP address (address)"
- "Show me all assets that require investigation."
- "Do I have any domains that are expiring within 30 days?"
- "What assets are using jQuery version 3.1.0?"
- "Get the hosts with port X open in my attack surface?"
- "Which of my assets have a registrant email of `name@example.com`?"
- "Which of my assets have services containing 'Azure' and vulnerabilities on them?"

## Example

You can use a natural language query to better understand your attack surface. In this example, the query is "**find all the page and host assets in my inventory with an ip address that is (list of IP addresses)**". Azure Copilot queries your Defender EASM inventory and provides details about the assets matching your criteria. You can then follow up with additional questions as needed.

:::image type="content" source="media/query-attack-surface/query-assets-ip-addresses.png" alt-text="Screenshot showing Azure Copilot providing results for a natural language attack surface query." lightbox="media/query-attack-surface/query-assets-ip-addresses.png":::

## Next steps

- Explore [capabilities](capabilities.md) of Azure Copilot.
- Learn more about [Defender EASM](/azure/external-attack-surface-management/overview).
