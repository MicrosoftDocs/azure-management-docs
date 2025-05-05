---
title: Upgrade Arc resource bridge
description: Learn how to upgrade Arc resource bridge using either cloud-managed upgrade or manual upgrade.  
ms.date: 10/03/2024
ms.topic: how-to
---

# Upgrade Arc resource bridge

This article describes how Arc resource bridge is upgraded, and the two ways upgrade can be performed: cloud-managed upgrade or manual upgrade. Currently, some private cloud providers differ in how they handle Arc resource bridge upgrades.

## Private cloud providers

Currently, private cloud providers differ in how they perform Arc resource bridge upgrades. Review the following information to see how to upgrade your Arc resource bridge for a specific provider.

For **Arc-enabled VMware vSphere**, manual upgrade and cloud-managed upgrade are available. Appliances on version 1.0.15 and higher are automatically opted-in to cloud-managed upgrade. Cloud-managed upgrade helps ensure the appliance VM is kept within n-3 supported versions but not the latest version. [Upgrade prerequisites](#prerequisites) must be met. Microsoft may attempt to perform a cloud-managed upgrade of your Arc resource bridge at any time if your appliance will soon be out of support. While Microsoft offers cloud-managed upgrade, you’re still responsible for ensuring that your Arc resource bridge is within the supported n-3 versions and upgraded once every 6 months. Disruptions could cause cloud-managed upgrade to fail and you may need to manual upgrade the Arc resource bridge. If your Arc resource bridge is close to being out of support, we recommend a manual upgrade to make sure you maintain a supported version, rather than waiting for cloud-managed upgrade. If there are continuous problems attempting cloud-managed upgrade, your Arc resource bridge may fall out of support and you are responsible for manually upgrading to reach a supported version.

For **Azure Arc VM management on Azure Local**, appliance version 1.0.15 or higher is only available on Azure Local build 23H2. In Azure Local 23H2, the LCM tool manages upgrades across all Azure Local, Arc resource bridge, and extension components as a "validated recipe" package. Any preview version of Arc resource bridge must be removed before updating from 22H2 to 23H2. Attempting to upgrade Arc resource bridge independent of other Azure Local environment components may cause problems in your environment that could result in a disaster recovery scenario. For more information, see [About updates for Azure Local](/azure/azure-local/update/about-updates-23h2).

For **Arc-enabled System Center Virtual Machine Manager (SCVMM)**, the manual upgrade feature is available for appliance version 1.0.15 and higher. Appliances running a version lower than 1.0.15 need to perform the recovery option to get to version 1.0.15 or higher. Review the steps for [performing the recovery operation](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery). This deploys a new resource bridge and reconnects pre-existing Azure resources.

## Prerequisites

Before an Arc resource bridge can be upgraded, the following prerequisites must be met:

- Arc resource bridge must be online and healthy with a status of `Running`. You can check the Azure resource of your Arc resource bridge to verify.

- The [credentials in the appliance VM](maintenance.md#update-credentials-in-the-appliance-vm) must be valid. To test the credentials, perform an operation on an Arc-enabled VM from Azure.

- Arc resource bridge must be in the same location path where it was originally deployed.

- The appliance VM needs 35 GB of free space.

- For Arc-enabled VMware, upgrading the resource bridge requires 200 GB of free space on the datastore. A new template is also created.

- (Manual upgrade only) When performing a manual upgrade, run the upgrade command from the management machine used to initially deploy the Arc resource bridge. The [appliance configuration files](system-requirements.md#configuration-files) initially created at deployment are also needed. You can also run the upgrade command from a different machine that meets the [management machine requirements](system-requirements.md#management-machine-requirements).

- (Manual upgrade only) The management machine needs 3.5 GB of free space.

## Overview

The upgrade process deploys a new resource bridge using the reserved appliance VM IP (`k8snodeippoolend` IP, VM IP 2). Once the new resource bridge is up, it becomes the active resource bridge. The old resource bridge is deleted, and its appliance VM IP (`k8dsnodeippoolstart`, VM IP 1) becomes the new reserved appliance VM IP that will be used in the next upgrade.

Deploying a new resource bridge is a process consisting of several steps: downloading the appliance image (~3.5 GB) from the cloud, using the image to deploy a new appliance VM, verifying the new resource bridge is running, connecting it to Azure, deleting the old appliance VM, and reserving the old IP to be used for a future upgrade.

Overall, the upgrade generally takes at least 30 minutes, depending on network speeds. A short intermittent downtime might happen during the handoff between the old Arc resource bridge to the new Arc resource bridge. Additional downtime can occur if prerequisites aren't met, or if a change in the network (DNS, firewall, proxy, etc.) impacts the Arc resource bridge's network connectivity.

There are two ways to upgrade Arc resource bridge: cloud-managed upgrades managed by Microsoft, or manual upgrades where Azure CLI commands are performed by an admin.

## Cloud-managed upgrade

Arc resource bridges on a supported [private cloud provider](#private-cloud-providers) with an appliance version 1.0.15 or higher are automatically opted into cloud-managed upgrade. With cloud-managed upgrade, Microsoft may attempt to upgrade your Arc resource bridge at any time if it is on an appliance version that will soon be out of support. The upgrade prerequisites must be met for cloud-managed upgrade to work. While Microsoft offers cloud-managed upgrade, you’re still responsible for checking that your resource bridge is healthy, online, in a "Running" status, and within the supported n-3 versions. Disruptions could cause cloud-managed upgrades to fail. If your Arc resource bridge is close to being out of support, we recommend a manual upgrade to make sure you maintain a supported version, rather than waiting for cloud-managed upgrade.

To check your resource bridge status and the appliance version, run the `az arcappliance show` command from your management machine or check the version of your Arc resource bridge Azure resource and ensure the status is **Running**. If your appliance VM isn't in a healthy, Running state, cloud-managed upgrade might fail.

Cloud-managed upgrades are handled through Azure. A notification is pushed to Azure to reflect the state of the appliance VM as it upgrades. As the resource bridge progresses through the upgrade, its status might switch back and forth between different upgrade steps. Upgrade is complete when the appliance VM `status` is `Running` and `provisioningState` is `Succeeded`.  

To check the status of a cloud-managed upgrade, run the following Azure CLI command from the management machine:  

```azurecli
az arcappliance show --resource-group [REQUIRED] --name [REQUIRED] 
```

## Manual upgrade

> [!WARNING] 
> For Azure Local, you must use the built-in LCM tool to upgrade Arc resource bridge. If you attempt to manual upgrade using the Azure CLI command, your environment will break and be irrecoverable. If you need assistance with an Arc resource bridge upgrade, please contact Microsoft Support.

Arc resource bridge can be manually upgraded from the management machine. You must meet all upgrade prerequisites before attempting to upgrade. The management machine must have the kubeconfig and [appliance configuration files](system-requirements.md#configuration-files) stored locally, or you won't be able to run the upgrade.

Manual upgrade generally takes between 30-90 minutes, depending on network speeds. The upgrade command takes your Arc resource bridge to the next appliance version, which might not be the latest available appliance version. Multiple upgrades could be needed to reach a [supported version](#supported-versions). You can check your appliance version by checking the Azure resource of your Arc resource bridge.

Before upgrading, you need the latest Azure CLI extension for `arcappliance`:

```azurecli
az extension add --upgrade --name arcappliance 
```

To manually upgrade your resource bridge, use the following command:

```azurecli
az arcappliance upgrade <private cloud> --config-file <file path to ARBname-appliance.yaml> 
```

For example, to upgrade a resource bridge on VMware, run: `az arcappliance upgrade vmware --config-file c:\contosoARB01-appliance.yaml`

To upgrade a resource bridge on SCVMM, run: `az arcappliance upgrade scvmm --config-file c:\contosoARB01-appliance.yaml`

To upgrade a resource bridge on Azure Local, transition to 23H2 and use the built-in upgrade management tool. For more information, see [About updates for Azure Local, version 23H2](/azure/azure-local/update/about-updates-23h2).

## Version releases

The Arc resource bridge version is tied to the versions of underlying components used in the appliance image, such as the Kubernetes version. When there's a change in the appliance image, the Arc resource bridge version gets incremented. This generally happens when a new `az arcappliance` CLI extension version is released. For detailed release info, see the [Arc resource bridge release notes](release-notes.md).

## Supported versions

We generally recommend using the most recent versions. The version support policy for the appliance generally covers the most recent version and the three previous versions (n-3). Even if a version is within the version support policy (n-3), the appliance should be upgraded at least once every 6 months to ensure the internal components and certificates are refreshed. You can check your appliance version and the version release date for an estimate on the last upgrade date and ensure an upgrade has been done at least once every 6 months. When a patch version is released, the upgrade path may skip the minor version and directly upgrade to the patch version. In such cases, the supported versions (n-3) excludes the skipped minor version and includes the patch version instead.

To see exactly what versions are in support, please refer to [Arc resource bridge release notes](release-notes.md).

If a resource bridge isn't upgraded to one of the supported versions (n-3), it falls outside the support window and will be unsupported. It might not always be possible to upgrade an unsupported resource bridge to a newer version, as component services used by Arc resource bridge may no longer be compatible. In addition, the unsupported resource bridge might not be able to provide reliable monitoring and health metrics.

If an Arc resource bridge can't be upgraded to a supported version, please contact Microsoft Support to determine options.

## Notification and upgrade availability

If your Arc resource bridge is at version n-3, you might receive an email notification letting you know that your resource bridge will be out of support once the next version is released. If you receive this notification, upgrade the resource bridge as soon as possible to allow debug time for any issues with manual upgrade, or submit a support ticket if cloud-managed upgrade was unable to upgrade your resource bridge.

To check if your Arc resource bridge has an upgrade available, run the command:

```azurecli
az arcappliance get-upgrades --resource-group [REQUIRED] --name [REQUIRED] 
```

To see the current version of an Arc resource bridge appliance, run `az arcappliance show` or check the Azure resource of your Arc resource bridge.

## Next steps

- Learn about [Arc resource bridge maintenance operations](maintenance.md).
- Learn about [troubleshooting Arc resource bridge](troubleshoot-resource-bridge.md).

