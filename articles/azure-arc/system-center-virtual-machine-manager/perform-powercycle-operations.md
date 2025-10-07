---
title:  Perform powercycle operations on SCVMM VMs in Azure
description: In this article, you learn how to perform power cycle operations such as start, stop and restart on the Azure Arc-enabled SCVMM Virtual Machines.  
ms.topic: how-to 
ms.date: 05/28/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
---

# Perform powercycle operations on SCVMM managed virtual machines in Azure

In this article, you learn how to perform power cycle operations such as start, stop and restart on the Azure Arc-enabled SCVMM Virtual Machines. 

## Prerequisites 

Before you perform power cycle operations on a VM, ensure that you meet the following prerequisites: 

- The SCVMM management server is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state. 
- The VM which will be operated from Azure is [enabled for management in Azure](enable-scvmm-inventory-resources.md). 
- *Azure Arc SCVMM VM Contributor* role or a custom Azure role with permissions to make any changes to the SCVMM VMs on which you want to perform the power operations. 

## Perform power cycle operations 

To perform power cycle operations, follow these steps: 

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **SCVMM management server** and then select the SCVMM server which manages the VM that you are planning to operate from Azure. 

2. Navigate to the dedicated **Virtual machines** inventory view under the SCVMM inventory. Alternatively, you can navigate to the inventory view for VMs enabled for management in Azure from **Azure Arc** > **Machines** blade.

3. Select the machine on which you want to perform power cycle operations. 

   You can now perform the following power cycle operations: 

    - **Start**: Select **Start** > **Yes** to start the machine. 
    - **Restart**: Select **Restart** > **Yes** to restart the machine. 
    - **Stop**: Select **Stop** > **Yes** to stop the machine. 
    
   :::image type="content" source="media/perform-powercycle-operations/power-cycle-operations.png" alt-text="Screenshot showing power cycle operations screen.":::

You can track the progress of the Azure operations from the Azure [activity log](https://ms.portal.azure.com/#view/Microsoft_Azure_ActivityLog/ActivityLogBlade). 
