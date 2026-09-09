---
title: Upgrade Arc resource bridge
description: Learn how to upgrade Arc resource bridge using either cloud-managed upgrade or manual upgrade.
ms.date: 09/09/2026
ms.topic: how-to
# Customer intent: "As a cloud administrator, I want to upgrade the Arc resource bridge using either cloud-managed or manual methods, so that I can maintain supported versions and ensure the appliance remains operational and compliant with upgrade requirements."
---

# Upgrade Arc resource bridge

This article describes how to upgrade the Arc resource bridge and the two ways you can perform the upgrade: cloud-managed upgrade or manual upgrade. Currently, some private cloud providers differ in how they handle Arc resource bridge upgrades.

## Private cloud providers

Private cloud providers have different support policies and upgrade procedures for the Arc resource bridge. Review the sections in this article to learn how to upgrade your Arc resource bridge for your private cloud.

### Arc-enabled VMware vSphere

For **Arc-enabled VMware vSphere**, you're responsible for upgrading your Azure Arc resource bridge to a version released within the past six months. The appliance version should also be within the most recent three versions released. We recommend performing manual upgrades every six months to refresh critical certificates within the appliance. If these certificates expire or the appliance is offline for longer than 45 days, your resource bridge might need to be [recovered](/azure/azure-arc/vmware-vsphere/recover-from-resource-bridge-deletion). To find the version release date, see the [Arc resource bridge release notes](release-notes.md).

For Arc-enabled VMware, Microsoft might offer cloud-managed upgrades as a supplementary service. Arc resource bridges on version 1.0.15 or higher are automatically opted into supplemental cloud-managed upgrades. Supplemental cloud-managed upgrades don't replace the need for you to manually upgrade at least once every six months. You should ensure that prerequisites are met for a supplemental cloud-managed upgrade to succeed. Microsoft might attempt to upgrade your Arc resource bridge at any time. If your appliance isn't healthy, supplemental cloud-managed upgrades might fail. They also might not succeed due to network disruptions or errors. You're primarily responsible for ensuring your resource bridge is on a supported version with regular manual upgrades. If your appliance is nearing the end of its supported version, perform a manual upgrade to avoid service disruptions.

### Azure Local

For **Azure Arc VM management on Azure Local**, appliance version 1.0.15 or higher is available only on Azure Local build 23H2. In this version, use the built-in LCM tool to manage upgrades for Azure Local, Arc resource bridge, and extensions as a single package. Remove any preview version of Arc resource bridge before updating from 22H2 to 23H2. Don't upgrade Arc resource bridge separately from other Azure Local components, as this action can cause critical issues. For details, see [About updates for Azure Local](/azure/azure-local/update/about-updates-23h2).

### Azure Arc-enabled SCVMM

For **Arc-enabled System Center Virtual Machine Manager (SCVMM)**, you're responsible for upgrading your Azure Arc resource bridge to a version released within the past six months. The appliance version should also be within the most recent three versions released. We recommend performing manual upgrades at least once every six months to refresh critical certificates within the appliance. If these certificates expire or the appliance is offline for longer than 45 days, you might need to [perform recovery of the resource bridge](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery). 

To find the version release date, refer to the [Arc resource bridge release notes](release-notes.md). Manual upgrade is available for appliance version 1.0.15 and higher. Appliances running a version lower than 1.0.15 need to [perform the recovery option](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery) to get to version 1.0.15 or higher.

## Overview

The upgrade process deploys a new resource bridge by using the reserved appliance virtual machine (VM) IP. When the new resource bridge is ready, it becomes active. The upgrade process deletes the old resource bridge and reserves its VM IP for the next upgrade.

The upgrade process consists of the following actions:

- Download the appliance image (~3.5 GB) from the cloud.
- Use the image to deploy a new appliance VM.
- Verify that the new resource bridge is running and connect it to Azure.
- Delete the old appliance VM.
- Reserve the old IP for a future upgrade.

The upgrade usually takes at least 30 minutes, depending on network speed. Expect a brief downtime during the transition from the old resource bridge to the new one. More downtime might occur if prerequisites aren't met or if there are network issues.

## Prerequisites

Before you upgrade an Arc resource bridge, ensure you meet the following prerequisites:

