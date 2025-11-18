---
title: Bring your own storage for conversation history in Azure Copilot
description: Azure Copilot gives tenant administrators the option to store conversation history in their own Cosmos DB instance.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: concept-article

# Customer intent: "As an Azure tenant administrator, I want to retain Azure Copilot conversation history within my organization, so that we can review and manage Azure Copilot conversation data."
---

# Bring your own storage for conversation history in Azure Copilot

By default, Azure Copilot [conversation data is managed by Microsoft](responsible-ai-faq.md). However, organizations can choose to store Azure Copilot conversation history in their own [Azure Cosmos DB](/azure/cosmos-db/introduction) instance.

The **Bring your own storage** option lets tenant administrators retain all Azure Copilot conversations in a Cosmos DB instance selected and managed by the tenant. Organizations maintain an audit trail of all Azure Copilot conversations across all tenant users, including the user-submitted prompts and all Azure Copilot responses. By retaining conversation history within your own environment, you have more flexibility to meet data retention, compliance, and governance requirements.

## Considerations

Before enabling **Bring your own storage**, it's important to be aware of how your selections impact previous conversation history.

When you set up conversation history storage in a Cosmos DB instance managed by your organization, you no longer have access to previous conversations that were stored by Microsoft. Users in your tenant are no longer able to access any of their Azure Copilot conversations that occurred before you enabled **Bring your own storage**.

Similarly, if you enabled **Bring your own storage**, then later update your conversation storage to use new Cosmos DB instance, users in your tenant lose access to previous conversations stored in the original Cosmos DB instance. However, if a user has access to the previous Cosmos DB instance, they can access that data according to their permissions for that resource and your organization's data retention policies.

If you disable **Bring your own storage**, Microsoft resumes conversation storage for your organization, and users can only access their new conversations that are stored by Microsoft. Users in your tenant lose access to any conversations previously stored in your organization's Cosmos DB instance, unless they have permissions to access the database directly.

## Permissions

- You must be a global administrator to configure conversation history storage for your Azure tenant.
- After you enable **Bring your own storage**, you'll be assigned the [DocumentDB Account Contributor](/azure/role-based-access-control/built-in-roles/databases#documentdb-account-contributor) role so that Azure Copilot can configure the necessary containers on your behalf.
- A system-assigned managed identity will be created with the [Cosmos DB Built-in Data Contributor](/azure/cosmos-db/table/reference-data-plane-security#built-in-roles) role assigned, in order to enable secure read/write access for conversation storage. If you later disable **Bring your own storage**, this managed identity will be deleted.

## Enable conversation history storage

To enable conversation history storage, follow these steps.

1. In the Azure portal, open the [**Conversation storage** page](https://portal.azure.com/#view/HubsExtension/AssetMenuBlade/~/conversationStorage/assetName/CopilotSettingsAsset/extensionName/Microsoft_Azure_Copilot) in *Azure Copilot admin center*.
1. Check the **Bring your own storage** box.
1. Select the subscription and Cosmos DB instance where you want to store conversation history.
1. Check the box confirming that you allow Azure Copilot to configure containers and assign the appropriate permissions in your Cosmos DB instance.
1. Select **Apply**.

Once you enable **Bring your own storage**, you see a confirmation message in Azure Copilot indicating that conversations are now stored in your organization's Cosmos DB instance. Likewise, if you disable **Bring your own storage**, you see a warning that their conversations are no longer stored in your organization's Cosmos DB instance, and that users in your tenant no longer have access to conversation history.

If you change from one Cosmos DB instance to another, a confirmation message indicates that future conversations will be stored in the new Cosmos DB instance.

## Data retention

Azure Copilot conversation history stored in your Cosmos DB instance is retained according to your organization's data retention policies. Be sure to review and configure these policies to meet your compliance and governance requirements.
