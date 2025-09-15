---
title: Switch to the new version
description: Learn how to switch to the new version of Azure Arc-enabled VMware vSphere and use its capabilities.
ms.topic: how-to 
ms.date: 03/13/2024
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
ms.custom:
  - build-2025
  - sfi-image-nochange
# Customer intent: As a VI admin, I want to switch to the new version of Arc-enabled VMware vSphere and leverage the associated capabilities.
---

# Switch to the new version

On August 21, 2023, we rolled out major changes to **Azure Arc-enabled VMware vSphere**. By switching to the new version, you can use all the Azure management services that are available for Arc-enabled Servers.

If you onboarded to Azure Arc-enabled VMware vSphere before **August 21, 2023**, and your VMs were Azure-enabled, you'll encounter the following breaking changes:

- For the VMs with Arc agents, starting from **February 27, 2024**, you'll no longer be able to perform any Azure management service-related operations.  
- From **April 1, 2024**, you'll no longer be able to perform any operations on the VMs, except the **Remove from Azure** operation. 

To continue using these machines, follow these instructions to switch to the new version.

> [!NOTE]
> If you're new to Arc-enabled VMware vSphere, you'll be able to leverage the new capabilities by default. To get started with the new version, see [Quickstart: Connect VMware vCenter Server to Azure Arc by using the helper script](quick-start-connect-vcenter-to-arc-using-script.md). 


## Switch to the new version (Existing customer)

If you onboarded to **Azure Arc-enabled VMware** before August 21, 2023, for VMs that are Azure-enabled, follow these steps to switch to the new version: 

>[!Note]
>If you had enabled guest management on any of the VMs, remove [VM extensions](/azure/azure-arc/vmware-vsphere/remove-vcenter-from-arc-vmware#step-1-remove-vm-extensions) and [disconnect agents](/azure/azure-arc/vmware-vsphere/remove-vcenter-from-arc-vmware#step-2-disconnect-the-agent-from-azure-arc).

1. From your browser, go to the vCenters blade on [Azure Arc Center](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/~/overview) and select the vCenter resource. 

2. Select all the virtual machines that are Azure enabled with the older version.  

3. Select **Remove from Azure**.  

    :::image type="VM Inventory view" source="media/switch-to-new-version/vm-inventory-view-inline.png" alt-text="Screenshot of VM Inventory view." lightbox="media/switch-to-new-version/vm-inventory-view-expanded.png":::

4. After successful removal from Azure, enable the same resources again in Azure.

5. Once the resources are re-enabled, the VMs are auto switched to the new version. The VM resources will now be represented as **Machine - Azure Arc (VMware)**.

    :::image type=" New VM browse view" source="media/switch-to-new-version/new-vm-browse-view-inline.png" alt-text="Screenshot of New VM browse view." lightbox="media/switch-to-new-version/new-vm-browse-view-expanded.png":::
 
## Next steps

[Create a virtual machine on VMware vCenter using Azure Arc](/azure/azure-arc/vmware-vsphere/quick-start-create-a-vm).