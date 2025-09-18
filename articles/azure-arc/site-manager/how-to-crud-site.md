---
title: "How to create and manage an Azure Arc site"
description: "Describes how to create, view, delete, or modify an Azure Arc site in the Azure portal using site manager."
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: how-to #Don't change
ms.date: 04/18/2024
ms.custom: sfi-image-nochange

#customer intent: As a site admin, I want to know how to create, delete, and modify sites so that I can manage my site.

# Customer intent: "As a site admin, I want to create, modify, and delete sites using a site manager, so that I can efficiently manage and organize geographically related resources within my Azure environment."
---

# Create and manage sites

This article guides you through how to create, modify, and delete a site using Azure Arc site manager (preview).

## Prerequisites

* An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/free/).
* Azure portal access
* Internet connectivity
* A resource group or subscription in Azure with at least one resource for a site. For more information, see [Supported resource types](./overview.md#supported-resource-types).

## Open Azure Arc site manager

In the [Azure portal](https://portal.azure.com), search for and select **Azure Arc**. Select **Site manager (preview)** from the Azure Arc navigation menu.

![Screenshot of Azure Arc portal view.](media/how-to-crud-site/screenshot-azure-arc-portal-view.jpg)




Alternatively, you can also search for Azure Arc site manager directly in the Azure portal using terms such as **site**, **Arc Site**, **site manager** and so on.

## Create a site

Create a site to manage geographically related resources.

1. From the main **Site manager** page in **Azure Arc**, select the blue **Create a site** button.

   ![Screenshot of Site manager get started tab.](media/how-to-crud-site/screenshot-site-manager-get-started-tab.jpg)
   
   
   
   
   
1. Provide the following information about your site:

   | Parameter | Description |
   |--|--|
   | **Site name** | Custom name for site. |
   | **Display name** | Custom display name for site. |
   | **Site address**| Physical address for a site. Providing Country is mandatory, while Street address, City, State / Province and Zip code are optional.|
   | **Site scope** | Either **Subscription** or **Resource group**. The scope can only be defined at the time of creating a site and can't be modified later. All the resources in the scope can be viewed and managed from site manager.  |
   | **Subscription / Resource group** | Select the Subscription / Resource group as per the site scope. |
   | **Parent Site**| Subscription scope sites do not have a parent site. Resource Group scope sites can only be a child of the parent Subscription scope site.|
   
1. Once all these details are provided, select **Review + create**.

   ![Screenshot of create site basics page.](media/how-to-crud-site/screenshot-create-site-basics-page.jpg)
   
   ![Screenshot of create site scope page.](media/how-to-crud-site/screenshot-create-site-scope-page.jpg)
   
   
   
1. On the summary page, review and confirm the site details then select **Create** to create your site.

   ![Screenshot of create site review page.](media/how-to-crud-site/screenshot-create-site-review-page.jpg)
   
   
   
   
If a site is created from a resource group or subscription that contains resources that are supported by site, these resources will automatically be visible within the created site. 

## View and modify a site

Once you create a site, you can access it and its managed resources through site manager.

1. From the main **Site manager** page in **Azure Arc**, select **Sites** to view all existing sites.

   ![Screenshot of Site manager sites tab.](media/how-to-crud-site/screenshot-site-manager-sites-tab.jpg)
   
   
   
   
1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to delete.

   ![Screenshot of Site manager sites view.](media/how-to-crud-site/screenshot-site-manager-sites-view.jpg)
   
   
   
1. On a specific site's resource page, you can:

   * View resources
   * Modify resources (modifications affect the resources elsewhere as well)
   * View connectivity status
   * View update status
   * View alerts
   * Add new resources

Currently, only some aspects of a site can be modified. These are as follows:

| Site Attribute | Modification that can be done |
|--|--|
| Display name | Update the display name of a site to a new unique name. |
| Site address | Update the address of a site. |

## Delete a site

Deleting a site doesn't affect the resources, resource group, or subscription in its scope. After a site is deleted, the resources of that site will still exist but can't be viewed or managed from site manager. You can create a new site for the resource group or the subscription after the original site is deleted.

1. From the main **Site manager** page in **Azure Arc**, select **Sites** to view all existing sites.

1. On the **Sites** page, you can view all existing sites. Select the name of the site that you want to delete.

1. On the site's resource page, select **Delete**.

   ![Screenshot of Site manager site details view.](media/how-to-crud-site/screenshot-site-manager-site-details.jpg)
   
   
   
