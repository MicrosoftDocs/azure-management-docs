---
title: Responsible AI FAQ for Azure Copilot
description: Learn how Azure Copilot uses data and what to expect.
ms.date: 11/18/2025
ms.topic: faq
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
hideEdit: true
# Customer intent: As an Azure user, I want to understand how Azure Copilot uses my data and the reliability of its responses, so that I can effectively manage and optimize my Azure environment while ensuring data privacy and security.
---

# Responsible AI FAQ for Azure Copilot

## What is Azure Copilot?

Azure Copilot is an AI companion that enables IT teams to operate, optimize, and troubleshoot applications and infrastructure more efficiently. With Azure Copilot, users can gain new insights into their workloads, unlock untapped Azure functionality, and orchestrate tasks across both cloud and edge. Copilot leverages Large Language Models (LLMs), the Azure control plane, and insights about a user’s Azure and Arc-enabled assets. All of this is carried out within the framework of Azure’s steadfast commitment to safeguarding the customer's data security and privacy. For an overview of how Azure Copilot works and a summary of Copilot capabilities, see [Azure Copilot overview](overview.md).

## What is the current status of Azure Copilot?

Azure Copilot is generally available.

## Are Azure Copilot's results reliable?

Azure Copilot is designed to generate the best possible responses with the context to which it has access. However, like any AI system, Azure Copilot's responses will not always be perfect. All of Azure Copilot's responses should be carefully tested, reviewed, and vetted before making changes to your Azure environment.

## How does Azure Copilot use data from my Azure environment?

Azure Copilot generates responses grounded in your Azure environment. Azure Copilot only has access to resources that you have access to and can only perform actions that you have the permissions to perform, after your confirmation. Azure Copilot will respect all existing access management and protections such as Azure Role-Based Action Control, Privileged Identity Management, Azure Policy, and resource locks.

## What data does Azure Copilot collect?

User-provided prompts and Azure Copilot's responses are not used to further train, retrain, or improve Azure OpenAI Service foundation models that generate responses. User-provided prompts and Azure Copilot's responses are collected and used to improve Microsoft products and services only when users have given explicit consent to include this information within feedback. We collect user engagement data, such as, number of chat sessions and session duration, the skill that's used in a particular session, thumbs up, thumbs down, feedback, etc. This information is retained and used as set forth in the [Microsoft Privacy Statement](https://privacy.microsoft.com/en-us/privacystatement).

## What should I do if I see unexpected or offensive content?

The Azure team has built Azure Copilot guided by our [AI principles](https://www.microsoft.com/ai/principles-and-approach) and [Responsible AI Standard](https://aka.ms/RAIStandardPDF). We have prioritized mitigating exposing customers to offensive content. However, you might still see unexpected results. We're constantly working to improve our technology in preventing harmful content.

If you encounter harmful or inappropriate content in the system, please provide feedback or report a concern by selecting the downvote button on the response.

## How current is the information Azure Copilot provides?

We frequently update Azure Copilot to ensure Azure Copilot provides the latest information to you. In most cases, the information Azure Copilot provides will be up to date. However, there might be some delay between new Azure announcements to the time Azure Copilot is updated.

## Do all Azure services have the same level of integration with Azure Copilot?

No. Some Azure services have richer integration with Azure Copilot. We will continue to increase the number of scenarios and services that Azure Copilot supports. To learn more about some of the current capabilities, see [Azure Copilot capabilities](capabilities.md) and the articles in the **Enhanced scenarios** section.
