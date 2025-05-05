---
title: Billing service for Extended Security Updates for Windows Server 2012 through Azure Arc
description: Learn about billing services for Extended Security Updates for Windows Server 2012 enabled by Azure Arc.
ms.date: 01/21/2025
ms.topic: concept-article
---

# Billing service for Extended Security Updates for Windows Server 2012 enabled by Azure Arc

Three factors affect billing for Extended Security Updates (ESUs):

- The number of cores provisioned
- The edition of the license (Standard vs. Datacenter)
- The application of any eligible discounts

Billing is monthly. Decrementing, deactivating, or deleting a license results in charges for up to five more calendar days from the time of decrement, deactivation, or deletion. Reduction in billing isn't immediate. This is an Azure-billed service and can be used to decrement a customer's Microsoft Azure Consumption Commitment (MACC) and be eligible for Azure Consumption Discount (ACD).

> [!NOTE]
> Licenses or extra cores provisioned after End of Support are subject to a one-time back-billing charge during the month in which the license was provisioned. This isn't reflective of the recurring monthly bill.

## Back-billing for ESUs enabled by Azure Arc

Licenses that are provisioned after the End of Support (EOS) date of October 10, 2023 are charged a back bill for the time elapsed since the EOS date. For example, an ESU license provisioned in December 2023 is back-billed for October and November upon provisioning. Enrolling late in Windows Server 2012 ESUs makes you eligible for all the critical security patches up to that point. The back-billing charge reflects the value of these critical security patches.

If you deactivate and then reactivate a license, you're billed for the window during which the license was deactivated. It's not possible to evade charges by deactivating a license before a critical security patch and reactivating it shortly before.

If the region or the tenant of an ESU license is changed, this is subject to back-billing charges.

> [!NOTE]
> The back-billing cost appears as a separate line item in invoicing. If you acquired a discount for your core Windows Server 2012 ESUs enabled by Azure Arc, the same discount might or might not apply to back-billing. You should verify that the same discounting, if applicable, has been applied to back-billing charges as well.
> 

Note that estimates in the Azure Cost Management forecast might not accurately project monthly costs. Due to the episodic nature of back-billing charges, the projection of monthly costs might appear as overestimated during initial months.

## Billing associated with modifications to an Azure Arc ESU license

- **License type:** License type (either Standard or Datacenter) is an immutable property. The billing associated with a license is specific to the edition of the provisioned license.

    > [!NOTE]
    > If you previously provisioned a Datacenter Virtual Core license, it's charged with and offer the virtualization benefits associated with the pricing of a Datacenter edition license.
    > 

- **Core modification:** If cores are added to an existing ESU license, they're subject to back-billing (that is, charges for the time elapsed since EOS) and regularly billed from the calendar month in which they were added. If cores are reduced or decremented to an existing ESU license, the billing rate reflects the reduced number of cores within five days of the change.

- **Activation:** Licenses are billed for their number and edition of cores from the point at which they're activated. The activated license doesn't need to be linked to any Azure Arc-enabled servers to initiate billing. Activation and reactivation are subject to back-billing. Licenses that were activated but not linked to any servers may be back-billed if they weren't billed upon creation. Customers are responsible for deletion of any activated but unlinked ESU licenses.

- **Deactivation or deletion:** Licenses that are deactivated or deleted will be billed through up to five calendar days from the time of the change.

## Billing for transition scenario for Volume Licensing

Licenses for Windows Server 2012/R2 ESUs enabled by Azure Arc that have been provisioned with the specification of an Invoice Id for the Year 1 Volume Licensing entitlement won't be charged until October 10, 2024. These licenses won't be back-billed to October 2023. Licenses with Year 1 created after October 10, 2024 will be back-billed to October 10, 2024, the last day of the Year 1 of WS2012/R2 ESU program. Customers don't need to reactivate or recreate licenses between years of the WS2012/R2 ESU program.

## Services included with Windows Server 2012 ESUs enabled by Azure Arc

Purchase of Windows Server 2012/R2 ESUs enabled by Azure Arc provides you with the benefit of access to more Azure management services at no extra cost for enrolled servers. See [Access to Azure services](prepare-extended-security-updates.md#access-to-azure-services) to learn more.

Azure Arc-enabled servers allow you the flexibility to evaluate and operationalize Azure’s robust security, monitoring, and governance capabilities for your non-Azure infrastructure, delivering key value beyond the observability, ease of enrollment, and financial flexibility of Windows Server 2012 ESUs enabled by Azure Arc. 

## Additional notes

- You'll be billed if you connect an activated Azure Arc ESU license to environments like Azure Local or Azure VMware Solution. These environments are eligible for free Windows Server 2012 ESUs enabled by Azure Arc and shouldn't be activated through Azure Arc.

- You'll be billed for all of the cores provisioned in the license. If provision licenses for free ESU usage like Visual Studio Development environments, you shouldn't provision additional cores for the scope of licensing applied to non-paid ESU coverage.

- Migration and modernization of End-of-Life infrastructure to Azure, including Azure VMware Solution and Azure Local, can reduce the need for paid Windows Server 2012 ESUs. You must decrement the cores with their Azure Arc ESU licenses or deactivate and delete ESU licenses to benefit from the cost savings associated with Azure Arc’s flexible monthly billing model. This isn't an automatic process.

- For customers seeking to transition from Volume Licensing based Multiple Activation Keys (MAKs) for Year 1 of WS2012/R2 ESUs to WS2012/R2 ESUs enabled by Azure Arc for Year 2, [there's a transition process](license-extended-security-updates.md#scenario-5-you-have-already-purchased-the-traditional-windows-server-2012-esus-through-volume-licensing) that is exempt from back-billing.
