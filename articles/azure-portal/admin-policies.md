---
title: Set Azure portal administrator policies
description: Azure portal administrators can set options and policies for users in their organization.
ms.topic: how-to
ms.date: 10/6/2025
# Customer intent: As an Azure portal administrator, I want to understand how to set options for the users in my tenant, so that I can maintain standards and manage options effectively.
---

# Set Azure portal administrator policies

Azure portal administrators (users with the [Global Administrator role](/entra/identity/role-based-access-control/permissions-reference#global-administrator)) can set policies that affect all users in the tenant. This article provides details on some of those options.

## Set feedback policies

Azure portal administrators can enable or disable feedback policy controls for their tenant. These policies can affect whether or not surveys can be shown to users, whether or not Microsoft is able to contact users about their feedback, and other feedback options. By default, all of the feedback options are enabled for users in your tenant.

In order to change feedback policies, you must [elevate your access](/azure/role-based-access-control/elevate-access-global-admin) to ensure that you can access all subscriptions and management groups in your tenant.

Once your access is elevated, open [Feedback Policy Controls](https://portal.azure.com/#view/Microsoft_Azure_Resources/FeedbackPolicyControls.ReactView) in the Azure portal.

:::image type="content" source="media/admin-policies/feedback-policy-controls.png" alt-text="Screenshot of the Feedback Policy Controls option in the Azure portal.":::

You can **Select all** to allow all feedback options, or check individual boxes to allow any of the following options:

- **Initiate and submit feedback to Microsoft**: Allows users to send feedback to Microsoft in the Azure portal, such as by using the feedback option in the page header.
- **Receive and respond to in-product surveys from Microsoft**: Allows users to receive surveys from Microsoft while using the Azure portal and provide responses to survey questions.
- **Provide email address so that Microsoft can follow up on user feedback**: Allows users to provide their own email address when submitting feedback, so that Microsoft can contact them directly.
- **Include log files and content samples when they submit feedback to Microsoft**: Allows users to add Microsoft-generated files, such as log files and content samples, when they provide feedback.
- **Include screenshots and attachments when they submit feedback to Microsoft**: Allows users to add screenshots and other attachments when they provide feedback.

## Set a directory timeout period

Azure portal administrators can change the directory timeout setting for their tenant. This setting controls how long a user can be inactive in the portal before they are signed out.

For more information, see [Change the directory timeout setting](set-preferences.md#change-the-directory-timeout-setting-admin).

## Manage access to Azure Copilot

Azure portal administrators can manage access to Azure Copilot for users in their tenant, including limiting access to specific Microsoft Entra users or groups.

For more information, see [Manage access to Azure Copilot](/azure/copilot/manage-access).
