---
title: Quickstart - Request a quota increase in the Azure portal
description: This quickstart shows you how to increase a quota in the Azure portal.
ms.date: 05/21/2026
ms.topic: how-to
# Customer intent: "As an Azure account administrator, I want to request a quota increase through the Azure portal, so that I can ensure my services have the necessary resources to operate effectively."
---

# Quickstart: Request a quota increase in the Azure portal

This article guides you through requesting a quota increase.

For more information about quotas, see [Quotas overview](quotas-overview.md).

## Prerequisites

An Azure account with the Contributor role (or another role that includes Contributor access).

Only paid subscriptions can requesst quota increases. For more information on the limitations, see [Azure subscription and service limits, quotas, and constraints](/azure/azure-resource-manager/management/azure-subscription-service-limits#managing-limits). 


## Request a quota increase

You can submit a request for a quota increase directly from **My quotas**. For this example, you can select any adjustable quota in your subscription.

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Enter *quotas* into the search box, and then select **Quotas**.

   :::image type="content" source="media/quickstart-increase-quota-portal/quotas-portal.png" alt-text="Screenshot of the Quotas service page in the Azure portal.":::

1. On the Overview page, select a provider, like **Compute** or **Networking**.
1. On the **My quotas** page, under **Quota name**, select the quota you want to increase. 
1. Depending on the provider, you'll see a **Request increase** or an **Adjustable** column. 
    - If there is a pencil icon or **Yes** with a pencil icon, you can select the pencil icon to request an increase for that quota.
      
            a. The **New Quota Request** window will open.
         
            b. Type in the number for the new quota and then select **Submit**.
         
            c. Your request will be reviewed, and you'll be notified if the request can be fulfilled. This usually happens within a few minutes.

            d. If your request isn't fulfilled, you'll see a link to create a support request. 
       
     - If there is support request icon or **No** with a support request icon, it is considered a non-adjustable quota and will will need to create a support request to request an increase. For more information, see [Request an increase for non-adjustable quotas](per-vm-quota-requests.md#request-an-increase-for-non-adjustable-quotas).

## Next steps

- [Increase VM-family vCPU quotas](per-vm-quota-requests.md)
- [Increase regional vCPU quotas](regional-quota-requests.md)
- [Increase spot vCPU family quotas](spot-quota.md)
- [Increase networking quotas](networking-quota-requests.md)
- [Increase Azure Storage account quotas](storage-account-quota-requests.md)
