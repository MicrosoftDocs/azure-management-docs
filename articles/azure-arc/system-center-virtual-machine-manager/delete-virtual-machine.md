---
title:  Delete a SCVMM-managed VM in Azure through Azure Arc-enabled System Center Virtual Machine Manager
description: In this article, you learn how to delete a SCVMM-managed virtual machine and its Azure resource through Azure Arc enabled SCVMM.
ms.topic: how-to 
ms.date: 06/04/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
---

# Delete a SCVMM-managed VM in Azure through Azure Arc-enabled SCVMM

In this article, you learn how to delete a SCVMM-managed virtual machine and its Azure resource through Azure Arc-enabled SCVMM.

## Prerequisites

Before you delete a virtual machine or remove its Azure resource, ensure that you meet the following prerequisites: 

- The SCVMM management server, that manages the VM which is to be deleted from the host or which’s Azure resource is to be removed, is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
- Ensure the VM, which is to be deleted from the host or which’s Azure resource is to be removed, is [enabled for management in Azure](enable-scvmm-inventory-resources.md).
- If the VM, which is to be deleted from the host or which’s Azure resource is to be deleted, has the Arc agent installed (guest management enabled), [uninstall the agent and remove any VM extensions](/azure/azure-arc/servers/manage-agent?toc=%2Fazure%2Fazure-arc%2Fsystem-center-virtual-machine-manager%2Ftoc.json&tabs=windows#uninstall-the-agent) to prevent billing beyond the lifetime of the VM.
- *Azure Arc SCVMM VM Contributor* role or a custom Azure role with permissions to delete the SCVMM VMs which you want to be deleted.
	
## Delete a Virtual Machine

>[!Important] 
>- This operation also deletes the VM on your SCVMM managed on-premises host. To remove the machine from Azure only and keep the on-premises resources intact, perform the [Remove from Azure](#remove-a-virtual-machine-from-azure-only) instead.
>- Before you delete a VM, ensure all the critical data is backed up, the VM owner is informed and all dependencies and services regarding the VM are considered. 

To delete a VM, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **SCVMM management server** and then select the SCVMM server which manages the VM which you are planning to delete from Azure.
2. Navigate to the dedicated **Virtual machines** inventory view under the SCVMM inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.
3. Select the machine which you want to delete and then select **Delete**.
 
   :::image type="content" source="media/delete-virtual-machine/delete-virtual-machine.png" alt-text="Screenshot showing delete VM option." lightbox="media/delete-virtual-machine/delete-virtual-machine.png":::

   When prompted, confirm that you want to delete it.
 
    :::image type="content" source="media/delete-virtual-machine/delete.png" alt-text="Screenshot showing Delete screen." lightbox="media/delete-virtual-machine/delete.png":::

## Remove a Virtual Machine from Azure only

To remove a VM from Azure only, follow these steps: 

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **SCVMM management server** and then select the SCVMM server which manages the VM which you are planning to operate from Azure. 
2. Navigate to the dedicated **Virtual machines** inventory view under the SCVMM inventory. Select the machine for which you want to remove the Azure representation and then select **Remove from Azure**.

   :::image type="content" source="media/delete-virtual-machine/remove-from-azure.png" alt-text="Screenshot showing Virtual machines screen." lightbox="media/delete-virtual-machine/remove-from-azure.png":::

   When prompted, confirm that you want to remove the Azure representation of the VM.

You can track the progress of the Azure operations from the Azure [activity log](https://portal.azure.com/#view/Microsoft_Azure_ActivityLog/ActivityLogBlade).

## Next step

- [Create a Virtual machine](create-virtual-machine.md).
- [Remove SCVMM environment](remove-scvmm-from-azure-arc.md).