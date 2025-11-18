---
title: Manage access to Agents (preview) in Azure Copilot
description: Tenant administrators can control which Azure Copilot preview features are available to users in their Azure tenant.
author: JnHs
ms.author: jenhayes
ms.date: 11/18/2025
ms.service: copilot-for-azure
ms.topic: overview

# Customer intent: "As an Azure Copilot user, I want to manage preview feature availability, so that I have control over which features are available to users in my Azure tenant."
---

# Manage access to Agents (preview) in Azure Copilot

To manage access to [Agents (preview) in Azure Copilot](agents-preview.md), an administrator can request or remove access at the tenant level.

## Request access to Agents (preview)

To enable Agents (preview) in Azure Copilot, an administrator can request access at the tenant level. In many cases, this request is already made on your behalf. Global administrators for a tenant can confirm preview access by following these steps:

1. In the Azure portal, search for **Azure Copilot admin center** and select it.
1. In the service menu, under **Settings**, select **Access management**.
1. In the **Agents (preview)** section, ensure that **Request access to Agents (preview)** is toggled **On**.

After **Request access to Agents (preview)** is selected, you see a notice that approval is pending. Once your request is approved, you see a message that access has been granted. This means that all users in your tenant who have access to Azure Copilot can use the Agents (preview) features. As new agent capabilities become available, they automatically become available to users in your tenant.

For most Azure tenants, the request to access Agents (preview) is automatically made on your behalf. However, access will be rolled out gradually over time.

> [!TIP]
> Want earlier access to Azure Copilot? [Fill out this form](https://aka.ms/azurecopilot/agents/feedbackprogram) to help us prioritize your organization as we expand capacity. Submission does not guarantee access and still requires that **Request access to Agents (preview)** is selected.

## Disable access to Agents (preview)

If you decide to remove access to Agents (preview), toggle the **Request access to Agents (preview)** setting to **Off**. This action can be performed at any time by any Global Administrator for the tenant.

When you do so, users in the tenant will no longer have access to the preview features. Any ongoing conversations that include agents remain active and visible in history, but no agent functionality can be used. Users can still access the standard Azure Copilot capabilities, unless you have [restricted their access](manage-access.md).

> [!NOTE]
> If you disable access to Agents (preview), users in your tenant won't have access to agent capabilities while they are in preview. However, once these capabilities become generally available, all users in your tenant who can access Azure Copilot will have access to agent capabilities.

## Related content

- Learn how to [manage user access to Azure Copilot](manage-access.md).
- Understand [conversation storage options for Azure Copilot](bring-your-own-storage.md).
