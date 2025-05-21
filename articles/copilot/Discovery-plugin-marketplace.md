---
title: Discover Azure Marketplace solutions 
description: Learn how Microsoft Copilot in Azure helps you find the right Azure Marketplace solutions using natural language queries.
ms.date: 05/21/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom: ignite-2025, ignite-2025-copilotinAzure, devx-track-marketplace, build-2025
ms.author: aherskovitz
author: aherskovitz
---

# Discover Azure Marketplace solutions using Microsoft Copilot

Microsoft Copilot in Azure enables you to find the right Azure Marketplace solutions using natural language queries. Instead of relying on filters or knowing specific product names, you can describe your needs in your own words, and Copilot will provide tailored suggestions.

When you ask a question in Copilot, the Discovery Plugin:

- Understands your intent (e.g., "secure APIs" → API Gateway or WAF solutions).
- Enriches your query with context and inferred filters.
- Searches the Azure Marketplace using intelligent APIs.
- Returns relevant offers, services, or follow-ups.

All this happens in real time, within your Azure Copilot chat.

## Sample prompts

Here are a few examples of the kinds of prompts you can use to discover Azure Marketplace solutions. Modify these prompts based on your real-life scenarios, or try additional prompts to explore different solutions.

- "Show me database offers for scalable cloud storage."
- "List AI-powered virtual machine configurations for data analysis."
- "I am looking for a service to analyze customer sentiment from voice recordings."
- "Show me virtual machine options optimized for machine learning workloads."
- "Can you recommend a solution for managing digital identities using blockchain?"
- "I am looking for a tool to transcribe audio files into text."
- "What are the best blockchain solutions for supply chain management?"
- "I need a service for converting speech to multiple languages in real-time."
- "Can you suggest blockchain platforms for creating smart contracts?"
- "What blockchain solutions are available for secure financial transactions?"
- "List virtual machines with GPU support for AI model training."
- "I need a tool for real-time voice-to-text transcription for meetings."
- "Can you recommend blockchain services for tokenizing digital assets?"
- "Show me virtual machine offers with pre-installed development tools."
- "I am looking for a speech-to-text API for integrating into my application."

## Response Structure

When you use Microsoft Copilot in Azure to search for solutions, the response typically includes the following:

1. **Top 3 Relevant Products**:  
   Copilot provides a list of the most relevant products based on your query. Each product includes:
   - **Product Name**: The name of the product or service.
   - **Description**: A brief overview of the product's features and use cases.
   - **Link**: A direct link to the product's page in the Azure Marketplace for more details.   
   
   For example, the prompt "**I am looking for speech to text recognition tool.**" provides tools like Azure Speech Service for real-time speech-to-text conversion.

:::image type="content" source="media/discover-marketplace/speech-to-text-tools.png" alt-text="Screenshot of Microsoft Copilot in Azure providing speech-to-text recognition tools.":::


2. **Explore Marketplace Button**:  
   In the response, you may see an **"Explore Marketplace"** button. This button allows you to:

   - Navigate directly to the Azure Marketplace.
   - View detailed information about the suggested products.
   - Compare options and proceed with deployment or purchase.

   The **"Explore Marketplace"** button ensures a seamless transition from discovering solutions in Copilot to exploring and implementing them in the Azure Marketplace.

   :::image type="content" source="media/discover-marketplace/explore-marketplace-button.png" alt-text="Screenshot of the Explore Marketplace button in Microsoft Copilot in Azure.":::

3. **Additional Suggestions**:  
   Copilot may also provide follow-up questions or related prompts to refine your search or explore other solutions.

   :::image type="content" source="media/discover-marketplace/additional-suggestions.png" alt-text="Screenshot of Microsoft Copilot in Azure providing additional suggestions.":::

By using these features, you can quickly find and act on the solutions that best meet your needs.

## Examples

In this example, the prompt "**I am looking for a blockchain solution for managing digital assets. Can you help me find one?**" provides curated blockchain solutions for managing digital assets.

:::image type="content" source="media/discover-marketplace/blockchain-solutions.png" alt-text="Screenshot of Microsoft Copilot in Azure providing blockchain solutions.":::

In this example, the prompt "**Show me virtual machine offers.**" provides a list of virtual machine options, including Azure Virtual Machines and Windows Virtual Desktop.

:::image type="content" source="media/discover-marketplace/virtual-machine-offers.png" alt-text="Screenshot of Microsoft Copilot in Azure providing virtual machine offers.":::


## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [Azure Marketplace](https://azure.microsoft.com/en-us/partners/marketplace).