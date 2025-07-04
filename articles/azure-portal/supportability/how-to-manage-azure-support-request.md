---
title: Manage an Azure support request
description: Learn about viewing support requests and how to send messages, upload files, and manage options.
tags: billing
ms.topic: how-to
ms.date: 12/03/2024
# Customer intent: As an Azure user, I want to manage my support requests effectively, so that I can efficiently communicate with support engineers and ensure timely resolution of issues.
---

# Manage an Azure support request

After you [create an Azure support request](how-to-create-azure-support-request.md), you can manage it in the [Azure portal](https://portal.azure.com).

> [!TIP]
> You can create and manage requests programmatically by using the [Azure support ticket REST API](/rest/api/support) or [Azure CLI](/cli/azure/azure-cli-support-request). Additionally, you can view open requests, reply to your support engineer, or edit the severity of your ticket in the [Azure mobile app](https://azure.microsoft.com/get-started/azure-portal/mobile-app/).

To manage a support request, you must have the [Owner](/azure/role-based-access-control/built-in-roles#owner), [Contributor](/azure/role-based-access-control/built-in-roles#contributor), or [Support Request Contributor](/azure/role-based-access-control/built-in-roles#support-request-contributor) role at the subscription level. To manage a support request that was created without a subscription, you must be an [Admin](/azure/active-directory/roles/permissions-reference).

## View support requests

View the details and status of support requests by going to **Help + support** >  **All support requests** in the Azure portal.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="All support requests":::

You can search, filter, and sort support requests. By default, you might only see recent open requests. Change the filter options to select a longer period of time or to include support requests that were closed.

To view details about a support request to view details, including its severity and any messages associated with the request, select it from the list.

## Send a message

1. From **All support requests**, select the support request.

1. In the **Support Request**, select **New message**.

1. Enter your message and select **Submit**.

## Change the severity level

> [!NOTE]
> The maximum severity level depends on your [support plan](https://azure.microsoft.com/support/plans).

1. From **All support requests**, select the support request.

1. In the **Support Request**, select **Change severity**.

1. The Azure portal shows one of two screens, depending on whether your request is already assigned to a support engineer:

    - If your request isn't assigned, you see a screen like the following. Select a new severity level, then select **Change**.

      :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Select a new severity level":::

    - If your request is assigned, you see a screen like the following. Select **OK**, then create a [new message](#send-a-message) to request a change in severity level. If you have an urgent need to change the severity level, and the support engineer assigned to your case is unavailable, you can call [customer service](https://support.microsoft.com/en-us/topic/customer-service-phone-numbers-c0389ade-5640-e588-8b0e-28de8afeb3f2#ID0EBBD=signinorgid) (available at all hours) and ask the agent to change the severity level for you.

      :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="Can't select a new severity level":::

## Allow collection of advanced diagnostic information

When you create a support request, you can select **Yes** or **No** in the **Advanced diagnostic information** section. This option determines whether Azure support can gather [diagnostic information](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) such as [log files](how-to-create-azure-support-request.md#advanced-diagnostic-information-logs) from your Azure resources that can potentially help resolve your issue. Azure support can only access advanced diagnostic information if your case was created through the Azure portal and you granted permission to allow it.

To change your **Advanced diagnostic information** selection after the request is created:

1. From **All support requests**, select the support request.

1. In the **Support Request**, select **Advanced diagnostic information** near the top of the screen.

1. Select **Yes** or **No**, then select **Submit**.

    :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Grant permissions for diagnostic information":::

## Upload files

You can use the file upload option to upload a diagnostic file, such as a [browser trace](../capture-browser-trace.md) or any other files that you think are relevant to a support request.

1. From **All support requests**, select the support request.

1. In the **Support Request**, select the **Upload file** box, then browse to find your file and select **Upload**.

### File upload guidelines

Follow these guidelines when you use the file upload option:

- To protect your privacy, don't include personal information in your upload.
- The file name must be no longer than 110 characters.
- You can't upload more than one file. To include multiple different files, package them together in a compressed format such as .zip.
- Files can't be larger than 4 MB.
- All files must have a valid file name extension, such as `.docx` or `.xlsx`. Most file name extensions are supported, but you can't upload files with these extensions: `.bat, .cmd, .exe, .ps1, .js, .vbs, .com, .lnk, .reg, .bin,. cpl, .inf, .ins, .isu, .job, .jse, .msi, .msp, .paf, .pif, .rgs, .scr, .sct, .vbe, .vb, .ws, .wsf, .wsh`

## Close a support request

To close a support request, select the **Close request** option near the top of the screen. When prompted to confirm, select **Close**. You'll receive a confirmation email when your request is closed.

## Reopen a closed request

To reopen a closed support request, select **Reopen request** near the top of the screen. When prompted to confirm, select **Reopen request.** Your support request will then be active again.

> [!NOTE]
> Closed support requests can generally be viewed and reopened for a period of 13 months. After that time, they may be removed, making them unavailable to view or reopen.

## Cancel a support plan

To cancel a support plan, see [Cancel a support plan](/azure/cost-management-billing/manage/cancel-azure-subscription#cancel-a-subscription-in-the-azure-portal).

## Get help with a support request

If you need assistance managing a support request, [create another support request](how-to-create-azure-support-request.md) to get help. For the **Issue type**, select **Technical**, then select **All Services**. For **Service type**, select **Portal** and for **Problem type** select **Issue with Support Ticket Experience**.

## Next steps

- Review the process to [create an Azure support request](how-to-create-azure-support-request.md).
- Learn about the [Azure support ticket REST API](/rest/api/support).
