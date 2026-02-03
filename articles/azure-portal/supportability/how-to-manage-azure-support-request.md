---
title: Manage an Azure support request
description: Learn about viewing support requests and how to send messages, upload files, and manage options.
tags: billing
ms.topic: how-to
ms.date: 02/02/2026
# Customer intent: As an Azure user, I want to manage my support requests effectively, so that I can efficiently communicate with support engineers and ensure timely resolution of issues.
---

# Manage an Azure support request

After you [create an Azure support request](how-to-create-azure-support-request.md), you can manage it in the [Azure portal](https://portal.azure.com).

To manage a support request, you must have the [Owner](/azure/role-based-access-control/built-in-roles#owner), [Contributor](/azure/role-based-access-control/built-in-roles#contributor), or [Support Request Contributor](/azure/role-based-access-control/built-in-roles#support-request-contributor) role at the subscription level. To manage a support request that you created without a subscription, you must be an [Admin](/azure/active-directory/roles/permissions-reference).

> [!TIP]
> In addition to using the Azure portal, you can create and manage requests programmatically by using the [Azure support ticket REST API](/rest/api/support) or [Azure CLI](/cli/azure/support). You can also use the [Azure mobile app](https://azure.microsoft.com/get-started/azure-portal/mobile-app/) to view open requests, reply to your support engineer, or edit the severity of your ticket.

## View support requests

View the details and status of support requests in **Help + support** by selecting  **All support requests** in the service menu.

:::image type="content" source="media/how-to-manage-azure-support-request/all-requests-lower.png" alt-text="All support requests":::

You can search, filter, and sort support requests. By default, you might only see recent open requests. To select a longer time range, or to include closed support requests, change the filter options.

To view details about a support request, including its severity and any messages associated with the request, select it from the list.

## Send a message about a support request

To provide additional details about your request after you submit it, send a message.

1. From **All support requests**, select the support request.
1. In the **Support Request**, select **New message**.
1. Enter your message and select **Submit**.

## Change the severity level of a support request

Follow these steps if you need to change the severity level of an existing support request.

> [!NOTE]
> The maximum severity level depends on your [support plan](https://azure.microsoft.com/support/plans). In some cases, you can't change the severity level.

1. From **All support requests**, select the support request.
1. In the **Support Request**, select **Change severity**.

1. The Azure portal shows one of two screens, depending on whether a support engineer is already assigned to your request:

    - If a support engineer isn't assigned, you see a screen like the following. Select a new severity level, and then select **Change**.

      :::image type="content" source="media/how-to-manage-azure-support-request/unassigned-can-change-severity.png" alt-text="Select a new severity level":::

    - If a support engineer is assigned, you see a screen like the following. Select **OK**, and then send a message requesting the severity level change.

      :::image type="content" source="media/how-to-manage-azure-support-request/assigned-cant-change-severity.png" alt-text="Can't select a new severity level":::

      If you urgently need to change the severity level, and the support engineer assigned to your case is unavailable, you can call [customer service](https://support.microsoft.com/en-us/topic/customer-service-phone-numbers-c0389ade-5640-e588-8b0e-28de8afeb3f2#ID0EBBD=signinorgid) (available at all hours) and ask the agent to change the severity level for you.

## Allow collection of advanced diagnostic information

When you create a support request, you select **Yes** or **No** in the **Advanced diagnostic information** section. This option determines whether Azure support can gather [diagnostic information](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) such as [log files](how-to-create-azure-support-request.md#advanced-diagnostic-information-logs) from your Azure resources that can potentially help resolve your issue. Azure support can only access advanced diagnostic information if you create your case through the Azure portal and grant permission to allow it.

To change your **Advanced diagnostic information** selection after creating the request:

1. From **All support requests**, select the support request.
1. In the **Support Request**, select **Advanced diagnostic information**.
1. Select **Yes** or **No**, and then select **Submit**.

    :::image type="content" source="media/how-to-manage-azure-support-request/grant-permission-manage.png" alt-text="Grant permissions for diagnostic information":::

## Upload files to a support request

Use the file upload option to upload a diagnostic file, such as a [browser trace](../capture-browser-trace.md) or any other files that you think are relevant to a support request.

1. From **All support requests**, select the support request.

1. In the **Support Request**, select **Upload file**, and then browse to select one or more files. You can attach up to five files to your support request. To include more files, package them together in a compressed format such as .zip.
1. Select **Upload**.

### File upload guidelines

Follow these guidelines when you use the file upload option:

- To protect your privacy, don't include personal information in your upload.
- The file name can't be longer than 110 characters.
- Files can't be larger than 5 MB.
- All files must have a valid file name extension, such as `.docx` or `.xlsx`. Most file name extensions are supported, but you can't upload files with these extensions: `.bat, .cmd, .exe, .ps1, .js, .vbs, .com, .lnk, .reg, .bin, .cpl, .inf, .ins, .isu, .job, .jse, .msi, .msp, .paf, .pif, .rgs, .scr, .sct, .vbe, .vb, .ws, .wsf, .wsh`

## Close a support request

Select **Close request** to close a support request. When prompted, select **Close** to confirm. You receive a confirmation email when your request is closed.

## Reopen a closed support request

Select **Reopen request** to reopen a closed support request. When prompted, select **Reopen request** to confirm. Your support request is active again, and a support engineer is assigned to it.

> [!NOTE]
> You can generally view and reopen closed support requests for 13 months. After that time, they might be removed, making them unavailable to view or reopen.

## Get help with a support request

If you need assistance managing a support request, [create another support request](how-to-create-azure-support-request.md) to get help. Be sure to explain the issue that you're having with managing your original support request.

## Cancel a support plan

To cancel a support plan, see [Cancel a support plan](/azure/cost-management-billing/manage/cancel-azure-subscription#cancel-a-support-plan-in-the-azure-portal).

## Next steps

- Review the process to [create an Azure support request](how-to-create-azure-support-request.md).
- Learn about the [Azure support ticket REST API](/rest/api/support).
