---
title:  Create a virtual machine on VMware vSphere using Azure Arc
description: This article helps you create a virtual machine using Azure portal. 
ms.date: 07/18/2025
ms.topic: how-to
ms.services: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
keywords: "VMware, Arc, Azure"
---


# Create a virtual machine on VMware vSphere using Azure Arc

Once your administrator has connected a VMware vCenter server to Azure, enabled vCenter resources such as Resource Pools, Clusters, Hosts, VM templates and VM networks in Azure, and provided you the required permissions on those resources, you'll be able to create a new VMware vCenter managed virtual machine in Azure. 

In this article, you'll learn how to create a new VMware vCenter managed virtual machine from Azure Arc portal. Refer to the **Reference** section of this documentation to explore non-portal methods such as Azure CLI, PowerShell, REST APIs, SDKs and Infrastructure-as-Code mechanisms.

## Prerequisites

- An Azure subscription and resource group where you have *Azure Arc VMware VM Contributor* role.
- A resource pool or a cluster or a host resource on which you have *Azure Arc VMware Private Cloud Resource User* role.
- A virtual machine template resource on which you have *Azure Arc VMware Private Cloud Resource User* role.
- A virtual network resource on which you have *Azure Arc VMware Private Cloud Resource User* role.
- A datastore resource on which you have *Azure Arc VMware Private Cloud Resource User* role. 

## Create a VM in Azure portal

1. Go to  [Azure portal](https://portal.azure.com/).
2. You can initiate the creation of a new VM in either of the following two ways:
   - Select **Azure Arc** as the service and then select **VMware vCenters** under **Host environments** from the left blade. Search and select your VMware vCenters. Select **Virtual machines** under **vCenter inventory** from the left blade and select **Add**. 

       :::image type="content" source="media/create-virtual-machine/add.png" alt-text="Screenshot of Add screen.":::

   Or
   - Select **Azure Arc** as the service and then select **Machine** under **Azure Arc resources** from the left blade. Select **Add/Create** and select **Create a machine in a connected host environment** from the dropdown.

       :::image type="content" source="media/create-virtual-machine/create-machine.png" alt-text="Screenshot of create a machine screen.":::

1. Once the **Create an Azure Arc virtual machine** page opens, under **Basics** > **Project details**, select the **Subscription** and **Resource group** where you want to deploy the VM.
1. Under **Instance details**, provide the following details:
   - **Virtual machine name** - Specify the name of the virtual machine.
   - **Custom location** - Select the custom location that your administrator has shared with you.
   - **Virtual machine kind** - Select **VMware**.
   - **Resource pool/cluster/host** - Select a resource pool/cluster/host from the dropdown menu.
   - **Datastore** - Select a datastore from the dropdown menu.

       :::image type="content" source="media/create-virtual-machine/machines.png" alt-text="Screenshot of machines screen.":::

1. Under **Template details**, provide the following details:
   - **Template** - Choose the VM template for deployment.
   - **Override template defaults** - Select the checkbox to override the default CPU cores and memory on the VM templates.
   - Specify computer name for the VM if the VM template has computer name associated with it.

1. Keep the **Enable Guest Management** checkbox selected to automatically install Azure connected machine agent immediately after the creation of the VM. [Azure connected machine agent (Arc agent)](../servers/agent-overview.md) is required if you're planning to use Azure management services to govern, patch, monitor, and secure your VM through Azure.

1. Under **Administrator account**, provide the following details:
   - Username
   - Password
   - Confirm password

    :::image type="content" source="media/create-virtual-machine/admin-account.png" alt-text="Screenshot of administrator account screen.":::

    If you are creating a Linux virtual machine, SSH key can be used as an authentication method instead of administrator account.

1. If you chose to enable guest management, choose the **Connectivity method** for the Arc agent that is installed in your VM connect to Azure. The available options are Public endpoint, Proxy server and Private endpoint. 

     - If you want to connect the Arc agent via proxy, provide the proxy server details. 

     - If you want to connect Arc agent via private endpoint, follow these steps to set up Azure private link and provide the same details.

1. Under **Disks**, you can optionally change the disks configured in the template. You can add more disks or update existing disks.

    :::image type="content" source="media/create-virtual-machine/disks.png" alt-text="Screenshot of Disks tab screen.":::

1. Under **Networking**, you can optionally change the network interfaces configured in the template. You can add Network interface cards (NICs) or update the existing NICs. You can also change the network that this NIC will be attached to provided you have appropriate permissions to the network resource.

    :::image type="content" source="media/create-virtual-machine/networking.png" alt-text="Screenshot of Networking tab screen.":::

1. Under **Advanced**, customize the guest operating system settings of the VM as required.

    :::image type="content" source="media/create-virtual-machine/advanced.png" alt-text="Screenshot of Advanced tab screen.":::

1. Under **Tags**, you can optionally add tags to the VM resource.

    :::image type="content" source="media/create-virtual-machine/tags.png" alt-text="Screenshot of Tags tab screen.":::

1. Under **Review + create**, review all the properties and select **Create**. The VM will be created in a few minutes.

## Next steps

- [Set up and manage self-service access](setup-and-manage-self-service-access.md).
- [Perform VM powercycle operations](perform-powercycle-operations.md).
- [Update configuration and resize a VM](update-configuration-and-resize-vm.md).
