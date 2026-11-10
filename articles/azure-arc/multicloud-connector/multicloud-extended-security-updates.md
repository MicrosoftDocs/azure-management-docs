---
title: Attach Windows Server Extended Security Updates licenses with the multicloud connector
description: Learn how to use the Windows Server ESU solution of the multicloud connector enabled by Azure Arc to attach an ESU license to eligible Arc-enabled Windows Server machines in AWS and GCP.
ms.service: azure-arc
ms.topic: how-to
ms.date: 11/09/2026
---

# Attach Windows Server Extended Security Updates licenses with the multicloud connector enabled by Azure Arc

The **Windows Server ESU** solution of the multicloud connector attaches an Extended Security Updates (ESU) license to eligible Azure Arc-enabled Windows Server machines. Instead of linking licenses machine by machine, you select a license once on the connector, and eligible machines that the connector onboards from AWS and GCP are covered as they're discovered.

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md#prerequisites), meet these requirements.

### Solution dependency

The **AWS Arc EC2 onboarding** or **GCP Arc VM onboarding** solution must be enabled on the same connector. ESU is delivered through the Azure Connected Machine agent, so an AWS EC2 instance or GCP VM must be Arc-enabled before it can receive updates. You can't add **Windows Server ESU** to a connector that doesn't have **AWS Arc EC2 onboarding** or **GCP Arc VM onboarding**.

### Cloud permissions

This solution requires no additional AWS or GCP permissions. The default permission is **None**.

### License requirement

You must already have a Windows Server ESU license in Azure that meets all of these conditions:

| Attribute | Required value |
| --- | --- |
| Version | Windows Server 2016 |
| SKU | Standard |
| Core type | Virtual cores (vCore) |
| Activation status | Activated |

The solution attaches an existing license. It doesn't create one. To create a license, see [Deliver Extended Security Updates for Windows Server](../servers/deliver-extended-security-updates.md).

> [!NOTE]
> Standard SKU with virtual cores is required because source-cloud VMs run on third-party hosts. Virtual core licensing requires a minimum of eight virtual cores per VM, and you select Standard even when the guest operating system is Datacenter edition. For details, see [License provisioning guidelines](../servers/license-extended-security-updates.md).

## Add the solution

1. In the Azure portal, go to **Azure Arc**, and under **Management**, select **Multicloud connectors**.
2. Select your AWS or GCP connector, and then select **Solutions**.
3. In the **Windows Server ESU** row, select **Add**.
4. In **Windows Server ESU settings**, select an existing, activated Standard SKU Windows Server 2016 vCore ESU license from the dropdown.
5. Select **Save**.

If the dropdown is empty, no license in the subscription meets all four conditions in the previous table. Create or activate a qualifying license first.

## Manage core coverage

You're responsible for managing your ESU license and all linked machines so that core counts are provisioned accurately. Insufficient core coverage can result in noncompliance.

Because the connector attaches the license as it onboards eligible machines, license consumption grows as your multicloud estate grows. Plan for this growth:

- Review the number of cores on the license against the number of eligible machines on a regular basis.
- Increase the core count on the license before onboarding a large batch of Windows Server 2016 instances.
- Reduce the core count or deactivate the license as machines are upgraded, migrated, or decommissioned. Billing follows the cores provisioned on an activated license and doesn't adjust on its own.

A single ESU license covers up to 10,000 cores. Split larger estates across multiple licenses.

## Find eligible machines

Use Azure Resource Graph to list eligible Arc-enabled Windows Server 2016 machines onboarded from AWS and GCP, along with their core counts and license assignment states.

```kusto
resources
| where type =~ 'microsoft.hybridcompute/machines'
| where resourceGroup startswith "AWS_" or resourceGroup startswith "GCP_"
| extend osSku = tolower(tostring(properties.osSku)), cores = toint(properties.detectedProperties.coreCount)
| extend licenseAssignmentState = tolower(tostring(properties.licenseProfile.esuProfile.licenseAssignmentState))
| extend esuKeyState = tolower(tostring(properties.licenseProfile.esuProfile.esuKeyState))
| extend esuEligibility = tolower(tostring(properties.licenseProfile.esuProfile.esuEligibility))
| extend agentVersion = tostring(properties.agentVersion)
| extend agentMajor = toint(split(agentVersion, '.')[0]), agentMinor = toint(split(agentVersion, '.')[1])
| where osSku contains '2016' and licenseAssignmentState != 'assigned' and esuKeyState != 'active' and esuEligibility == 'eligible' and (agentMajor > 1 or (agentMajor == 1 and agentMinor >= 62))
| project name, resourceGroup, osSku, cores, status = tostring(properties.status), licenseAssignmentState
```

## Install the updates

Attaching a license makes a machine eligible to receive ESU patches. The machine still needs a patching solution to install them. You can use Azure Update Manager, Windows Server Update Services, Microsoft Update, Microsoft Configuration Manager, or a third-party patch management solution.

To confirm enrollment on a machine, run `azcmagent show` and check that the **Extended Security Updates** section reports a status of **Active**.

## Related content

- [Onboard VMs to Azure Arc](onboard-multicloud-vms-arc.md)
- [Deliver Extended Security Updates for Windows Server](../servers/deliver-extended-security-updates.md)
- [License provisioning guidelines for Extended Security Updates for Windows Server](../servers/license-extended-security-updates.md)
- [Troubleshoot delivery of Extended Security Updates for Windows Server](../servers/troubleshoot-extended-security-updates.md)
