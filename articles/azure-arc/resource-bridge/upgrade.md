---
title: Upgrade Arc resource bridge
description: Learn how to upgrade Arc resource bridge using either cloud-managed upgrade or manual upgrade.  
ms.date: 10/03/2024
ms.topic: how-to
# Customer intent: "As a cloud administrator, I want to upgrade the Arc resource bridge using either cloud-managed or manual methods, so that I can maintain supported versions and ensure the appliance remains operational and compliant with upgrade requirements."
---

# Upgrade Arc resource bridge

This article describes how Arc resource bridge is upgraded, and the two ways upgrade can be performed: cloud-managed upgrade or manual upgrade. Currently, some private cloud providers differ in how they handle Arc resource bridge upgrades. 

## Private cloud providers

Private cloud providers have different support policies and upgrade procedures for Arc resource bridge. Review the sections below to learn how to upgrade your Arc resource bridge for your private cloud.

### Arc-enabled VMware vSphere
For **Arc-enabled VMware vSphere**, you must manual upgrade  every 6 months. Choose a version released in the last six months or one of the three most recent versions — whichever is most recent. We recommend performing manual upgrades every 6 months to refresh critical certificates within the appliance before expiration. If these certificates expire or the appliance is offline for longer than 45 days, your resource bridge may need to be [manually recovered](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion). To find your appliance version and its release date, see the [Arc resource bridge release notes](release-notes.md). Microsoft may offer supplemental cloud-managed upgrades as limited assistance, but you are primarily responsible for ensuring your appliance is on a supported version with regular manual upgrades. 

If your appliance is version 1.0.15 or higher, it is automatically opted into supplemental cloud-managed upgrades. Microsoft may upgrade your Arc resource bridge if your appliance is close to being unsupported. However, supplemental cloud-managed upgrades may not succeed due to network disruptions or errors. Don’t wait for a cloud upgrade when your appliance is about to expire. Manually upgrade to avoid service disruption.

### Azure Local
For **Azure Arc VM management on Azure Local**, appliance version 1.0.15 or higher is available only on Azure Local build 23H2. In this version, use the built-in LCM tool to manage upgrades for Azure Local, Arc resource bridge, and extensions as a single package. Remove any preview version of Arc resource bridge before updating from 22H2 to 23H2. Do not upgrade Arc resource bridge separately from other Azure Local components, as this can cause severe issues. For details, see [About updates for Azure Local](/azure/azure-local/update/about-updates-23h2).

### Azure Arc-enabled SCVMM
For **Arc-enabled System Center Virtual Machine Manager (SCVMM)**, you are responsible for upgrading your Azure Arc resource bridge to a version that has been released within the past 6 months or within the latest n-3 versions, whichever is the most recent. We recommend performing manual upgrades at least every 6 months to refresh critical certificates within the appliance before expiration. If these certificates expire or the appliance is offline for longer than 45 days, you may need to [perform recovery of the resource bridge](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery). 

Check your appliance version and its release date to estimate the last upgrade. Make sure you upgrade at least every six months. For version information, please refer to [Arc resource bridge release notes](release-notes.md). Manual upgrade is available for appliance version 1.0.15 and higher. Appliances running a version lower than 1.0.15 need to perform the recovery option to get to version 1.0.15 or higher. Review the steps for [performing the recovery operation](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery). 


## Overview

The upgrade process deploys a new resource bridge using the reserved appliance VM IP. When the new resource bridge is ready, it becomes active. The upgrade process deletes the old resource bridge and reserves its VM IP for the next upgrade.

The upgrade process consists of the following actions:

- Download the appliance image (~3.5 GB) from the cloud.
- Use the image to deploy a new appliance VM.
- Verify that the new resource bridge is running and connect it to Azure.
- Delete the old appliance VM.
- Reserve the old IP for a future upgrade.

The upgrade usually takes at least 30 minutes, depending on network speed. Expect a brief downtime during the transition from the old resource bridge to the new one. More downtime may occur if prerequisites are not met or if there are network issues.


## Prerequisites

Before an Arc resource bridge can be upgraded, the following prerequisites must be met:

- Arc resource bridge must be online and healthy with a status of `Running`. You can check the Azure resource of your Arc resource bridge to verify.

