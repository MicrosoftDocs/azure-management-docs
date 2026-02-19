---
title: Create and Manage Labels for an Azure Arc Site 
description: Learn how to create and manage labels for an Azure Arc site.
author: dawagle
ms.author: dawagle
ms.service: azure-arc
ms.topic: how-to
ms.date: 12/16/2025
ms.subservice: azure-arc-site-manager
---

# Create and manage labels for an Azure Arc site

You can use labels to tag Azure Arc sites with metadata for categorization and operations. You create the labels as key/value pairs (label name: label value).

## Prerequisites

- An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/free/).
- Azure portal access.
- Internet connectivity.
- A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).
- A site created for the associated resource group or subscription or service group. If you don't have one, see [Create and manage sites](./how-to-crud-site.md).

## Create and manage labels

1. In the [Azure portal](https://portal.azure.com/), go to **Azure Arc** and select **Site manager** to open the site manager.

1. From the Azure Arc site manager, go to the **Sites** page.

1. On the **Sites** page, you can see the list of all of your sites. Select the sites that you want and select **Assign labels**.  

1. In the **Assign labels** pane, define the label name and the label value. You can add multiple labels up to a limit of 15 for an individual site. Define the multiple values for a single label by using comma-separated values. The character limit for a label name is 256. The character limit for a label value is 512.  

1. For any prior label name, you can delete the value pair to manage the list of labels for selected sites.  

   :::image type="content" source="media/managesitelabels/sitelabel.png" alt-text="Screenshot that shows how to create a site label.":::

## Filter through labels

1. In the [Azure portal](https://portal.azure.com/), go to **Azure Arc** and select **Site manager** to open the site manager.

1. From the Azure Arc site manager, go to the **Sites** page.

1. On the **Sites** page, you can see the list of all of your sites. The labels associated with a site are listed under the column **Labels**.

1. To filter the list of sites on this page, select **Add filter**. You can enter the label name in the **Filter** field. The **Value** field allows a choice of options from the list of label values detected for that label name.

   :::image type="content" source="media/managesitelabels/sitelabelfiltering.png" alt-text="Screenshot that shows the site list filtering by using labels.":::

## Define a site label strategy  

A planned labeling strategy ensures that categorization aligns with business goals and operational needs. You can assess and plan your specific requirements by using effective standards:

- **Evaluate the categorization needs of your organization.** Your labeling approach must align with organization standards and maintain consistency. Inconsistent tagging creates confusion and reduces the effectiveness of resource management. To ensure that your site labels complement these established practices, review your company's existing naming conventions and governance policies.
- **Identify the operational requirements for your sites.** Site labels capture critical operational metadata that supports consumption across partner products to enable identification and associated operations on sites.