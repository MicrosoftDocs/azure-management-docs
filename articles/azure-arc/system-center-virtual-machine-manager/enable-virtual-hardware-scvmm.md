---
title:  Enable VM CRUD and power cycle operational ability in an SCVMM managed Arc-enabled server machine
description: This article describes how to enable VM CRUD and power cycle operational ability on an SCVMM managed VM that has Arc agents installed via the Azure Arc-enabled Servers route.
ms.topic: how-to 
ms.date: 06/18/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
# Customer intent: As a systems administrator, I want to enable virtual hardware management and VM CRUD capabilities for SCVMM VMs with Arc agents, so that I can streamline operations and manage resources more effectively within my Azure environment.
---

# Enable VM CRUD and power cycle operational ability in an SCVMM managed Arc-enabled server machine

This article describes how to enable VM CRUD and power cycle operational ability on an SCVMM managed VM that has Arc agents installed via the Azure Arc-enabled Servers route.

>[!IMPORTANT]
>- This article is applicable only if you've installed Arc agents directly in SCVMM managed virtual machines **before** onboarding to Azure Arc-enabled SCVMM by deploying Azure Arc resource bridge. 
>- If you installed Arc agents directly in SCVMM managed virtual machines, **after** onboarding to Azure Arc-enabled SCVMM by deploying Azure Arc resource bridge, refer to this [troubleshooting guide](https://github.com/microsoft/AzureArcSCVMMTSG/blob/main/1_Unifed%20resource%20model.md) to link the machines to Azure Arc-enabled SCVMM and get the full range of Azure Arc's VM management capabilities. 

## Prerequisites

Before you enable VM CRUD and power cycle management on an SCVMM managed VM, ensure you meet these prerequisites:

- An Azure subscription and resource group where you have *Arc SCVMM VM Administrator* role. 
- Your SCVMM management server instance must be [onboarded](quickstart-connect-system-center-virtual-machine-manager-to-arc.md) to Azure Arc. The onboarded SCVMM management server must be in a *Connected* state and its associated Azure Arc resource bridge must be in a *Running* state. 

## Link Azure Arc-enabled server machines to Azure Arc-enabled SCVMM 

To enable the virtual machine lifecycle and power cycle management ability and self-service access, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to the **Virtual machines** inventory page of your SCVMM management servers. The virtual machines that have Arc agent installed via the Arc-enabled Servers route will have **Link to SCVMM** status under virtual hardware management.


     :::image type="content" source="media/enable-virtual-hardware-scvmm/virtual-machines.png" alt-text="Screenshot of Virtual machines screen." lightbox="media/enable-virtual-hardware-scvmm/virtual-machines.png":::

1. Select **Link to SCVMM** to view the pane with the list of all the machines under SCVMM management server with Arc agent installed but not linked to the SCVMM management server in Azure Arc.


     :::image type="content" source="media/enable-virtual-hardware-scvmm/link-to-scvmm.png" alt-text="Screenshot of Link to SCVMM screen." lightbox="media/enable-virtual-hardware-scvmm/link-to-scvmm.png":::

1. Choose all the machines that need to be enabled in Azure, and select **Link** to link the machines to the Azure Arc-enabled SCVMM management server.

1. After you perform the **Link to SCVMM** operation, the **virtual hardware management** status reflects as **Enabled** for all the VMs, and you can perform lifecycle and power cycle operations on them. 

## Next step

[Set up and manage self-service access to SCVMM resources](set-up-and-manage-self-service-access-scvmm.md).