- Arc resource bridge is online and healthy with a status of `Running`. Check the Azure resource of your Arc resource bridge to verify.
- The [credentials in the appliance VM](maintenance.md#update-credentials-in-the-appliance-vm) are valid. To test the credentials, perform an operation on an Arc-enabled private cloud VM from the Azure portal.
- Arc resource bridge is in the same location path where you originally deployed it.
- The appliance VM has 35 GB of free space.
- For Arc-enabled VMware, upgrading the resource bridge requires 200 GB of free space on the datastore. A new template is also created.
- (Manual upgrade only) When you perform a manual upgrade, run the upgrade command from the management machine you used to initially deploy the Arc resource bridge. You can also run the upgrade command from a different machine that meets the [management machine requirements](system-requirements.md#management-machine-requirements).
- (Manual upgrade only) The management machine has 3.5 GB of free space.

## Check the version

To check the appliance version of your Arc resource bridge, check the Azure resource of your resource bridge in Azure Resource Manager. If the appliance status or provisioning state is "Upgrade Failed" or "Failed", an upgrade attempt might have failed. If an upgrade fails, the appliance version shown might not reflect the actual version. The actual version is most likely the version prior to upgrading. The upgrade must succeed for your appliance to be on the new version.

## Manual upgrade

> [!WARNING]
> For Azure Local, you must use the built-in Azure Local LCM tool to upgrade Arc resource bridge. If you attempt a manual upgrade by using the Azure CLI command, your environment breaks and is irrecoverable. If you need assistance with an Arc resource bridge upgrade, contact Microsoft Support.

You can manually upgrade the Arc resource bridge from your management machine. Before upgrading, ensure you meet all prerequisites. The management machine must have the kubeconfig stored locally. Manual upgrade generally takes about 30-90 minutes, depending on your network speeds. An upgrade takes your Arc resource bridge to the next appliance version, which might not be the newest appliance version. To reach a supported version, you might need multiple upgrades. During an upgrade, the status of your resource bridge might change based on its progress. Upgrade is complete when the appliance status is `Running` and `provisioningState` is `Succeeded`. You can view the status by going to your Azure resource in the Azure portal.

### Perform a manual upgrade

1. Before upgrading, get the latest Azure CLI extension for `arcappliance`:

    ```azurecli
    az extension add --upgrade --name arcappliance
    ```

1. The appliance kubeconfig is required to run a manual upgrade. By default, it's stored in the management machine's CLI directory used to deploy the resource bridge. You can also retrieve it by using the following [get-credentials command](/cli/azure/arcappliance#az-arcappliance-get-credentials):

    ```azurecli
    az arcappliance get-credentials \
      --resource-group [REQUIRED] \
      --name [REQUIRED] \
      --credentials-dir [OPTIONAL]
    ```

1. (Arc-enabled VMware) To [upgrade a resource bridge on VMware](/cli/azure/arcappliance/upgrade#az-arcappliance-upgrade-vmware) without the configuration file, provide the following required and optional parameters instead of the configuration file:

    ```azurecli
    az arcappliance upgrade vmware \
      --resource-group [REQUIRED] \
      --name [REQUIRED] \
      --kubeconfig [REQUIRED] \
      --address [OPTIONAL] \
      --username [OPTIONAL] \
      --password [OPTIONAL]
    ```

1. (Arc-enabled SCVMM) To [upgrade on SCVMM](/cli/azure/arcappliance/upgrade#az-arcappliance-upgrade-scvmm) without the configuration file, provide the following optional parameters instead of the configuration file:

    ```azurecli
    az arcappliance upgrade scvmm \
      --address [OPTIONAL] \
      --kubeconfig [OPTIONAL] \
      --location [OPTIONAL] \
      --name [OPTIONAL] \
      --password [OPTIONAL] \
      --resource-group [OPTIONAL] \
      --username [OPTIONAL]
    ```

(Azure Local) To upgrade a resource bridge on Azure Local, transition to 23H2 and use the built-in upgrade management tool. For more information, see [About updates for Azure Local, version 23H2](/azure/azure-local/update/about-updates-23h2).

## Upgrade guidance

Manually upgrade at least once every six months to refresh critical certificates within the appliance. Your appliance version should also be within the three most recently released versions. For more information, see the [supported version policy](overview.md#supported-versions).

When a patch version is released, the upgrade path might skip the minor version and directly upgrade to the patch version. In these cases, the supported versions (n-3) exclude the skipped minor version and include the patch version. To see what versions are in support, refer to the [Arc resource bridge release notes](release-notes.md).

If you don't upgrade a resource bridge to a supported version, it's unsupported. You might not be able to upgrade an unsupported resource bridge to a newer version because internal components might no longer be compatible. In addition, the unsupported resource bridge might not be able to provide reliable monitoring and health metrics. You might also stop receiving email notifications about your Arc resource bridge. If you can't upgrade your Arc resource bridge to a supported version, you might need to perform recovery.

## Notification and upgrade availability

You might receive an email notification about your Arc resource bridge being unsupported due to a new version release. If you receive this email, upgrade the resource bridge as soon as possible to allow time for troubleshooting any issues during manual upgrade. To see the current version of an Arc resource bridge, check the Azure resource of your Arc resource bridge. To check if your Arc resource bridge has an upgrade available, run the following command:

```azurecli
az arcappliance get-upgrades --resource-group [REQUIRED] --name [REQUIRED]
```

## Next steps

- Learn about [Arc resource bridge maintenance operations](maintenance.md).
- Learn about [troubleshooting Arc resource bridge](troubleshoot-resource-bridge.md).
