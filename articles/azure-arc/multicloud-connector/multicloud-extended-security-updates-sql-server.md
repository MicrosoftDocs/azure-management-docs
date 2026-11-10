---
title: Manage SQL Server licensing and subscribe to Extended Security Updates with the multicloud connector
description: Learn how to use the SQL Server licensing and ESU solution of the multicloud connector enabled by Azure Arc to manage licensing and subscribe eligible SQL Server instances in AWS and GCP to Extended Security Updates.
ms.service: azure-arc
ms.topic: how-to
ms.date: 11/10/2026
---

# Manage SQL Server licensing and subscribe to Extended Security Updates with the multicloud connector enabled by Azure Arc

The **SQL Server licensing and ESU** solution of the multicloud connector lets you manage licensing and subscribe to Extended Security Updates (ESU) for eligible SQL Server instances. You set the license type and subscription choice once on the connector, and eligible instances on Arc-enabled servers in AWS and GCP are configured as they're discovered.

This solution covers the SQL Server product. To cover the underlying Windows Server operating system, also add the [**Windows Server ESU** solution](multicloud-extended-security-updates.md). A machine that requires ESU coverage for both SQL Server and Windows Server needs both solutions.

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md), meet these requirements.

### Solution dependency

The **Arc onboarding** solution must be enabled on the same connector. SQL Server instances are detected through the Azure Connected Machine agent and the Azure extension for SQL Server, so the AWS EC2 instance or GCP VM must be Arc-enabled first.

### Cloud permissions

This solution requires no additional AWS or GCP permissions. The default permission is **None**.

### Instance requirements

- SQL Server 2014 (12.x) or SQL Server 2016 (13.x) is installed on the Arc-enabled machine.
- Eligible instances are covered by Software Assurance or a pay-as-you-go subscription.
- You purchased ESU through volume licensing for all prior ESU coverage years for the eligible instances on the machine.

## License types

The solution applies one license type to the instances it subscribes to. Choose the option that matches how you license SQL Server on these machines.

| Option | Use when |
| --- | --- |
| **Pay-as-you-go** | You want to run production Enterprise or Standard edition on the server with pay-as-you-go billing (PAYG). |
| **License with Software Assurance** | You have production Enterprise or Standard edition on the server covered by Software Assurance or a SQL subscription (Paid). |

## Add the solution

1. In the Azure portal, go to **Azure Arc**, and under **Management**, select **Multicloud connectors**.
2. Select your AWS or GCP connector, and then select **Solutions**.
3. In the **SQL Server licensing and ESU** row, select **Add**.
4. In **SQL Server licensing and ESU settings**, choose a **License Type**.
5. Under **Extended Security Updates**, select **Subscribe to Extended Security Updates**.
6. Select **Save**.

## What you confirm when you subscribe

By subscribing to Extended Security Updates, you confirm that:

- You purchased ESU through volume licensing for all prior ESU coverage years for eligible SQL Server instances on the machines connected through this connector.
- If a connected machine has SQL Server instances from multiple eligible versions, you will be billed separately for each eligible version.
- If a connected machine has an instance of Analysis Services, Integration Services, Reporting Services, or Power BI Report Server, ESU will be billed based on the installed edition and version.
- Based on your SQL Server configuration and activation day, an estimated one-time bill-back charge is applied immediately, followed by an estimated hourly charge. These charges appear as separate line items on your bill.
- ESUs are free for eligible resources in Azure Dev/Test subscriptions, and eligible HADR replicas aren't charged.

## Unsubscribe

To terminate the subscription and stop ongoing charges immediately, open the solution settings, select **Unsubscribe from Extended Security Updates**, and then select **Save**.

Subscriptions also end automatically when you migrate the instance to Azure or upgrade it to a supported version of SQL Server. If you delete a source machine in AWS or GCP, periodic sync removes the corresponding Arc-enabled server and its SQL Server instance resources in Azure, which ends billing for that instance.

## Find eligible instances

Use [Azure Resource Graph](/azure/governance/resource-graph/overview) to list connected SQL Server 2014 and SQL Server 2016 instances discovered through AWS and GCP connectors.

```kusto
resources
| where type =~ 'microsoft.hybridcompute/machines'
| where resourceGroup startswith "AWS_" or resourceGroup startswith "GCP_"
| where tostring(properties.status) =~ 'Connected'
| project id, machineId = tolower(id), location
| join kind=inner (
    resources
    | where type =~ 'microsoft.hybridcompute/machines/extensions'
    | where name in~ ('WindowsAgent.SqlServer', 'LinuxAgent.SqlServer')
    | extend settings = parse_json(tolower(tostring(properties.settings)))
    | extend enableExtendedSecurityUpdates = coalesce(tostring(settings.enableextendedsecurityupdates), 'ESUNotProvided')
    | extend licenseType = coalesce(tostring(settings.licensetype), 'NotProvided')
    | project extensionMachineId = tolower(substring(id, 0, indexof(id, '/extensions'))), extensionId = id
) on $left.machineId == $right.extensionMachineId
| join kind=inner (
    resources
    | where type =~ 'microsoft.azurearcdata/sqlserverinstances'
    | where tostring(properties.version) in~ ('SQL Server 2014', 'SQL Server 2016')
    | project machineId = tolower(tostring(properties.containerResourceId)), version = tostring(properties.version), edition = tostring(properties.edition)
    | distinct machineId, version, edition
) on $left.machineId == $right.machineId
| project extensionId, location, version, edition
```

## Patch delivery

After you subscribe to an instance, ESU patches arrive through the same channels as regular SQL Server updates: Microsoft Update, Windows Update, Microsoft Configuration Manager, [Azure Update Manager](/azure/update-manager/overview), or direct download from the Azure portal.

Instances with automatic updates enabled receive ESUs automatically. For instances with automatic updates disabled, download the patches manually from the **Extended Security Updates** pane for the SQL Server instance in the Azure portal.

Microsoft releases SQL Server ESUs only when the Microsoft Security Response Center rates a vulnerability as **Critical**, so there's no regular release cadence. ESUs don't include new features, functional improvements, or customer-requested fixes.

## Related content

- [Onboard VMs to Azure Arc](onboard-multicloud-vms-arc.md)
- [Attach Windows Server Extended Security Updates licenses](multicloud-extended-security-updates.md)
