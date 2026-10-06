---
title: Onboard your VMware vCenter resources in Azure
description: Learn how to browse your vCenter inventory and represent a subset of your VMware vCenter resources in Azure to enable self-service.
ms.topic: how-to
ms.date: 10/06/2026
ms.service: azure-arc
ms.subservice: vmware-vsphere-azure-arc
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
# Customer intent: As a VI admin, I want to represent a subset of my vCenter resources in Azure to enable self-service.
---

# Onboard your VMware vCenter resources in Azure

After you connect your VMware vCenter to Azure, you can browse your vCenter inventory from the Azure portal.

:::image type="content" source="media/enable-vcenter-resources/browse-vmware-inventory.png" alt-text="Screenshot of where to browse your VMware Inventory from the Azure portal." lightbox="media/enable-vcenter-resources/browse-vmware-inventory.png":::

To view all the connected vCenters, visit the VMware vCenter blade in Azure Arc center. From there, you can browse your virtual machines (VMs), resource pools, templates, networks, and datastores. From the inventory of your vCenter resources, you can select and onboard one or more resources in Azure through Azure Arc. When you onboard a vCenter resource in Azure, it creates an Azure resource that represents your vCenter resource. You can use this Azure resource to assign permissions or conduct management operations.

## Onboard resource pools, clusters, hosts, datastores, networks, and VM templates in Azure

In this section, you onboard resource pools, networks, and other non-VM resources in Azure.

>[!NOTE]
>Onboarding a VMware vSphere resource on Azure Arc is a read-only operation on vCenter. That is, it doesn't make changes to your resource in vCenter.

>[!NOTE]
> To onboard VM templates, VMware tools must be installed on them. If not installed, the **Manage Arc onboarding** option is grayed out.

1. From your browser, go to the vCenters blade on [Azure Arc Center](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/overview) and navigate to your inventory resources blade.

1. Select the resource or resources you want to onboard and then select **Manage Arc onboarding**.

1. Select your Azure Subscription and Resource Group and then select **Onboard**.

   This starts a deployment and creates a resource in Azure, creating representations for your VMware vSphere resources. It allows you to manage who can access those resources through Azure role-based access control (RBAC) granularly.

1. Repeat these steps for one or more network, resource pool, and VM template resources.

## Onboard existing virtual machines in Azure

1. From your browser, go to the vCenters blade on [Azure Arc Center](https://portal.azure.com/#blade/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/overview) and navigate to your vCenter.

   :::image type="content" source="media/enable-vcenter-resources/onboard-virtual-machines.png" alt-text="Screenshot of how to enable an existing virtual machine in the Azure portal." lightbox="media/enable-vcenter-resources/onboard-virtual-machines.png":::

1. Go to the VM inventory resource blade, select the VMs you want to onboard, and then select **Manage Arc onboarding**.

1. Select your Azure subscription and resource group.

1. Select **Arc agent with virtual hardware management** and then provide the Administrator username and password of the VM. For Linux VMs, there's an option to use SSH key-based authentication. 

   The Arc agent is the [Azure Arc connected machine agent](../servers/agent-overview.md). For information about the prerequisites for installing the Arc agent, see [Connected Machine agent prerequisites](../servers/prerequisites.md).

   Alternatively, you can choose not to install this agent by selecting **Virtual hardware management only**.

1. Select **Onboard** to start the deployment of the VM represented in Azure.

For information about the capabilities enabled by the Arc agent, see [supported operations](../servers/cloud-native/overview.md).

>[!NOTE]
>Moving VMware vCenter resources between resource groups and subscriptions isn't currently supported.
 
## Next steps

[Manage access to VMware resources through Azure RBAC](setup-and-manage-self-service-access.md).