- The [credentials in the appliance VM](maintenance.md#update-credentials-in-the-appliance-vm) must be valid. To test the credentials, perform an operation on an Arc-enabled VM from Azure.

- Arc resource bridge must be in the same location path where it was originally deployed.

- The appliance VM needs 35 GB of free space.

- For Arc-enabled VMware, upgrading the resource bridge requires 200 GB of free space on the datastore. A new template is also created.

- (Manual upgrade only) Run the upgrade command from the management machine used to initially deploy the Arc resource bridge. You can also run the upgrade command from a different machine that meets the [management machine requirements](system-requirements.md#management-machine-requirements).

- (Manual upgrade only) The management machine needs 3.5 GB of free space.

## Check the version
Check the Azure resource of your resource bridge in Azure Resource Manager to view the appliance version. If the appliance status or provisioningState is “UpgradeFailed” or “Failed”, an upgrade attempt may have failed. Upon upgrade failure, the appliance version shown in Azure Resource Manager or via the Azure CLI `show` command may not reflect the actual version. The actual version is most likely the version prior to upgrading. 

## Supplemental cloud-managed upgrade for Arc-enabled VMware

Microsoft may offer cloud-managed upgrades as a supplementary service for Arc-enabled VMware. However, you still need to keep your appliance on a supporte version. You should manual upgrade every 6 months. Microsoft may attempt to upgrade your Arc resource bridge at any time if it will soon be out of support. The upgrade prerequisites must be met for cloud-managed upgrade to work. If your appliance is soon to be unsupported, do no wait for cloud-managed upgrade - perform a manual upgrade as soon as possible.

To check the status and version of your resource bridge:
1. View the Azure resource for your Arc resource bridge.
2. Make sure the status is "Running." 

If your appliance VM is not healthy, cloud-managed upgrades may fail. If an upgrade fails, the reported version may be incorrect. The upgrade must succeed for your appliance to be on the new version.

Azure manages cloud upgrades. During an upgrade, the resource bridge changes status to show each step. Upgrade is complete when the appliance VM `status` is `Running` and `provisioningState` is `Succeeded`.  

To check the status of a cloud-managed upgrade, run the following Azure CLI command from the management machine:  

```azurecli
az arcappliance show --resource-group [REQUIRED] --name [REQUIRED] 
```

## Manual upgrade

> [!WARNING] 
> For Azure Local, you must use the built-in Azure Local LCM tool to upgrade Arc resource bridge. If you attempt to manual upgrade using the Azure CLI command, your environment will break and be irrecoverable. If you need assistance with an Arc resource bridge upgrade, please contact Microsoft Support.

You can manually upgrade the Arc resource bridge from your management machine. Before upgrading, make sure you meet all prerequisites. The management machine must have the kubeconfig locally stored. 

Manual upgrade generally takes between 30-90 minutes, depending on network speeds. The upgrade command takes your Arc resource bridge to the next appliance version, which might not be the latest available appliance version. Multiple upgrades may be needed to reach a [supported version](#supported-versions).

1. Before upgrading, you need the latest Azure CLI extension for `arcappliance`:
```azurecli
az extension add --upgrade --name arcappliance 
```
2. To run an upgrade, you need the kubeconfig, which can be retrieved with the following [get-credentials command](/cli/azure/arcappliance#az-arcappliance-get-credentials):
```azurecli
az arcappliance get-credentials --resource-group [REQUIRED] --name [REQUIRED] --credentials-dir [OPTIONAL]
```
3. (Arc-enabled VMware) To [upgrade a resource bridge on VMware](/cli/azure/arcappliance/upgrade#az-arcappliance-upgrade-vmware) without the configuration file, you must provide the following required and optional parameters in lieu of the configuration file:
```
az arcappliance upgrade vmware --resource-group [REQUIRED] --name [REQUIRED] --kubeconfig [REQUIRED] --address [OPTIONAL] --username [OPTIONAL] --password [OPTIONAL]
```

(Arc-enabled SCVMM) To [upgrade on SCVMM](/cli/azure/arcappliance/upgrade#az-arcappliance-upgrade-scvmm) without the configuration file,  you must provide the following optional parameters in lieu of the configuration file:
```
az arcappliance upgrade scvmm --address [OPTIONAL] --kubeconfig [OPTIONAL] --location [OPTIONAL] --name [OPTIONAL] --password [OPTIONAL] --resource-group [OPTIONAL] --username [OPTIONAL]
```

To upgrade a resource bridge on Azure Local, transition to 23H2 and use the built-in upgrade management tool. For more information, see [About updates for Azure Local, version 23H2](/azure/azure-local/update/about-updates-23h2).

## Version releases

The Arc resource bridge version is tied to the versions of underlying components used in the appliance image, such as the Kubernetes version. When there's a change in the appliance image, the Arc resource bridge version gets incremented. This generally happens when a new `az arcappliance` CLI extension version is released. For detailed release info, see the [Arc resource bridge release notes](release-notes.md).

## Supported versions

We generally recommend keeping your Arc resource bridge on a version released within the last 6 months or within the latest n-3 versions, whichever is more recent. We recommend performing manual upgrades every 6 months refresh critical certificates within the appliance and ensure compliance with the [supported version policy](overview.md#supported-versions).  Support covers the current release and the three before it. Even if your version is supported, upgrade at least every six months. This is to ensure the internal components and certificates are refreshed. You can check your appliance version and the version release date for an estimate on the last upgrade date and ensure an upgrade has been done at least once every 6 months. 

When a patch version is released, the upgrade path may skip the minor version and directly upgrade to the patch version. In such cases, the supported versions (n-3) excludes the skipped minor version and includes the patch version instead. To see what versions are in support, please refer to [Arc resource bridge release notes](release-notes.md).

If a resource bridge isn't upgraded to one of the supported versions (n-3), it falls outside the support window and will be unsupported. It might not always be possible to upgrade an unsupported resource bridge to a newer version, as component services used by Arc resource bridge may no longer be compatible. In addition, the unsupported resource bridge might not be able to provide reliable monitoring and health metrics. If you can’t upgrade your Arc bridge to a supported version, contact Microsoft Support.

## Notification and upgrade availability

If your Arc resource bridge is at version n-3, you might receive an email notification letting you know that your resource bridge will be out of support once the next version is released. If you receive this notification, upgrade the resource bridge as soon as possible to allow debug time for any issues with manual upgrade.

To check if your Arc resource bridge has an upgrade available, run the command:

```azurecli
az arcappliance get-upgrades --resource-group [REQUIRED] --name [REQUIRED] 
```

To see the current version of an Arc resource bridge appliance, check the Azure resource of your Arc resource bridge.

## Next steps

- Learn about [Arc resource bridge maintenance operations](maintenance.md).
- Learn about [troubleshooting Arc resource bridge](troubleshoot-resource-bridge.md).

