---
title: Set Azure portal administrator policies
description: You can use the Azure portal on all modern devices and with the latest browser versions.
ms.topic: concept-article
ms.date: 09/30/2025
ms.custom: accessibility
# Customer intent: As an Azure portal administrator, I want to understand how to set options for the users in my tenant, so that I can maintain standards and manage options effectively.
---

# Set Azure portal administrator policies

Azure portal administrators (users with the [Global Administrator role](/entra/identity/role-based-access-control/permissions-reference#global-administrator)) can set policies that affect all users in the tenant. This article provides details on some of those options.

## Set feedback policies

Azure portal administrators can enable or disable feedback policy controls for their tenant. These policies can affect whether or not surveys can be shown to users, whether or not Microsoft is able to contact users about their feedback, and other feedback options.

In order to set feedback policies, you must [elevate your access](/azure/role-based-access-control/elevate-access-global-admin) to ensure that you can access all subscriptions and management groups in your tenant.

:::image type="content" source="media/admin-policies/feedback-policy-controls.png" alt-text="Screenshot of the Feedback Policy Controls option in the Azure portal.":::

## Set a directory timeout period

Azure portal administrators can change the directory timeout setting for their tenant. This setting controls how long a user can be inactive in the portal before they are signed out.

For more information, see [Change the directory timeout setting](set-preferences.md#change-the-directory-timeout-setting-admin).

## Manage access to Microsoft Copilot in Azure

Azure portal administrators can manage access to Microsoft Copilot in Azure for users in their tenant, including limiting access to specific Microsoft Entra users or groups.

For more information, see [Manage access to Microsoft Copilot in Azure](/azure/copilot/manage-access).
