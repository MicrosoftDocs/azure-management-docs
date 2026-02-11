---
title: Enable additional capabilities on Arc-enabled Server machines by linking to vCenter
description: Enable additional capabilities on Arc-enabled Server machines by linking to vCenter.
ms.topic: how-to
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - devx-track-azurecli
  - build-2025
  - sfi-image-nochange
# Customer intent: "As an IT administrator managing VMware machines, I want to link Arc-enabled Server machines to vCenter, so that I can enable additional virtual machine lifecycle and power cycle operations for efficient management."
---

# Enable additional capabilities on Arc-enabled Server machines by linking to vCenter

If you have VMware machines connected to Azure as Arc-enabled Servers, you can get extra capabilities by deploying the resource bridge and connecting vCenter to Azure. These extra capabilities include the ability to perform virtual machine lifecycle operations, such as create, resize, and power cycle operations like start, stop, and more. You get these extra capabilities without any disruption, and you keep the VM extensions configured on the Arc-enabled Server machines.   

Follow the steps in [Quickstart: Connect vCenter to Azure Arc using script](./quick-start-connect-vcenter-to-arc-using-script.md) to deploy the Arc Resource Bridge and connect vCenter to Azure.

>[!IMPORTANT]
> This article applies only if you directly installed Arc agents on the VMware machines and onboarded those machines as *Microsoft.HybridCompute/machines* ARM resources before connecting vCenter to Azure by deploying Resource Bridge. 

## Prerequisites

- An Azure subscription and resource group where you have *Azure Arc VMware Administrator role*. 
- Your vCenter instance must be [onboarded](quick-start-connect-vcenter-to-arc-using-script.md) to Azure Arc.
- Arc-enabled Servers machines and vCenter resource must be in the same Azure region.

## Link Arc-enabled Servers machines to vCenter from Azure portal

1. Go to the Virtual machines inventory page for your vCenter in the Azure portal. 

1. Virtual machines that have the Arc agent installed through the Arc-enabled Servers route show the **Link to vCenter** status under virtual hardware management.  

1. Select **Link to vCenter** to open a pane that lists all the machines under vCenter with the Arc agent installed but not linked to vCenter in Azure Arc.  

1. Select all the machines and select the option to link machines to vCenter.

    :::image type="content" source="media/enable-virtual-hardware/link-machine-to-vcenter.png" alt-text="Screenshot that shows the Link to vCenter page." lightbox="media/enable-virtual-hardware/link-machine-to-vcenter.png":::

1.	After linking to vCenter, the virtual hardware status shows as **Enabled** for all the VMs, and you can perform [virtual hardware operations](./perform-vm-ops-through-azure.md). 

    :::image type="content" source="media/enable-virtual-hardware/perform-virtual-hardware-operations.png" alt-text="Screenshot that shows the page for performing virtual hardware operations." lightbox="media/enable-virtual-hardware/perform-virtual-hardware-operations.png":::

    After linking to vCenter, virtual lifecycle operations and power cycle operations are enabled on the machines, and the kind property of Hybrid Compute Machine is updated as VMware.

## Link Arc-enabled Server machines to vCenter using Azure CLI

Use the following az commands to link Arc-enabled Server machines to vCenter at scale.  

**Create VMware resource from the specified Arc for Server machine in the vCenter** 

[!INCLUDE [azure-cli-specified-arc](./includes/azure-cli-specified-arc.md)]

**Create VMware resources from all Arc for Server machines in the specified resource group belonging to that vCenter**

[!INCLUDE [azure-cli-all](./includes/azure-cli-all.md)]

**Create VMware resources from all Arc for Server machines in the specified subscription belonging to that vCenter**

[!INCLUDE [azure-cli-subscription](./includes/azure-cli-subscription.md)]

### Required parameters 

**--vcenter-id -v**

ARM ID of the vCenter to which you link the machines. 

### Optional parameters 

**--ids**

One or more resource IDs, separated by spaces. It must be a complete resource ID containing all the information of *Resource Id* arguments. Provide either *--ids* or other *Resource Id* arguments. 

**--name -n**

Name of the Microsoft.HybridCompute Machine resource. Provide this parameter if you want to convert a single machine to a VMware VM. 

**--resource-group -g**

Name of the resource group to scan for HCRP machines. 

>[!NOTE]
>The default group configured by using `az configure --defaults group=` isn't used. You must specify this parameter explicitly.

**--subscription**

Name or ID of subscription. You can configure the default subscription by using `az account set -s NAME_OR_ID`. 

#### Known issue
 
During the first scan of the vCenter inventory after onboarding to Azure Arc-enabled VMware vSphere, Arc-enabled Servers machines are discovered under vCenter inventory. If the Arc-enabled Server machines aren't discovered and you try to perform the **Enable in Azure** operation, you encounter the following error:<br>

*A machine '/subscriptions/XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXXXX/resourceGroups/rg-contoso/providers/Microsoft.HybridCompute/machines/testVM1' already exists with the specified virtual machine MoRefId: 'vm-4441'. The existing machine resource can be extended with private cloud capabilities by creating the VirtualMachineInstance resource under it.*

When you encounter this error message, you can perform the **Link to vCenter** operation in 10 minutes. Alternatively, you can use any of the Azure CLI commands listed earlier to link an existing Arc-enabled Server machine to vCenter.

## Next steps

[Set up and manage self-service access to VMware resources through Azure RBAC](setup-and-manage-self-service-access.md).
