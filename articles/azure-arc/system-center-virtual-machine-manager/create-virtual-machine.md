---
title:  Create a virtual machine on System Center Virtual Machine Manager using Azure Arc
description: This article helps you create a virtual machine using Azure portal. 
ms.date: 02/09/2026
ms.topic: how-to
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
keywords: "VMM, Arc, Azure"
ms.custom:
  - build-2025
# Customer intent: "As a cloud administrator, I want to create a new virtual machine in Azure using System Center Virtual Machine Manager, so that I can efficiently manage and deploy resources across my hybrid cloud environment."
---


# Create a virtual machine on System Center Virtual Machine Manager using Azure Arc

After your administrator connects an SCVMM management server to Azure, enables VMM resources such as VMM clouds, VM templates, and VM networks in Azure, and gives you the required permissions on those resources, you can create a new SCVMM managed virtual machine in Azure. 

In this article, you learn how to create a new SCVMM managed virtual machine from Azure Arc portal. To explore non-portal methods such as Azure CLI, PowerShell, REST APIs, SDKs, and Infrastructure-as-Code mechanisms, see the **Reference** section of this documentation.

## Prerequisites

- An Azure subscription and resource group where you have *Azure Arc SCVMM VM Contributor* role.
- A cloud resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.
- A virtual machine template resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.
- A virtual network resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.

## Create a VM in Azure portal

1. Go to Azure portal.
1. Start creating a new VM by using one of the following methods:
   - Select **Azure Arc** as the service. Under **Host environments**, select **SCVMM management servers**. Search for and select your SCVMM management server. Under **SCVMM inventory**, select **Virtual machines** and select **Add**. 

       :::image type="content" source="media/create-virtual-machine/add.png" alt-text="Screenshot of Add screen.":::

   Or
   - Select **Azure Arc** as the service. Under **Azure Arc resources**, select **Machine**. Select **Add/Create** and select **Create a machine in a connected host environment**.

       :::image type="content" source="media/create-virtual-machine/create-machine.png" alt-text="Screenshot of create a machine screen.":::

1. When **Create an Azure Arc virtual machine** opens, under **Basics** > **Project details**, select the **Subscription** and **Resource group** where you want to deploy the VM.
1. Under **Instance details**, enter the following information:
   - **Virtual machine name** - Specify the name of the virtual machine.
   - **Custom location** - Select the custom location that your administrator shares with you.
   - **Virtual machine kind** - Select **System Center Virtual Machine Manager**.
   - **Cloud** - Select the target VMM private cloud.
   - **Availability set** - (Optional) Use availability sets to identify virtual machines that you want VMM to keep on separate hosts for improved continuity of service.

       :::image type="content" source="media/create-virtual-machine/machines.png" alt-text="Screenshot of machines screen.":::

1. Under **Template details**, enter the following information:
   - **Template** - Choose the VM template for deployment.
   - **Override template defaults** - Select the checkbox to override the default CPU cores and memory on the VM templates.
   - Specify computer name for the VM if the VM template has computer name associated with it.

       :::image type="content" source="media/create-virtual-machine/template-details.png" alt-text="Screenshot of template details screen.":::

1. Keep the **Enable Guest Management** checkbox selected to automatically install Azure connected machine agent immediately after the creation of the VM. [Azure connected machine agent (Arc agent)](../servers/agent-overview.md) is required if you're planning to use Azure management services to govern, patch, monitor, and secure your VM through Azure.

1. Under **Administrator account**, enter the following information and select **Next : Disks >**.
   - Username
   - Password
   - Confirm password

    :::image type="content" source="media/create-virtual-machine/admin-account.png" alt-text="Screenshot of administrator account screen.":::

1. Under **Disks**, you can optionally change the disks configured in the template. You can add more disks or update existing disks.

    :::image type="content" source="media/create-virtual-machine/disks.png" alt-text="Screenshot of Disks tab screen.":::

1. Under **Networking**, you can optionally change the network interfaces configured in the template. You can add Network interface cards (NICs) or update the existing NICs. You can also change the network that this NIC attaches to, if you have appropriate permissions to the network resource.

    :::image type="content" source="media/create-virtual-machine/networking.png" alt-text="Screenshot of Networking tab screen.":::

1. Under **Advanced**, enable processor compatibility mode if required.

    :::image type="content" source="media/create-virtual-machine/advanced.png" alt-text="Screenshot of Advanced tab screen.":::

1. Under **Tags**, you can optionally add tags to the VM resource.

    :::image type="content" source="media/create-virtual-machine/tags.png" alt-text="Screenshot of Tags tab screen.":::

1. Under **Review + create**, review all the properties and select **Create**. The VM is created in a few minutes.

## Next steps

- [Set up and manage self-service access](set-up-and-manage-self-service-access-scvmm.md).
- [Perform VM powercycle operations](perform-powercycle-operations.md).
- [Update configuration and resize a VM](update-configuration-and-resize-vm.md).
