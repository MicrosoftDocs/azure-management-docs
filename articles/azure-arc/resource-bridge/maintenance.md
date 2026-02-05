---
title: Azure Arc resource bridge maintenance operations
description: Learn how to manage Azure Arc resource bridge so that it remains online and operational.
ms.topic: concept-article
ms.date: 1/21/2025
# Customer intent: As an IT administrator responsible for managing an on-premises appliance VM, I want to perform maintenance operations on the Azure Arc resource bridge, so that I can ensure it remains online, secure, and operational for managing my on-premises resources effectively.
---

# Azure Arc resource bridge maintenance

To keep your Azure Arc resource bridge healthy and available, you need to perform maintenance such as updating credentials, monitoring upgrades, and creating a resource health alert. You can also update the proxy settings if there are changes post-deployment.

> [!IMPORTANT]
> Arc resource bridge can't be offline for longer than 45 days because the security key within the appliance VM may expire and can't be refreshed. As a best practice, [create a resource health alert](#create-resource-health-alerts) in the Azure portal to stay informed if an Arc resource bridge becomes unavailable.

## Prerequisites

The management machine used to perform maintenance must meet all of [the Arc resource bridge requirements](system-requirements.md).  

The following sections describe common maintenance tasks for Arc resource bridge.

## Update credentials in the appliance VM

Arc resource bridge consists of an on-premises appliance VM. The appliance VM [stores credentials](system-requirements.md#user-account-and-credentials) that are used to access the control plane of the on-premises infrastructure to view and manage on-premises resources (ex: vCenter credentials). The credentials used by Arc resource bridge are the same ones provided during deployment. It gives the resource bridge visibility to on-premises resources for guest management in Azure. If the credentials change (ex: your company requires a password change every 90 days), then the credentials stored in the Arc resource bridge must be updated. 

You can test if the credentials within the appliance VM are valid by going to the Azure portal and performing an action on a machine that's Arc-enabled via the resource bridge. You can also attempt to [upgrade the resource bridge](upgrade.md#manual-upgrade). If you receive an error, then it is possible that the credentials need to be updated. For guidance on maintaining credentials for Arc-enabled VMware, see [Update the vSphere account credentials](../vmware-vsphere/administer-arc-vmware.md#update-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding). For Arc-enabled SCVMM, see [Update the SCVMM account credentials](../system-center-virtual-machine-manager/administer-arc-scvmm.md).

## Upgrade Arc resource bridge

Each Arc-enabled private cloud has its own upgrade guidelines and procedures. For more information, please refer to the [Upgrade page](upgrade.md).

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

## Update proxy settings (Preview)

Starting with appliance version 1.7.0 and `az arcappliance` CLI version 1.7.0, you can update proxy settings on an appliance using the Azure CLI command: [`az arcappliance configuration proxy update`](/cli/azure/arcappliance/configuration/proxy/update). The [proxy update command](/cli/azure/arcappliance/configuration/proxy/update) is only supported for Azure Local and Arc-enabled VMware. 

Use this command when:

- Your organization switches to a new proxy server.
- You need to disable proxy usage entirely.
- You want to rotate proxy credentials or certificates.

If an upgrade fails due to incorrect network proxy settings, you can run a proxy update to fix the values. If a proxy update operation fails, you must retry and have it succeed before performing any other operation, including retrying upgrade.

## Recovery procedure

Each Arc private cloud has its own recovery procedure that should be followed to ensure a successful redeployment of the Arc resource bridge and re-connection to the existing custom location and Arc private cloud extension.

For Arc-enabled VMware, you can follow the [Arc-enabled VMware recovery procedure](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion).

For Arc-enabled SCVMM, you can follow the [Arc-enabled SCVMM recovery procedure](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery).

For Azure Local, you must contact support and only attempt to recover the Arc resource bridge with guidance by Microsoft. Arc resource bridge is a critical component to Azure Local and attempting a major operation without guidance may cause irrecoverable damage to your Azure Local environment.


## Delete Arc resource bridge

> [!IMPORTANT]
> For Azure Local, do not delete the Arc resource bridge unless you are given guidance by Microsoft. Arc resource bridge is a critical component to Azure Local and deleting it without guidance may cause irrecoverable damage to your Azure Local environment.

You might need to delete Arc resource bridge due to deployment failures, or when the resource bridge is no longer needed. Use the [`az arcappliance delete` command](deploy-cli.md#az-arcappliance-delete) to delete the Arc resource bridge. This command deletes the on-premises appliance VM, along with the Azure resource and underlying components across the two environments. Manually deleting the appliance VM or Azure resource may cause errors in future deployments as the connections between the two resources still exist on the backend.

## Next steps

- Learn about [upgrading Arc resource bridge](upgrade.md).
- Review the [Azure Arc resource bridge overview](overview.md) to understand more about requirements and technical details.
- Learn about [system requirements for Azure Arc resource bridge](system-requirements.md).
