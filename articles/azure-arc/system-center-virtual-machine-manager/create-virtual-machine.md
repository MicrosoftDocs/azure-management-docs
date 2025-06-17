---
title:  Create a virtual machine on System Center Virtual Machine Manager using Azure Arc
description: This article helps you create a virtual machine using Azure portal. 
ms.date: 06/10/2025
ms.topic: how-to
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
keywords: "VMM, Arc, Azure"
ms.custom:
  - build-2025
---


# Create a virtual machine on System Center Virtual Machine Manager using Azure Arc

Once your administrator has connected an SCVMM management server to Azure, enabled VMM resources such as VMM clouds, VM templates and VM networks in Azure, and provided you the required permissions on those resources, you'll be able to create a new SCVMM managed virtual machine in Azure. 

In this article, you'll learn how to create a new SCVMM managed virtual machine from Azure Arc portal. Refer to the **Reference** section of this documentation to explore non-portal methods such as Azure CLI, PowerShell, REST APIs, SDKs and Infrastructure-as-Code mechanisms.

## Prerequisites

- An Azure subscription and resource group where you have *Azure Arc SCVMM VM Contributor* role.
- A cloud resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.
- A virtual machine template resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.
- A virtual network resource on which you have *Azure Arc SCVMM Private Cloud Resource User* role.

## How to create a VM in Azure portal

1. Go to Azure portal.
2. You can initiate the creation of a new VM in either of the following two ways:
   - Select **Azure Arc** as the service and then select **SCVMM management servers** under **Host environments** from the left blade. Search and select your SCVMM management server. Select **Virtual machines** under **SCVMM inventory** from the left blade and select **Add**. 

       :::image type="content" source="media/create-virtual-machine/add.png" alt-text="Screenshot of Add screen.":::

   Or
   - Select **Azure Arc** as the service and then select **Machine** under **Azure Arc resources** from the left blade. Select **Add/Create** and select **Create a machine in a connected host environment** from the dropdown.

       :::image type="content" source="media/create-virtual-machine/create-machine.png" alt-text="Screenshot of create a machine screen.":::

1. Once the **Create an Azure Arc virtual machine** page opens, under **Basics** > **Project details**, select the **Subscription** and **Resource group** where you want to deploy the VM.
1. Under **Instance details**, provide the following details:
   - **Virtual machine name** - Specify the name of the virtual machine.
   - **Custom location** - Select the custom location that your administrator has shared with you.
   - **Virtual machine kind** - Select **System Center Virtual Machine Manager**.
   - **Cloud** - Select the target VMM private cloud.
   - **Availability set** - (Optional) Use availability sets to identify virtual machines that you want VMM to keep on separate hosts for improved continuity of service.

       :::image type="content" source="media/create-virtual-machine/machines.png" alt-text="Screenshot of machines screen.":::

1. Under **Template details**, provide the following details:
   - **Template** - Choose the VM template for deployment.
   - **Override template defaults** - Select the checkbox to override the default CPU cores and memory on the VM templates.
   - Specify computer name for the VM if the VM template has computer name associated with it.

       :::image type="content" source="media/create-virtual-machine/template-details.png" alt-text="Screenshot of template details screen.":::

1. Keep the **Enable Guest Management** checkbox selected to automatically install Azure connected machine agent immediately after the creation of the VM. [Azure connected machine agent (Arc agent)](../servers/agent-overview.md) is required if you're planning to use Azure management services to govern, patch, monitor, and secure your VM through Azure.

1. Under **Administrator account**, provide the following details and select **Next : Disks >**.
   - Username
   - Password
   - Confirm password

    :::image type="content" source="media/create-virtual-machine/admin-account.png" alt-text="Screenshot of administrator account screen.":::

1. Under **Disks**, you can optionally change the disks configured in the template. You can add more disks or update existing disks.

    :::image type="content" source="media/create-virtual-machine/disks.png" alt-text="Screenshot of Disks tab screen.":::

1. Under **Networking**, you can optionally change the network interfaces configured in the template. You can add Network interface cards (NICs) or update the existing NICs. You can also change the network that this NIC will be attached to provided you have appropriate permissions to the network resource.

    :::image type="content" source="media/create-virtual-machine/networking.png" alt-text="Screenshot of Networking tab screen.":::

1. Under **Advanced**, enable processor compatibility mode if required.

    :::image type="content" source="media/create-virtual-machine/advanced.png" alt-text="Screenshot of Advanced tab screen.":::

1. Under **Tags**, you can optionally add tags to the VM resource.

    :::image type="content" source="media/create-virtual-machine/tags.png" alt-text="Screenshot of Tags tab screen.":::

1. Under **Review + create**, review all the properties and select **Create**. The VM will be created in a few minutes.

## Next steps

- [Set up and manage self-service access](set-up-and-manage-self-service-access-scvmm.md).
- [Perform VM powercycle operations](perform-powercycle-operations.md).
- [Update configuration and resize a VM](update-configuration-and-resize-vm.md).
