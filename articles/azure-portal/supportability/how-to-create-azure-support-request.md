---
title: How to create an Azure support request
description: Customers who need assistance can use the Azure portal to find self-service solutions and to create and manage support requests.
ms.topic: how-to
ms.custom: support-help-page
ms.date: 08/14/2025
# Customer intent: As an Azure user seeking assistance, I want to create and manage support requests through the portal or API, so that I can effectively resolve issues and access technical support for my resources.
---

# Create an Azure support request

Azure enables you to create and manage support requests, also known as support tickets. You can create and manage requests in the Azure portal, as described in this article. You can also create and manage requests programmatically, using the [Azure support ticket REST API](/rest/api/support), or by using [Azure CLI](/cli/azure/support).

> [!NOTE]
> The Azure portal URL is specific to the Azure cloud where your organization is deployed.
>
>- Azure portal for commercial use is: [https://portal.azure.com](https://portal.azure.com)
>- Azure portal for the United States government is: [https://portal.azure.us](https://portal.azure.us)

Azure provides unlimited support for subscription management, which includes billing, [quota adjustments](../../quotas/quotas-overview.md), and account transfers. For technical support, you need a support plan. For more information, see [Compare support plans](https://azure.microsoft.com/support/plans).

## Getting started

You can open support requests in the Azure portal from the Azure portal menu, the global header, or the resource menu for a service. Before you can file a support request, you must have appropriate permissions.

### Azure role-based access control

You must have the appropriate access to a subscription in order to create a support request for it. This means you must have the [Owner](/azure/role-based-access-control/built-in-roles#owner), [Contributor](/azure/role-based-access-control/built-in-roles#contributor), or [Support Request Contributor](/azure/role-based-access-control/built-in-roles#support-request-contributor) role, or a custom role with [Microsoft.Support/*](/azure/role-based-access-control/resource-provider-operations#microsoftsupport), at the subscription level.

To create a support request without a subscription, for example a Microsoft Entra scenario, you must be an [Admin](/azure/active-directory/roles/permissions-reference).

> [!IMPORTANT]
> If a support request requires investigation into multiple subscriptions, you must have the required access for each subscription involved ([Owner](/azure/role-based-access-control/built-in-roles#owner), [Contributor](/azure/role-based-access-control/built-in-roles#contributor), [Reader](/azure/role-based-access-control/built-in-roles#reader), [Support Request Contributor](/azure/role-based-access-control/built-in-roles#support-request-contributor), or a custom role with the [Microsoft.Support/supportTickets/read](/azure/role-based-access-control/resource-provider-operations#microsoftsupport) permission).

If a support request requires confirmation or release of account-specific information, changes to account information, or operations such as subscription ownership transfer or cancelation, you must be an [account billing administrator](/azure/cost-management-billing/manage/add-change-subscription-administrator#determine-account-billing-administrator) for the subscription.

### Open a support request from the global header

To start a support request from anywhere in the Azure portal:

1. Select the **?** in the global header, then enter a few words to describe your issue.

   :::image type="content" source="media/how-to-create-azure-support-request/support-menu-issue.png" alt-text="Screenshot of the Help menu from the global header in the Azure portal.":::

1. Follow the prompts to share more details about your issue, including the specific resource, if applicable. We'll look for solutions related to your issues. In some cases, we might perform diagnostic checks to further investigate. In many cases, these solutions are able to resolve the problem completely.

   If none of the solutions resolve the problem, select **Create a support request**, then follow the steps to complete your request.

   :::image type="content" source="media/how-to-create-azure-support-request/header-create-support-request.png" alt-text="Screenshot of the Help menu with Create a support request link.":::

### Open a support request from a resource menu

To start a support request in the context of the resource you're currently working with:

1. From the resource menu, in the **Help** section, select **Support + Troubleshooting**.

   :::image type="content" source="media/how-to-create-azure-support-request/resource-context-support.png" alt-text="Screenshot of the Create a support request option in the resource pane.":::

1. Follow the prompts  to share more details about your issue. Some options may be preselected for you, based on the resource you were viewing when you selected **Support + Troubleshooting**. We'll look for solutions related to your issues. In some cases, we might perform diagnostic checks to further investigate. In many cases, these solutions are able to resolve the problem completely.

  If none of the solutions resolve the problem you're having, select **Create a support request**, then follow the steps to complete your request.

## Create a support request

When you create a new support request, you'll need to provide some information to help us understand the problem. This information is gathered in a few separate sections. In most cases, information you provided earlier will be prefilled for you. You can make any changes and enter additional details as needed.

### Problem description

The **Problem description** section is the first step of the support request process. Details  you provided earlier will be prefilled in this section, but you can make changes if needed. If you change the issue type, the previous information might not be saved.

You may need to provide additional information related to the issue type you select. If you select **Technical**, specify the service that your issue relates to. Depending on the service, you might see options for **Problem type** and **Problem subtype**. Be sure to select the service (and problem type/subtype if applicable) that is most related to your issue. Selecting an unrelated service may result in delays in addressing your support request.

> [!IMPORTANT]
> In most cases, you'll need to specify a subscription. Be sure to select the subscription where you are experiencing the problem. The support engineer assigned to your case will only be able to access resources in the subscription you specify. The access requirement serves as a point of confirmation that the support engineer is sharing information to the right audience, which is a key factor for ensuring the security and privacy of customer data. For details on how Azure treats customer data, see [Data Privacy in the Trusted Cloud](https://azure.microsoft.com/overview/trusted-cloud/privacy/).
>
> If the issue applies to multiple subscriptions, you can mention additional subscriptions in your description, or in a [message you send](how-to-manage-azure-support-request.md#send-a-message) later. However, the support engineer will only be able to work on [subscriptions to which you have access](#azure-role-based-access-control). If you don't have the required access for a subscription, we won't be able to work on it as part of your request.

:::image type="content" source="media/how-to-create-azure-support-request/support-request-problem-description.png" alt-text="Screenshot of the Problem description step of the support request process.":::

After you provide all of the requested information, select **Next**.

### Recommended solution

Based on the information you provided, we provide some recommended solutions that might be able to fix the problem. In some cases, we may even run a quick diagnostic check. These solutions are written by Azure engineers to solve most common problems.

If you're still unable to resolve the issue, continue creating your support request by selecting **Return to support request**, then selecting **Next**.

### Additional details

Next, we collect more details about the problem. Providing thorough and detailed information in this step helps us route your support request to the right engineer. Be sure not to include any personal or confidential information here.

1. Complete the **Problem details** so that we have more information about your issue. If possible, tell us when the problem started and any steps to reproduce it. You can optionally upload one file (or a compressed file such as .zip that contains multiple files), such as a log file or [browser trace](../capture-browser-trace.md). For more information on file uploads, see [File upload guidelines](how-to-manage-azure-support-request.md#file-upload-guidelines).

1. In the **Advanced diagnostic information** section, select **Yes** or **No**. Selecting **Yes** allows Azure support to gather [advanced diagnostic information](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/) from your Azure resources. If you prefer not to share this information, select **No**. For details about the types of files we might collect, see [Advanced diagnostic information logs](#advanced-diagnostic-information-logs).

   In some cases, you may see additional options. For example, for certain types of Virtual Machine problem types, you can choose whether to [allow access to a virtual machine's memory](#memory-dump-collection).

1. In the **Support method** section, select the **Support plan** and choose a **Severity** level that reflects the business impact. The [maximum available severity level and time to respond](https://azure.microsoft.com/support/plans/response/) depends on your [support plan](https://azure.microsoft.com/support/plans) and the country/region in which you're located, including the timing of business hours in that country/region.

   > [!TIP]
   > To add a support plan that requires an **Access ID** and **Contract ID**, go to **Help + Support**. In the service menu, under **Support**, select **Support Plans**. Select **Link support benefits**, then enter your information to see available plans. When a limited support plan expires or has no support incidents remaining, it won't be available to select.

1. Provide your preferred contact method, such as **Email** or **Phone**, then select your preferred **Support language**. You may also see an option to **Connect now** and chat with a support engineer. Chat is only supported in English language.

1. Complete the **Contact info** section so that we know how to reach you. Confirm that your country/region setting is accurate, as this setting affects the business hours in which a support engineer can work on your request.

  > [!TIP]
  > By enabling **Save contact changes for future support requests**, your personal contact info is saved for future use. However, the **Additional email for notification** value is not saved.

When you finish entering this information, select **Next**.

### Review + create

Before you create your request, review all of the details that you'll send to support. You can select **Previous** to return to any tab if you want to make changes. When you're satisfied that the support request is complete, select **Create**.

A support engineer will contact you using the method you indicated. For information about initial response times, see [Support scope and responsiveness](https://azure.microsoft.com/support/plans/response/).

### Advanced diagnostic information logs

When you allow collection of [advanced diagnostic information](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/), Microsoft support can collect information that can help solve your problem more quickly. Files commonly collected for different services or environments include:

- [Microsoft Azure PaaS VM logs](/troubleshoot/azure/virtual-machines/sdp352ef8720-e3ee-4a12-a37e-cc3b0870f359-windows-vm)
- [Microsoft Azure IaaS VM logs](https://github.com/azure/azure-diskinspect-service/blob/master/docs/manifest_by_file.md)
- [Microsoft Azure Service Fabric logs](/troubleshoot/azure/general/fabric-logs)
- [StorSimple support packages and device logs](https://support.microsoft.com/topic/storsimple-support-packages-and-device-logs-cb0a1c7e-6125-a5a7-f212-51439781f646)
- [Microsoft Entra logs](/troubleshoot/azure/active-directory/support-data-collection-diagnostic-logs)
- [Azure Stack Edge support package and device logs](/troubleshoot/azure/general/azure-stack-edge-support-package-device-logs)
- [Azure Synapse Analytics logs](/troubleshoot/azure/general/synapse-analytics-apache-spark-pools-diagnostic-logs)

Depending on your issue or environment type, we might collect other files in addition to the ones listed here. For more information, see [Data we use to deliver Azure support](https://azure.microsoft.com/support/legal/support-diagnostic-information-collection/).

### Memory dump collection

When you create a support case for certain Virtual Machine (VM) problem types, you choose whether to allow us to access your virtual machine's memory. If you do so, we may collect a memory dump to help diagnose the problem.

A complete memory dump is the largest kernel-mode dump file. This file includes all of the physical memory that is used by Windows. A complete memory dump does not, by default, include physical memory that is used by the platform firmware.

The dump is copied from the compute node (Azure host) to another server for debugging within the same datacenter. Customer data is protected, since the data doesn't leave Azure's secure boundary.

The dump file is created by generating a Hyper-V save state of the VM. During this process, the VM will be paused for up to 10 minutes, after which time the VM is resumed. The VM isn't restarted as part of this process.

## Next steps

- Understand how to [manage an Azure support request](how-to-manage-azure-support-request.md)
- Get help from your peers in [Microsoft Q&A](/answers/products/azure)
- Use the [Azure support ticket REST API](/rest/api/support)
- Review the [Azure support plans FAQ](https://azure.microsoft.com/support/faq)
- Learn about [Azure Quotas](../../quotas/quotas-overview.md)
