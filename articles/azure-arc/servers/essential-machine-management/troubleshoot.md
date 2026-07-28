---
title: Troubleshoot Essential Machine Management (preview)
description: Describes how to troubleshoot Essential Machine Management to automatically configure management for VMs in your subscription.
ms.topic: troubleshooting-general
ms.date: 07/27/2026
---

# Troubleshoot Essential Machine Management (preview)

This article provides troubleshooting steps for issues that might occur when you enable [Essential Machine Management](./enrollment.md). If you receive an error during enrollment, the error message usually provides a specific resolution. If you don't get any errors during enrollment, but the machines in the subscription aren't onboarding to the selected services, use the following sections to validate the different steps of the enrollment process to identify where any issues might have occurred.

## Errors during enrollment

The following are common errors that might occur when you enable Essential Machine Management for a subscription.

### Could not validate resource existence

The error message includes the resource ID of the Log Analytics workspace or Azure Monitor workspace that you selected during enrollment.

1. Check whether you have the **Essential Machine Management Administrator** role in the resource group of the Log Analytics workspace or Azure Monitor workspace.
1. If the workspaces are in a different subscription than the one you're enabling for Essential Machine Management, verify that you have `Microsoft.ManagedOps` resource provider registered in the subscription.

### Change Log Analytics workspace or Azure Monitor workspace

If you already configured Essential Machine Management and then enable it again by using a different Log Analytics workspace or Azure Monitor workspace, you get an error saying that you can't change the workspace after it's set.

To change either of the workspaces, you must first [disable the subscription](./enrollment.md#disable-a-subscription) and then re-enable it by using the new workspaces. All machines in the subscription are re-enrolled and configured with the new workspaces, but any data already collected in the old workspace is retained.

### Disable Defender for cloud

You receive an error if you attempt to disable Defender for Cloud for a subscription that you already enabled for Essential Machine Management. You must disable the subscription from the Defender for Cloud portal.

## Verify created objects

If you don't see any errors during enrollment, start by verifying that the objects in the following table are created in the resource group for the Log Analytics workspace and Azure Monitor workspace. These objects are the [data collection rules (DCRs)](/azure/azure-monitor/data-collection/data-collection-rule-overview) and solutions that enable change tracking and data collection for Azure Monitor.

| Type | Name | Description |
|:---|:---|:---|
| DCR | `<workspace id>-Managedops-AM-DCR` <sup>1</sup> | OpenTelemetry metrics from VM guests |
| DCR | `<workspace id>-Managedops-CT-DCR` <sup>1</sup> | Change tracking and inventory. Collects files, registry keys, software, Windows services, Linux daemons |
| Solution | `ChangeTracking(workspace name)` | Solution added to Log Analytics workspace to support Change tracking and inventory. |

<sup>1</sup> The `<workspace id>` in the DCR name might be truncated.

Verify that the alert rules were created by checking for the following rules in the resource group for the Azure Monitor workspace.

- `ManagedOps-High-CPU-Usage-Alert`
- `ManagedOps-High-Disk-IOPS-Alert`
- `ManagedOps-High-Network-Errors-Alert`
- `ManagedOps-High-Network-Inbound-Traffic-Alert`
- `ManagedOps-High-Network-Outbound-Traffic-Alert`
- `ManagedOps-Low-Available-Memory-Alert`
- `ManagedOps-Slow-Disk-Operations-Alert`
- `ManagedOps-VM-Availability-Alert`

:::image type="content" source="./media/troubleshoot/resource-group-objects.png" lightbox="./media/troubleshoot/resource-group-objects.png" alt-text="Screenshot of objects in the resource group created by subscription enablement.":::

## Check deployments for errors

If you don't see these objects within a few minutes of enabling the subscription, check for errors in the deployments that create them. Open **Deployments** in the resource group and search for deployments with `Managedops` in the name. Select **Related events** to view the [Activity log](/azure/azure-monitor/platform/activity-log) entries for the deployment. Alternatively, you can check the Activity log directly for the resource group and search for `Managedops` to identify any activity related to machine enablement.

The deployment names look similar to the following examples:

- `Managedops-ChangeTracking-{Subscription Id}`
- `Managedops-AzureMonitor-{Subscription Id}`

:::image type="content" source="./media/troubleshoot/deployments.png" lightbox="./media/troubleshoot/deployments.png" alt-text="Screenshot of deployments in the resource group created by subscription enablement.":::

## Verify policy assignments

If the required objects are created, and there are no errors in the deployments, verify that the policy assignment exists in the subscription. The assignment is responsible for applying the required configurations to the VMs in the subscription.

Open the **Policy** page in the Azure portal and select **Assignments**. Search for `ManagedOpsPolicy`. If you don't see the policy assignment, you might not have enough permission to make a policy assignment in that subscription. Verify permissions at [Required permissions](./enrollment.md#required-permissions).

:::image type="content" source="./media/troubleshoot/initiatives.png" lightbox="./media/troubleshoot/initiatives.png" alt-text="Screenshot of initiatives in the resource group created by subscription enablement.":::

## Check remediation tasks

If the assignments look correct, check the remediation tasks that enable the selected features on all existing VMs in the subscription. Select the assignment and select **Remediation**. If any of the **Remediated Resources** don't show that all resources are remediated, or if the **Remediation State** is **Failed**, select the **Complete** or **Failed** status to open the details for that remediation task.

:::image type="content" source="./media/troubleshoot/remediations.png" lightbox="./media/troubleshoot/remediations.png" alt-text="Screenshot of remediation tasks for the initiative created by subscription enablement.":::

In the details of the remediation task, select **Related Events** or **View Deployment** to locate a detailed error message.

:::image type="content" source="./media/troubleshoot/remediation-task.png" lightbox="./media/troubleshoot/remediation-task.png" alt-text="Screenshot of details of a remediation task.":::

## Contact Microsoft

If you completed all the previous steps and still have problems with Essential Machine Management, contact Microsoft at [machineenrollmentsupport@microsoft.com](mailto:machineenrollmentsupport@microsoft.com).
