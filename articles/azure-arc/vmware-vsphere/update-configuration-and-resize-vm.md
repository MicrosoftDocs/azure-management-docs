---
title: Update configuration and resize a VMware vCenter managed virtual machine in Azure
description: In this article, you learn how to update the configuration and resize a VMware vCenter-managed VM in Azure through Azure Arc-enabled VMware vSphere. 
ms.topic: how-to 
ms.date: 07/18/2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
---

# Update configuration and resize a VMware vCenter managed virtual machine in Azure 

In this article, you learn how to update the configuration and resize a VMware vCenter-managed VM in Azure through Azure Arc-enabled VMware vSphere. 

From Azure, you can modify the following configuration of a VM:

-	[Add or remove network interfaces](#add-or-remove-nics-to-a-virtual-machine).
-	[Add or remove disks](#add-or-remove-disks-to-a-virtual-machine).
-	[Resize the VM by changing the CPU vCores, memory size, and type](#resize-a-virtual-machine). 

## Prerequisites

Before you update and resize a virtual machine, ensure that you meet the following prerequisites: 
 
-	The VMware vCenter is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
-	Ensure the VM which will be modified from Azure is [enabled for management in Azure](browse-and-enable-vcenter-resources-in-azure.md).
-	If you are modifying the Disk settings, ensure the VM which will be modified from Azure is in a *Stopped* state.
-	*Azure Arc VMware VM Contributor* role or a custom Azure role with permissions to make changes to the VMware vSphere VM on which you want to perform update and resize operations.

## Add or remove NICs to a virtual machine

To add or remove NICs to a VM, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenter** and then select the VMware vCenter which manages the VM which you are planning to modify from Azure.
2. Navigate to the dedicated **Virtual machines** inventory view under the vCenter inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.
3. Select the machine for which you want to update the NICs and then select **Settings** > **Networking**:

   - **Add a network interface**: Select **Add network interface** and enter the mandatory fields such as Name, VM network, and Adapter settings, and check Enable power on at boot checkbox. Select **Add** and then **Save** to apply the changes.

     :::image type="content" source="media/update-configuration-and-resize-vm/add-network-interface.png" alt-text="Screenshot showing add network interface screen." lightbox="media/update-configuration-and-resize-vm/add-network-interface.png":::

   - **Remove a network interface**: Choose the network interface to be removed and select **Delete** > **Yes**.

## Add or remove disks to a virtual machine

To add or remove disks to a VM, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenter** and then select the VMware vCenters which manages the VM which you are planning to modify from Azure.
2. Navigate to the dedicated **Virtual machines** inventory view under the vCenter inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.
3. Select the machine for which you want to add or remove a disk and then select **Settings** > **Disks**:

   - **Add a disk**: Select **Add disk** and enter the mandatory fields such as Disk name, size, controller and mode. Select **Add** and then **Apply** to apply the changes.
   
     :::image type="content" source="media/update-configuration-and-resize-vm/add-a-disk.png" alt-text="Screenshot showing add a disk screen." lightbox="media/update-configuration-and-resize-vm/add-a-disk.png":::

   - **Remove a disk**: Choose the disk to be removed and select **Delete** > **Yes**.

## Resize a virtual machine

To resize a VM, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenter** and then select the VMware vCenter which manages the VM which you are planning to modify from Azure.
2. Navigate to the dedicated **Virtual machines** inventory view under the vCenter inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.
3. Select the machine that you want to resize and then select **Settings** > **Size**. You can change the number of CPU cores, and memory size. 
   
4. Select **Save** to apply the changes.

   :::image type="content" source="media/update-configuration-and-resize-vm/size.png" alt-text="Screenshot showing size screen." lightbox="media/update-configuration-and-resize-vm/size.png":::

## Next step

- [Create a virtual machine](create-virtual-machine.md).
- [Perform VM powercycle operations](perform-powercycle-operations.md).

