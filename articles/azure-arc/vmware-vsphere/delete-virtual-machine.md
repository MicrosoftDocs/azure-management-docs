---
title:  Delete a VMware vCenter-managed VM in Azure through Azure Arc-enabled VMware vSphere
description: In this article, you learn how to delete a VMware vCenter-managed virtual machine and its Azure resource through Azure Arc-enabled VMware vSphere.
ms.topic: how-to 
ms.date: 07/18/2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
---

# Delete a VMware vCenter-managed VM in Azure through Azure Arc-enabled VMware vSphere

In this article, you learn how to delete a VMware vCenter-managed virtual machine and its Azure resource through Azure Arc-enabled VMware vSphere.

## Prerequisites

Before you delete a virtual machine or remove its Azure resource, ensure that you meet the following prerequisites: 

- The VMware vCenter that manages the VM, which is to be deleted from the host or which’s Azure resource is to be removed, is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
- Ensure the VM, which is to be deleted from the host or which’s Azure resource is to be removed, is [enabled for management in Azure](browse-and-enable-vcenter-resources-in-azure.md).
- If the VM, which is to be deleted from the host or which’s Azure resource is to be deleted, has the Arc agent installed (guest management enabled), [uninstall the agent and remove any VM extensions](/azure/azure-arc/servers/manage-agent?toc=%2Fazure%2Fazure-arc%2Fvmware-vsphere%2Ftoc.json&tabs=windows#uninstall-the-agent) to prevent billing beyond the lifetime of the VM.
- *Azure Arc VMware VM Contributor* role or a custom Azure role with permissions to delete the VMware vSphere VMs which you want to be deleted.
	
## Delete a Virtual Machine

>[!Important] 
>- This operation also deletes the VM on your VMware vCenter managed on-premises host. To remove the machine from Azure only and keep the on-premises resources intact, perform the [Remove from Azure](#remove-a-virtual-machine-from-azure-only) instead.
>- Before you delete a VM, ensure all the critical data is backed up, the VM owner is informed and all dependencies and services regarding the VM are considered. 

To delete a VM, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenters** and then select the VMware vCenter, which manages the VM, which you're planning to delete from Azure.
2. Navigate to the dedicated **Virtual machines** inventory view under the vCenter inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.
3. Select the machine, which you want to delete and then select **Delete**.
 
   :::image type="content" source="media/delete-virtual-machine/delete-virtual-machine.png" alt-text="Screenshot showing delete VM option." lightbox="media/delete-virtual-machine/delete-virtual-machine.png":::

   When prompted, confirm that you want to delete it.
 
    :::image type="content" source="media/delete-virtual-machine/delete.png" alt-text="Screenshot showing Delete screen." lightbox="media/delete-virtual-machine/delete.png":::

## Remove a Virtual Machine from Azure only

To remove a VM from Azure only, follow these steps: 

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenters** and then select the VMware vCenters which manages the VM, which you're planning to operate from Azure. 
2. Navigate to the dedicated **Virtual machines** inventory view under the vCenter inventory. Select the machine for which you want to remove the Azure representation and then select **Remove from Azure**.

   :::image type="content" source="media/delete-virtual-machine/remove-from-azure.png" alt-text="Screenshot showing Virtual machines screen." lightbox="media/delete-virtual-machine/remove-from-azure.png":::

   When prompted, confirm that you want to remove the Azure representation of the VM.

You can track the progress of the Azure operations from the Azure [activity log](https://ms.portal.azure.com/#view/Microsoft_Azure_ActivityLog/ActivityLogBlade).

## Next step

- [Create a Virtual machine](create-virtual-machine.md).
- [Remove vCenter environment](remove-vcenter-from-arc-vmware.md).