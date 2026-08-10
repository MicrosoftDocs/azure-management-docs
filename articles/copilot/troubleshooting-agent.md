---
title: Azure Copilot Troubleshooting Agent (preview)
description: The Azure Copilot Troubleshooting Agent helps you diagnose problems and find solutions in your Azure environment.
ms.date: 06/22/2026
ms.service: azure-copilot
ms.topic: concept-article

# Customer intent: "As an Azure Copilot user, I want to understand how to use the Troubleshooting Agent, so that I can resolve problems in my Azure environment."
---

# Azure Copilot Troubleshooting Agent (preview)

The Azure Copilot Troubleshooting Agent helps you diagnose issues, find solutions, and resolve problems in your Azure environment. When possible, the Troubleshooting Agent analyzes your specific environment to run root cause diagnostics. Once it identifies the root cause, the Troubleshooting Agent determines the appropriate mitigation steps. It provides tailored solutions and step-by-step instructions. In many cases, the Troubleshooting Agent even offers a one-click fix to resolve the issue for you.

> [!IMPORTANT]
> The Azure Copilot Troubleshooting Agent is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

To use the Troubleshooting Agent, select **Troubleshooting** from the **New chat** options in Azure Copilot. If the Troubleshooting Agent can't resolve your issue, it can create a support request for you, gathering all the necessary details to help Microsoft Support assist you more effectively. You can review and confirm the details before the request is submitted.

> [!NOTE]
> Administrators can [enable or disable access to the Azure Copilot Troubleshooting Agent](manage-access.md#manage-user-access-to-azure-copilot-and-agents) and other [Azure Copilot agents](agents.md). If you don't see the Troubleshooting Agent as an option when starting a new chat, check with your administrator.

## Supported resource types

The Troubleshooting Agent works with all Azure resource types. It's often particularly useful for resolving issues with the following resource types:

- Azure Cosmos DB
- Azure Virtual Machines
- Azure Kubernetes Service (AKS)

## Troubleshooting sample prompts

Here are a few examples of the kinds of prompts you can use with the Troubleshooting Agent. Modify these prompts based on your real-life scenarios, or try additional prompts to create different kinds of queries. If you're not already working in the context of a resource, you might need to provide the specific resource that you want to troubleshoot.

- "Help me troubleshoot why my Cosmos DB Cassandra API is failing."
- "I’m trying to connect to Azure Cosmos DB Cassandra API from my local development machine, but I keep getting a timeout. What should I do?"
- "Help me investigate why my VM is unhealthy."
- "I can't connect to my VM, can you help me troubleshoot?"
- "Investigate the health of my pods."
- "Investigate networking issues causing pod connectivity failures."
- "Identify reasons for high CPU or memory usage in my AKS cluster."
- "Create a support request."
- "Open a support ticket for my problem"

## Current considerations and limitations

Keep in mind the following considerations and limitations when working with the Azure Copilot Troubleshooting Agent.

- Automatic mitigation of common issues isn't available for all issues or resource types. For example, one-click fixes aren't currently available to resolve issues with AKS. In these cases, the Troubleshooting Agent provides detailed instructions to help you resolve the issue, or it can create a support request for further investigation.
- Troubleshooting capabilities are based on currently available diagnostic data and predefined checks.
