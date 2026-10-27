---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: Manage the Power Cycle of AWS EC2s
description: Learn how to start, stop, and restart AWS EC2s with the Azure Arc Multicloud Connector 
author:      JalenMcG # GitHub alias
ms.author:   jalenmcgee # Microsoft alias
ms.service: azure-arc
ms.topic: feature-guide
ms.date:     10/07/2026
ms.subservice: servers-azure-arc
---

# Manage EC2 instance power cycle with the multicloud connector enabled by Azure Arc

The **Power Cycle management** solution of the multicloud connector lets you operate your AWS resources from Azure, including start, stop, and restart actions on EC2 instances. This capability extends the connector beyond discovering and Azure Arc-enabling resources, so you can act on the machines you find without switching to the AWS console.

> [!IMPORTANT]
> The **Power Cycle management** solution is currently in preview. It currently supports Amazon Web Services (AWS) only. Google Cloud Platform (GCP) isn't supported.

## Supported actions

| Action | Applies to | Description |
| --- | --- | --- |
| Start | EC2 instances | Starts a stopped instance. |
| Stop | EC2 instances | Stops a running instance. Supports optional force and hibernate behavior. |
| Restart | EC2 instances | Reboots a running instance. |

Creating and deleting EC2 instances isn't supported through the multicloud connector.

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md#prerequisites), meet these requirements.

### Solution dependency

The **Inventory** solution must be enabled on the same connector, with the **EC2** service included in the scanned services. **Power Cycle** actions read instance state from the inventory resource, so you can't add **Power Cycle management** to a connector that doesn't have **Inventory** with EC2.

### Resource provider registration

Register these resource providers in your Azure subscription:

- `Microsoft.AwsConnector`
- `Microsoft.HybridCompute`

### AWS permissions

Your CloudFormation template must grant the connector IAM role permission to run **Power Cycle** operations:

```json
{
  "Sid": "Ec2LifecycleManagement",
  "Effect": "Allow",
  "Action": [
    "ec2:StartInstances",
    "ec2:StopInstances",
    "ec2:RebootInstances"
  ],
  "Resource": "*"
}
```

If you enable **Power Cycle** **management** on an existing connector, upload a new CloudFormation template so these permissions are applied.

### Instance requirements

- The EC2 instance must be discovered and projected into Azure by an inventory sync.
- The instance must be in a supported AWS region.

## Add the solution

Add **Power Cycle** **management** when you [create a connector](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal), or add it to an existing connector:

1. In the Azure portal, go to **Azure Arc**, and under **Management**, select **Multicloud connectors**.
2. Select your AWS connector, and then select **Solutions**.
1. In the **Power Cycle** **management (preview)** row, select **Add**.
4. Complete the **Authentication template** step to apply the updated CloudFormation template in AWS.

## How instance state is determined

Power cycle actions are enabled or disabled based on the most likely current state of the machine. The connector uses a dual-path model, because an Arc heartbeat is fresher than a periodic inventory sync.

### Arc-enabled machines

For machines onboarded to Azure Arc, the `Microsoft.HybridCompute/machines` resource `properties.status` value provides the state. This value reflects the Arc connected heartbeat.

| Arc status | Effective state | Start | Stop | Restart |
| --- | --- | --- | --- | --- |
| Connected | Running | Disabled | Enabled | Enabled |
| Disconnected | Stopped or unavailable | Enabled | Disabled | Disabled |
| Unknown | Unknown | Falls back to inventory state | Falls back to inventory state | Falls back to inventory state |

### Inventory-only machines

For machines that aren't Azure Arc–enabled, the inventory resource provides the state:

`Microsoft.AwsConnector/ec2Instances/default.properties.awsProperties.state.name.value`

| AWS EC2 state | Effective state | Start | Stop | Restart |
| --- | --- | --- | --- | --- |
| `running` | Running | Disabled | Enabled | Enabled |
| `stopped` | Stopped | Enabled | Disabled | Disabled |
| Transitional (`stopping`, `starting`) or unknown | Changing or unknown | Disabled | Disabled | Disabled |

> [!NOTE]
> Inventory state depends on the periodic sync interval rather than a live signal, so it can be stale. Treat it as best effort. A shorter sync interval produces fresher state.

If the connector doesn't have the **Power Cycle** **management** solution enabled, the Start, Stop, and Restart actions are disabled and the portal shows a banner explaining how to enable the solution.

## Query instance power state

Use [Azure Resource Graph](/azure/governance/resource-graph/overview) to report on running and stopped instances across your connected AWS accounts.

```kusto
awsresources
| where type =~ "microsoft.awsconnector/ec2instances"
| extend state = tostring(properties.awsProperties.state.name.value)
| extend instanceType = tostring(properties.awsProperties.instanceType.value)
| project subscriptionId, resourceGroup, name, state, instanceType, location
| order by state asc
```

## Related content

- [What is Multicloud connector enabled by Azure Arc?](overview.md)
- [Connect a public cloud to Azure](add-public-cloud.md)
- [View and manage multicloud inventory](view-multicloud-inventory.md)
- [Onboard VMs to Azure Arc](onboard-multicloud-vms-arc.md)
