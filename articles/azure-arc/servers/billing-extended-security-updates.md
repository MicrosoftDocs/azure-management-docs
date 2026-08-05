---
title: Billing service for Extended Security Updates for Windows Server through Azure Arc
description: Learn about billing services for Extended Security Updates for Windows Server 2012 and Windows Server 2016 enabled by Azure Arc.
ms.date: 07/16/2026
ms.topic: concept-article
zone_pivot_groups: extended-security-updates-windows-server
# Customer intent: As a cloud administrator managing Extended Security Updates for Windows Server via Azure Arc, I want to understand the billing structure and back-billing implications so that I can accurately forecast costs and optimize my license management.
---

# Billing service for Extended Security Updates for Windows Server enabled by Azure Arc

Three factors affect billing for Extended Security Updates (ESUs):

- The number of cores provisioned
- The edition of the license (Standard vs. Datacenter)
- The application of any eligible discounts

Billing is monthly. Decrementing, deactivating, or deleting a license results in charges for up to five more calendar days from the time of decrement, deactivation, or deletion. Reduction in billing isn't immediate. This is an Azure-billed service and can be used to decrement a customer's Microsoft Azure Consumption Commitment (MACC) and be eligible for Azure Consumption Discount (ACD).

> [!NOTE]
> Licenses or extra cores provisioned after End of Support are subject to a one-time back-billing charge during the month in which the license was provisioned. This isn't reflective of the recurring monthly bill.

## Back-billing for ESUs enabled by Azure Arc

When you provision licenses after the End of Support (EOS) date, you pay back-billing charges for the time elapsed since the EOS date. When you enroll late, you become eligible for all the critical security patches up to that point, and the back-billing charge reflects the value of these critical security patches. The EOS date depends on the version of Windows Server.

::: zone pivot="windows-server-2012"

The EOS date for Windows Server 2012 and 2012 R2 is October 10, 2023. For example, an ESU license provisioned in December 2023 is back-billed for October and November upon provisioning.

::: zone-end

::: zone pivot="windows-server-2016"

The EOS date for Windows Server 2016 is January 12, 2027. Licenses provisioned after that date are back-billed to January 12, 2027. Billing for Windows Server 2016 ESUs enabled by Azure Arc begins January 13, 2027.

::: zone-end

If you deactivate and then reactivate a license, you're billed for the window during which the license was deactivated. It's not possible to evade charges by deactivating a license before a critical security patch and reactivating it shortly before.

If the region or the tenant of an ESU license is changed, this is subject to back-billing charges.

> [!NOTE]
> The back-billing cost appears as a separate line item in invoicing. If you acquired a discount for your core Windows Server ESUs enabled by Azure Arc, the same discount might or might not apply to back-billing. Verify that the same discounting, if applicable, is applied to back-billing charges as well.

Estimates in the Azure Cost Management forecast might not accurately project monthly costs. Due to the episodic nature of back-billing charges, the projection of monthly costs might appear as overestimated during initial months.

## Billing associated with modifications to an Azure Arc ESU license

- **License type:** License type (either Standard or Datacenter) is an immutable property. The billing associated with a license is specific to the edition of the provisioned license.

    > [!NOTE]
    > If you previously provisioned a Datacenter Virtual Core license, it's charged with and offer the virtualization benefits associated with the pricing of a Datacenter edition license.

- **Core modification:** If cores are added to an existing ESU license, they're subject to back-billing charges for the time elapsed since EOS. The new cores will then be regularly billed from the calendar month in which they were added. If cores are reduced or decremented to an existing ESU license, the billing rate reflects the reduced number of cores within five days of the change.

- **Activation:** Licenses are billed for their number and edition of cores from the point at which they're activated. The activated license doesn't need to be linked to any Azure Arc-enabled servers to initiate billing. Activation and reactivation are subject to back-billing. Licenses that were activated but not linked to any servers may be back-billed if they weren't billed upon creation. Customers are responsible for deletion of any activated but unlinked ESU licenses.

- **Deactivation or deletion:** Licenses that are deactivated or deleted are billed through up to five calendar days from the time of the change.

   > [!NOTE]
   > If you delete and then recreate an ESU license, back-billing still applies for the corresponding period. Deletion doesn't exempt you from charges for that period.
   >
   > In principle, there are no cases in which back-billing is waived after reactivation or recreation, and there are no conditions under which it can be avoided.
   >
   > Before performing reactivation or recreation, always confirm the billing start date and the conditions under which back-billing will occur. We also recommend reviewing the publicly available information on pricing calculations.

## Billing for transition scenario for Volume Licensing

::: zone pivot="windows-server-2012"

Licenses for Windows Server 2012/R2 ESUs enabled by Azure Arc that have been provisioned with the specification of an Invoice Id for the Year 1 Volume Licensing entitlement won't be charged until October 10, 2024. These licenses won't be back-billed to October 2023. Licenses with Year 1 created after October 10, 2024 will be back-billed to October 10, 2024, the last day of the Year 1 of WS2012/R2 ESU program. Customers don't need to reactivate or recreate licenses between years of the WS2012/R2 ESU program.

::: zone-end

::: zone pivot="windows-server-2016"

Transitioning from Volume Licensing isn't supported for Windows Server 2016 ESUs enabled by Azure Arc.

::: zone-end

## Services included with Windows Server ESUs enabled by Azure Arc

When you purchase Windows Server ESUs enabled by Azure Arc, you get access to more Azure management services at no extra cost for enrolled servers. To learn more, see [Access to Azure services](prepare-extended-security-updates.md#access-to-azure-services).

Azure Arc-enabled servers give you the flexibility to evaluate and operationalize Azure’s robust security, monitoring, and governance capabilities for your non-Azure infrastructure. These services deliver key value beyond the observability, ease of enrollment, and financial flexibility of Windows Server ESUs enabled by Azure Arc.

