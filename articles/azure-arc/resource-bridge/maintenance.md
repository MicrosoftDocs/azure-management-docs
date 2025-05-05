---
title: Azure Arc resource bridge maintenance operations
description: Learn how to manage Azure Arc resource bridge so that it remains online and operational.
ms.topic: concept-article
ms.date: 1/21/2025
---

# Azure Arc resource bridge maintenance operations

To keep your Azure Arc resource bridge deployment online and operational, you need to perform maintenance operations such as updating credentials, monitoring upgrades, and ensuring the appliance VM is online.

> [!IMPORTANT]
> Arc resource bridge can't be offline for longer than 45 days. After 45 days, the security key within the appliance VM may no longer be valid and can't be refreshed. As a best practice, [create a resource health alert](#create-resource-health-alerts) in the Azure portal to stay informed if an Arc resource bridge becomes unavailable.

## Prerequisites

To maintain the on-premises appliance VM, the [appliance configuration files generated during deployment](deploy-cli.md#az-arcappliance-createconfig) need to be saved in a secure location and made available on the management machine.

The management machine used to perform maintenance operations must meet all of [the Arc resource bridge requirements](system-requirements.md).  

The following sections describe common maintenance tasks for Arc resource bridge.

## Update credentials in the appliance VM

For more information on maintaining credentials for Arc-enabled VMware, see [Update the vSphere account credentials](../vmware-vsphere/administer-arc-vmware.md#updating-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding). For Arc-enabled SCVMM, see [Update the SCVMM account credentials](../system-center-virtual-machine-manager/administer-arc-scvmm.md).

Arc resource bridge consists of an on-premises appliance VM. The appliance VM [stores credentials](system-requirements.md#user-account-and-credentials) that are used to access the control plane of the on-premises infrastructure to view and manage on-premises resources. The credentials used by Arc resource bridge are the same ones provided during deployment of the resource bridge, which gives the resource bridge visibility to on-premises resources for guest management in Azure. If the credentials change, the credentials stored in the Arc resource bridge must be updated. 

You can test if the credentials within the appliance VM are valid by going to the Azure portal and performing an action on an Arc-enabled Private Cloud VM. If you receive an error, then it is possible that the credentials need to be updated.

## Troubleshoot Arc resource bridge

If you experience problems with the appliance VM, the appliance configuration files can help with troubleshooting. You can include these files when you [open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md).

You might want to [collect logs](/cli/azure/arcappliance/logs#az-arcappliance-logs-vmware), which requires you to pass credentials to the on-premises control center:

- For VMware vSphere, use the username and password provided to Arc resource bridge at deployment.
- For Azure Local, see [Collect logs](/azure/azure-local/manage/collect-logs).

## Delete Arc resource bridge

> [!IMPORTANT]
> For Azure Local, do not delete the Arc resource bridge unless you are given guidance by Microsoft. Arc resource bridge is a critical component to Azure Local and deleting it without guidance may cause irrecoverable damage to your Azure Local environment.

You might need to delete Arc resource bridge due to deployment failures, or when the resource bridge is no longer needed. Use the [`az arcappliance delete` command](deploy-cli.md#az-arcappliance-delete) to delete the Arc resource bridge. This command deletes the on-premises appliance VM, along with the Azure resource and underlying components across the two environments. Manually deleting the appliance VM or Azure resource may cause errors in future deployments as the connections between the two resources still exist on the backend.

## Create resource health alerts

You can [create a resource health alert rule](/azure/service-health/resource-health-alert-monitor-guide) in the Azure portal to monitor the state of your Arc resource bridge. Follow these steps to create an alert that notifies you if an Arc resource bridge becomes unavailable.

1. In the Azure portal, search and navigate to **Service Health**.
1. In the service menu, under **RESOURCE HEALTH**, select **Resource health**.
1. In the **Subscription** dropdown, select the subscription used to deploy your resource bridge.
1. In the **Resource type** dropdown, select **Azure Arc Resource Bridge**. 
1. Select the resource bridge(s) from the list for which you want to configure alerts. If you want to set up alerts for all the resource bridges in your subscription, you can select **Add resource health alert** without selecting any resource bridges. This will also add health alerts for resource bridges you may deploy in the future.
1. To receive notifications only when the resource bridge becomes unhealthy, set the following conditions in the **Condition** tab:

   - **Event status**: **Active**
   - **Current resource status**: **Unavailable**
   - **Previous resource status**: **Available**

1. Select one or more **Reason type** values for your alert:

   - **Platform Initiated** : Alerts you when a resource becomes unavailable due to platform issues.
   - **Unknown**: Alerts you when a resource becomes unavailable, but the reason isn't known.
   - **User Initiated**: Alerts you when a resource becomes unavailable due to an action taken by a user.

1. Select **Next: Actions** to continue. In the **Actions** tab, if you want to receive an email when the alert is triggered, select **Use quick actions (preview)** and complete the following:

   1. Enter an **Action group name** and **Display name**
   1. Check the **Email** box and enter an email address.
   1. Select **Save.**

1. Select **Next: Details** to continue. In the **Details** tab:

   1. Select the resource group and region in which to create the alert rule.
   1. Enter a name for your alert rule, and a description if desired.

1. Select **Review + create**, then select **Create**.

For more information about resource health alert rule options, see [Create or edit an activity log, service health, or resource health alert rule](/azure/azure-monitor/alerts/alerts-create-activity-log-alert-rule?tabs=resource-health).

## Next steps

- Learn about [upgrading Arc resource bridge](upgrade.md).
- Review the [Azure Arc resource bridge overview](overview.md) to understand more about requirements and technical details.
- Learn about [system requirements for Azure Arc resource bridge](system-requirements.md).
