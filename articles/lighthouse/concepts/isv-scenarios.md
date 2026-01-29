---
title: Azure Lighthouse in ISV scenarios
description: ISVs can use the capabilities of Azure Lighthouse for more flexibility with customer offerings.
ms.date: 01/20/2026
ms.topic: concept-article
# Customer intent: "As an Independent Software Vendor, I want to use Azure Lighthouse to manage customer resources in a secure and scalable way, so that I can offer enhanced managed services while maintaining control over my intellectual property and support processes."
---

# Azure Lighthouse in ISV scenarios

A typical scenario for [Azure Lighthouse](../overview.md) involves a service provider that manages resources in its customers' Microsoft Entra tenants. Independent Software Vendors (ISVs) using SaaS-based offerings with their customers can also benefit from the capabilities of Azure Lighthouse. Azure Lighthouse is especially helpful for ISVs who offer managed services that require access to a customer's subscription scope.

## Managed Service offers in Microsoft Marketplace

As an ISV, you might already have published solutions to Microsoft Marketplace. If you offer managed services to your customers, you can publish a Managed Service offer. These offers streamline the onboarding process and make your services more scalable to as many customers as needed. Azure Lighthouse supports a wide range of [management tasks and scenarios](cross-tenant-management-experience.md#enhanced-services-and-scenarios) that you can use to provide value to your customers.

For more information, see [Publish a Managed Service offer to Microsoft Marketplace](../how-to/publish-managed-services-offers.md).

## Using Azure Lighthouse with Azure managed applications

[Azure managed applications](/azure/azure-resource-manager/managed-applications/overview) are another way that ISVs can provide services to their customers. Use Azure Lighthouse along with your Azure managed applications to enable enhanced scenarios.

For more information, see [Azure Lighthouse and Azure managed applications](managed-applications.md).

## SaaS-based multitenant offerings

In another scenario, the ISV hosts resources in a subscription in their own tenant, then uses Azure Lighthouse to let customers access those specific resources. Once this access is granted, the customer can sign in to their own tenant and access the resources as needed. The ISV maintains their IP in their tenant and can use their own support plan for requests related to the solution. Since the resources are in the ISV's tenant, the ISV can easily perform all needed actions, such as signing into VMs, installing apps, and handling maintenance tasks.

In this scenario, users in the customer's tenant are essentially granted access as a "managing tenant," even though the customer isn't managing the ISV's resources. Because the customer is directly accessing the ISV's tenant, it's important to grant only the minimum permissions necessary, so that they can't make changes to the solution or access other ISV resources.

To enable this architecture, the ISV needs to obtain the object ID for a user group in the customer's Microsoft Entra tenant, along with their tenant ID. The ISV then builds an ARM template granting this user group the appropriate permissions, and [deploys it on the ISV's subscription](../how-to/onboard-customer.md) that contains the resources that the customer will access.

## Next steps

- Learn about [cross-tenant management experiences](cross-tenant-management-experience.md).
- Learn more about [Azure Lighthouse architecture](architecture.md).
