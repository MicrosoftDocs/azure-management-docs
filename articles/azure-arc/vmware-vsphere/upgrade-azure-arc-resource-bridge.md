---
title: Upgrade the Azure Arc resource bridge associated with VMware vSphere environment
description: This article describes how to upgrade the Azure Arc resource bridge associated with your VMware vSphere environment.
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
ms.topic: how-to 
ms.date: 07/09/2025
---

# Upgrade the Azure Arc resource bridge associated with VMware vSphere environment

This article describes how to upgrade the Azure Arc resource bridge associated with your VMware vSphere environment.

## Overview

Deploying a new resource bridge is a process consisting of several steps: downloading the resource bridge VM VHDX from the cloud, using the VHDX to deploy a new VM, verifying that new resource bridge VM is running, associating the existing resources to the new resource bridge VM, connecting it to Azure and deleting the old resource bridge VM. All these steps are executed sequentially after you trigger the upgrade.

The upgrade generally takes 30-90 minutes, depending on the network speeds. A short intermittent downtime might happen during the handoff between the old resource bridge to the new resource bridge. Additional downtime can occur if prerequisites aren't met, or if a change in the network (DNS, firewall, proxy, etc.) impacts the resource bridge's network connectivity.

The resource bridge associated with your VMware vSphere environment must be upgraded manually to remain in the latest version. Microsoft triggers the upgrade of your resource bridge only when it's about to fall out of support. 

## Versions and upgrade availability

Periodically, new versions of the Azure Arc resource bridge are released to include security and feature updates. Generally, the latest released version and the previous three versions (up to n-3) of resource bridge are supported. A resource bridge on an unsupported version must be upgraded or redeployed to be in a production support window to ensure the certificates in the resource bridge are still valid. Hence, it's highly recommended to maintain the resource bridge in the latest version and to perform upgrades every six months. To stay informed on releases, visit the [Azure Arc resource bridge release notes](/azure/azure-arc/resource-bridge/release-notes).

If your resource bridge is at version n-3, you might receive an email notification letting you know that your resource bridge will be out of support once the next version is released. If you receive this notification, upgrade the resource bridge as soon as possible to be in the latest version. If there are any issues with manual upgrade, submit a support ticket to receive assistance from Microsoft while you're still in a supported version.

To check the current version of your resource bridge, run `az arcappliance show` from your workstation machine, which has the configuration files stored locally or check the Azure resource of your resource bridge. 

To check if your resource bridge has an upgrade available, run the following command:  

```azurecli
az arcappliance get-upgrades --resource-group <ResourceGroup> --name <ARBName>
```

## Prerequisites

Before you upgrade a resource bridge, the following prerequisites must be met:

- The Azure Arc resource bridge must be online and healthy with a status of *Running*. You can check the [Azure resource of your resource bridge](https://portal.azure.com/#view/Microsoft_Azure_ArcCenterUX/ArcCenterMenuBlade/~/resourceBridges) to verify.  
- The credentials in the resource bridge VM must be valid. You can update the credentials by following the steps given in [this article](/azure/azure-arc/system-center-virtual-machine-manager/administer-arc-scvmm#update-the-scvmm-account-credentials-using-a-new-password-or-a-new-scvmm-account-after-onboarding). To test the credentials, perform an operation on an Azure-enabled VM from Azure.
- The resource bridge must be in the same location path where it was originally deployed.
- The datastore in which the resource bridge is deployed must have at least 200GB of free space. 
- The Azure resource of the resource bridge must not have any resource locks placed on it.
- The workstation machine from which the upgrade is triggered must have the `kubeconfig` and the three `.yaml` appliance configuration files initially created during deployment stored locally.  

## Upgrade the Azure Arc resource bridge

# [Managed upgrade](#tab/managed)

Microsoft offers cloud-managed upgrades as a supplementary service, but this doesn't replace the need for manual upgrades every six months. With cloud-managed upgrade, Microsoft attempts to upgrade the Arc resource bridge if it will soon be out of support. The upgrade trigger time will at the discretion of Microsoft. The upgrade prerequisites must be met for cloud-managed upgrade to work. Disruptions, errors, or unhealthy resource bridge state could cause cloud-managed upgrades to fail and since Microsoft has negligible control over the resource bridge, which runs in an on-premises environment, we recommend periodic manual upgrades to ensure a supported Azure Arc resource bridge version.  

Cloud-managed upgrades are handled through Azure. A notification is pushed to Azure to reflect the state of the resource bridge as it upgrades. As the resource bridge progresses through the upgrade, its status might switch back and forth between different upgrade steps. Upgrade is complete when the **resource bridge status** is *Running* and **provisioningState** is *Succeeded*. 

To check the status of a cloud-managed upgrade, run the following Azure CLI command from the management machine: 

```azurecli
az arcappliance show --resource-group <ResourceGroup> --name <ARBName>
```

# [Manual upgrade](#tab/manual)

The Azure Arc resource bridge can be manually upgraded, and the upgrade command takes your Azure Arc resource bridge to the immediate next version, which might not be the latest available version. Multiple upgrades could be needed to reach a [supported version](/azure/azure-arc/resource-bridge/release-notes). To manually upgrade your resource bridge, follow these steps:

1. Ensure that you have installed the latest `az arcappliance` CLI extension by running the following extension upgrade command from the vCenter server:
     ```azurecli
     az extension add --upgrade --name arcappliance
     ```
2. To manually upgrade your resource bridge, run the following command from workstation machine, which has the `kubeconfig` and the three `.yaml` appliance configuration files stored locally:

     ```azurecli
     az arcappliance upgrade vmware --config-file <file path to ARBname-appliance.yaml>  
     ```

Once the upgrade is successful, you can navigate to the [Azure resource of your resource bridge](https://portal.azure.com/#view/Microsoft_Azure_ArcCenterUX/ArcCenterMenuBlade/~/resourceBridges) and verify if it is in a [supported version](/azure/azure-arc/resource-bridge/release-notes).

---

>[!NOTE]
>- It might not always be possible to upgrade an unsupported resource bridge to a newer version, as component services used by the Azure Arc resource bridge may no longer be compatible. In addition, the unsupported resource bridge might not be able to provide reliable monitoring and health metrics. 
>- If a resource bridge can't be upgraded to a supported version, you must perform a fresh deployment of a resource bridge and reassociate the existing resources with the new resource bridge. For more information, see [how to perform recovery operations on your resource bridge](/azure/azure-arc/system-center-virtual-machine-manager/disaster-recovery).
>- We recommend you to trigger the upgrade as soon as you receive the email notification to allow debugging time if there are any issues. To receive assistance from Microsoft, submit a support ticket while you're still in a supported version.

## Next Step

- [Troubleshoot SCVMM-specific upgrade errors](/azure/azure-arc/system-center-virtual-machine-manager/troubleshoot-scvmm).
- Learn how to [administer Arc-enabled SCVMM](/azure/azure-arc/system-center-virtual-machine-manager/administer-arc-scvmm).