---
title: Cloud Solution Provider program considerations
description: For CSP partners, Azure delegated resource management helps improve security and control by enabling granular permissions.
ms.date: 01/21/2026
ms.topic: concept-article
# Customer intent: As a Cloud Solution Provider partner, I want to manage my customers' Azure subscriptions using delegated resource management, so that I can enhance security, improve efficiency, and scale my services while maintaining necessary permissions.
---

# Azure Lighthouse and the Cloud Solution Provider program

If you're a [CSP (Cloud Solution Provider)](/partner-center/enroll/csp-overview) partner, you can already access the Azure subscriptions created for your customers through the CSP program by using the Administer On Behalf Of (AOBO) functionality. This access allows you to directly support, configure, and manage your customers' subscriptions.

By using [Azure Lighthouse](../overview.md), you can use Azure delegated resource management along with AOBO. This approach helps improve security and reduces unnecessary access by enabling more granular permissions for your users. It also allows for greater efficiency and scalability, as your users can work across multiple customer subscriptions by using a single sign-in in your tenant.

> [!TIP]
> To help safeguard customer resources, be sure to review and follow the [recommended security practices](recommended-security-practices.md) along with the [partner security requirements](/partner-center/partner-security-requirements).

## Administer on Behalf of (AOBO)

With AOBO, any user with the [Admin Agent](/partner-center/account-settings/permissions-overview#admin-agent-role) role in your tenant has AOBO access to Azure subscriptions that you create through the CSP program. Any users who need access to any customers' subscriptions must be a member of this group. AOBO doesn't allow the flexibility to create distinct groups that work with different customers, or to enable different roles for groups or users.

![Diagram showing tenant management using AOBO.](../media/csp-1.jpg)

## Azure Lighthouse

By using Azure Lighthouse, you can assign different groups to different customers or roles, as shown in the following diagram. Through [Azure delegated resource management](architecture.md), you can give individual users or groups a role that lets them perform specific tasks on customer resources. Because roles can be configured as needed, you can reduce the number of users who have the Admin Agent role with full AOBO access.

![Diagram showing tenant management using AOBO and Azure Lighthouse.](../media/csp-2.jpg)

Azure Lighthouse helps improve security by limiting broad access to your customers' resources. It also gives you more flexibility to manage multiple customers at scale by using the [Azure built-in role](tenants-users-roles.md#role-support-for-azure-lighthouse) that's most appropriate for each user's duties, without providing your users more access than necessary.

To further minimize the number of permanent assignments, you can [create eligible authorizations](../how-to/create-eligible-authorizations.md) to grant additional permissions to your users on a just-in-time basis.

Onboarding a subscription that you created through the CSP program follows the steps described in [Onboard a customer to Azure Lighthouse](../how-to/onboard-customer.md). Any user who has the Admin Agent role in the customer's tenant can perform this onboarding.

> [!TIP]
> [Managed Service offers](managed-services-offers.md) with private plans aren't supported with subscriptions established through a reseller of the Cloud Solution Provider (CSP) program. Instead, you can onboard these subscriptions to Azure Lighthouse by [using Azure Resource Manager templates](../how-to/onboard-customer.md).

> [!NOTE]
> The [**My customers** page in the Azure portal](../how-to/view-manage-customers.md) now includes a **Cloud Solution Provider (Preview)** section, which displays billing info and resources for CSP customers who have [signed the Microsoft Customer Agreement (MCA)](/partner-center/customers/confirm-customer-agreement) and are [under the Azure plan](/partner-center/customers/azure-plan-get-started). For more info, see [Get started with your Microsoft Partner Agreement billing account](/azure/cost-management-billing/understand/mpa-overview).
>
> CSP customers might appear in this section whether or not they're also onboarded to Azure Lighthouse. If they are, they also appear in the **Customers** section, as described in [View and manage customers and delegated resources](../how-to/view-manage-customers.md#cloud-solution-provider-preview). Similarly, a CSP customer doesn't need to appear in the **Cloud Solution Provider (Preview)** section of **My customers** in order for you to onboard them to Azure Lighthouse.

## Link your partner ID to track your impact on delegated resources

Members of the [Microsoft AI Cloud Partner Program](https://partner.microsoft.com/) can link a partner ID with the credentials they use to manage delegated customer resources. By linking partner IDs, Microsoft can identify and recognize partners who drive Azure customer success. The link also allows [CSP (Cloud Solution Provider)](/partner-center/enroll/csp-overview) partners to receive [partner earned credit (PEC)](/partner-center/billing/partner-earned-credit) for customers who have [signed the Microsoft Customer Agreement (MCA)](/partner-center/customers/confirm-customer-agreement) and are [under the Azure plan](/partner-center/customers/azure-plan-get-started).

To earn recognition for Azure Lighthouse activities, [link your partner ID](/azure/cost-management-billing/manage/link-partner-id) with at least one user account in your managing tenant, and ensure that the linked account has access to each of your onboarded subscriptions. For simplicity, create a service principal account in your tenant, associate it with your Partner ID, then grant it access to every customer you onboard with an [Azure built-in role that is eligible for partner earned credit](/partner-center/billing/azure-roles-perms-pec).

For more information, see [Link a partner ID](/azure/cost-management-billing/manage/link-partner-id).

## Next steps

- Learn about [cross-tenant management experiences](cross-tenant-management-experience.md).
- Learn how to [onboard a subscription to Azure Lighthouse](../how-to/onboard-customer.md).
- Learn about the [Cloud Solution Provider program](/partner-center/csp-overview).
