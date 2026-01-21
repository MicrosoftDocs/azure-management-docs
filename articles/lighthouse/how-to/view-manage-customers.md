---
title: View and manage customers and delegated resources in the Azure portal
description: As a service provider or enterprise using Azure Lighthouse, you can view  delegated resources and subscriptions by going to My customers in the Azure portal. 
ms.date: 01/21/2026
ms.topic: how-to
# Customer intent: As a service provider, I want to view and manage my customers and their delegated resources in the Azure portal, so that I can efficiently consolidate and oversee multiple customer engagements.
---

# View and manage customers and delegated resources in the Azure portal

Service providers using [Azure Lighthouse](../overview.md) can visit **My customers** in the [Azure portal](https://portal.azure.com) to view delegated customer resources and subscriptions.

To view information about a customer, you must have been granted the [Reader](/azure/role-based-access-control/built-in-roles#reader) role (or another built-in role that includes Reader access) when that customer was onboarded.

> [!TIP]
> Though this article refers to service providers and customers, [enterprises managing multiple tenants](../concepts/enterprise.md) can use the same process to consolidate their management experience.

To access **My customers** in the Azure portal, enter "My customers" in the search box in the Azure portal page header. You can also navigate to **Azure Lighthouse** in the Azure portal, then select **Manage your customers**.

The **Customers** section of **My customers** only shows information about customers who delegate subscriptions or resource groups to your Microsoft Entra tenant through Azure Lighthouse. If you work with other customers (such as through the [Cloud Solution Provider (CSP) program](/partner-center/enroll/csp-overview)), you won't see those customers in the **Customers** section unless you [onboarded their resources to Azure Lighthouse](onboard-customer.md). However, you might see details about certain CSP customers in the [**Cloud Solution Provider (Preview)** section](#cloud-solution-provider-preview).

> [!NOTE]
> Your customers can view details about service providers by navigating to **Service providers** in the Azure portal. For more information, see [View and manage service providers](view-manage-service-providers.md).

## View and manage customer details

To view customer details, select **Customers** from the service menu of **My customers**.

For each customer, you see the customer's name and customer ID (tenant ID), along with the **Offer ID** and **Offer version** associated with the engagement. In the **Delegations** column, you see the number of delegated subscriptions and resource groups.

Options at the top of the pane let you sort, filter, and group your customer information by specific customers, offers, or keywords.

For additional details, use the following options:

- To see all of the subscriptions, offers, and delegations associated with a customer, select the customer's name.
- To see details about an offer and its delegations, select the offer name.
- To see details about role assignments for delegated subscriptions or resource groups, select the entry in the **Delegations** column.

> [!NOTE]
> If a customer renames a subscription after it was delegated, you'll see the updated subscription name. However, if they rename their tenant, you might still see the older tenant name in some places in the Azure portal.

## View and manage delegations

Delegations show the subscription or resource group that's been delegated, along with the users and permissions that have access to it. To view this info, select **Delegations** from the service menu.

Options at the top of the pane let you sort, filter, and group this information by specific customers, offers, or keywords.

### View role assignments

The users and permissions associated with each delegation appear in the **Role assignments** column. Select an entry to view more details. After you do so, select **Role assignments** to see the full list of users, groups, and service principals that have been granted access to the subscription or resource group. From there, you can select a particular user, group, or service principal name to see more information.

### Remove delegations

If you included users with the [Managed Services Registration Assignment Delete Role](/azure/role-based-access-control/built-in-roles#managed-services-registration-assignment-delete-role) when onboarding a customer to Azure Lighthouse, those users can remove delegations by selecting the trash can icon that appears in the row for that delegation. When you remove a delegation, users in the service provider's tenant lose the access that the delegation previously granted.

For more information, see [Remove access to a delegation](remove-delegation.md).

## View delegation change activity

The **Activity log** section of **My customers** keeps track of every time that a customer subscription or resource group is delegated to your tenant. It also records removal of any previously delegated resources. To view this information, users must be [assigned the Monitoring Reader role at root scope](monitor-delegation-changes.md#assign-the-monitoring-reader-role-at-root-scope).

For more information, see [View delegation changes in the Azure portal](monitor-delegation-changes.md#view-delegation-changes-in-the-azure-portal).

## Work in the context of a delegated subscription

You can work directly in the context of a delegated subscription within the Azure portal, without switching the directory you're signed in to. To do so:

1. Select the **Settings** icon from the global controls in the Azure portal page header.
1. In [Directories + subscriptions](../../azure-portal/set-preferences.md#directories--subscriptions), ensure that the **Advanced filters** toggle is [turned off](../../azure-portal/set-preferences.md#subscription-filters).
1. In the **Default subscription filter** section, select the appropriate directory and subscription. If you were granted access to a resource group rather than to an entire subscription, select the subscription to which that resource group belongs. You'll work in the context of that subscription, but can only access the designated resource group.

:::image type="content" source="../media/subscription-filter-delegated.png" alt-text="Screenshot of the default subscription filter with one delegated subscription selected.":::

After making this selection, when you access an Azure service that supports [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md), the service defaults to the context of the delegated subscription in your filter.

You can change the default subscription at any time by following the steps above and choosing a different subscription, or multiple subscriptions. If you want the filter to include all of the subscriptions to which you have access, select **All directories**, then check the **Select all** box.

:::image type="content" source="../media/subscription-filter-all.png" alt-text="Screenshot of the default subscription filter with all directories and subscriptions selected":::

> [!IMPORTANT]
> Checking the **Select all** box sets the filter to show all of the subscriptions to which you *currently* have access. If you later gain access to more subscriptions—for example, if you onboard a new customer to Azure Lighthouse—these subscriptions aren't automatically added to your filter. To include them, return to **Directories + subscriptions** and select the additional subscriptions (or uncheck and then recheck **Select all** again).

You can also select a delegated subscription or resource group from within an individual service that supports [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md#enhanced-services-and-scenarios). If you don't see the subscription, check to make sure it's not excluded from your **Default subscription filter**.

## Cloud Solution Provider (Preview)

A separate **Cloud Solution Provider (Preview)** section of **My customers** shows billing information and resources for your CSP customers who have [signed the Microsoft Customer Agreement (MCA)](/partner-center/customers/confirm-customer-agreement) and are [under the Azure plan](/partner-center/customers/azure-plan-get-started). For more information, see [Get started with your Microsoft Partner Agreement billing account](/azure/cost-management-billing/understand/mpa-overview).

These CSP customers appear in this section whether or not you also onboarded them to Azure Lighthouse. Similarly, a CSP customer doesn't have to appear in the **Cloud Solution Provider (Preview)** section of **My customers** in order for you to onboard them to Azure Lighthouse.

## Next steps

- Learn about [cross-tenant management experiences](../concepts/cross-tenant-management-experience.md).
- Learn how your customers can [view and manage service providers](view-manage-service-providers.md) by going to **Service providers** in the Azure portal.
