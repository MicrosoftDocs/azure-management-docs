---
title:  Manage access to Microsoft Copilot in Azure
description: Learn how administrators can manage user access to Microsoft Copilot in Azure.
ms.date: 04/08/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.author: jenhayes
author: JnHs
ms.custom:
  - build-2024
  - copilot-learning-hub
  - sfi-ga-nochange
# Customer intent: "As a Global Administrator, I want to manage user access to Microsoft Copilot in Azure, so that I can control whether users in my organization can use its features."
---

# Manage access to Microsoft Copilot in Azure

By default, Copilot in Azure is available to all users in a tenant. However, [Global Administrators](/entra/identity/role-based-access-control/permissions-reference#global-administrator) can manage access to Copilot in Azure for their organization. Access can also be optionally granted to specific Microsoft Entra users or groups.

If Copilot in Azure is not available for a user, they'll see an unauthorized message when they select the **Copilot** button in the Azure portal.

> [!NOTE]
> In some cases, your tenant may not have access to Copilot in Azure by default. Global Administrators can enable access by following the steps described in this article at any time.

As always, Microsoft Copilot in Azure only has access to resources that the user has access to. It can only take actions that the user has permission to perform, and requires confirmation before making changes. Copilot in Azure complies with all existing access management rules and protections such as Azure role-based access control (Azure RBAC), Privileged Identity Management, Azure Policy, and resource locks.

## Manage user access to Microsoft Copilot in Azure

To manage access to Microsoft Copilot in Azure for users in your tenant, any Global Administrator in that tenant can follow these steps.

1. [Elevate your access](/azure/role-based-access-control/elevate-access-global-admin?tabs=azure-portal#step-1-elevate-access-for-a-global-administrator) so that your Global Administrator account can manage all subscriptions in your tenant.

1. In the Azure portal, search for **Copilot in Azure admin center** and select it.

1. In **Copilot in Azure admin center**, under **Settings**, select **Access management**.

1. Select the toggle next to **On for entire tenant** to change it to **Off for entire tenant**.

1. To grant access to specific Microsoft Entra users or groups, select **Manage RBAC roles**.

1. Assign the **Copilot for Azure User** role to specific users or groups. For detailed steps, see [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

1. When you're finished, [remove your elevated access](/azure/role-based-access-control/elevate-access-global-admin?tabs=azure-portal#step-2-remove-elevated-access).

Global Administrators for a tenant can change the **Access management** selection at any time.

> [!IMPORTANT]
> In order to use Microsoft Copilot in Azure, your organization must allow websocket connections to `https://directline.botframework.com`. Please ask your network administrator to enable this connection.

## Next steps

- [Learn more about Microsoft Copilot in Azure](overview.md).
- Read the [Responsible AI FAQ for Microsoft Copilot in Azure](responsible-ai-faq.md).
- Explore the [capabilities](capabilities.md) of Microsoft Copilot in Azure and learn how to [write effective prompts](write-effective-prompts.md).
