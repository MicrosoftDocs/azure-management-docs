---
title: Set up and manage self-service access to VMware resources through Azure RBAC
description: Learn how to manage access to your on-premises VMware resources through Azure role-based access control (Azure RBAC).
ms.topic: how-to
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
# Customer intent: As a VI admin, I want to manage access to my vCenter resources in Azure so that I can keep environments secure
ms.custom:
  - build-2025
---

# Set up and manage self-service access to VMware resources

After you enable your VMware vSphere resources in Azure, the final step in setting up a self-service experience for your teams is to provide them access. This article describes how to use built-in roles to manage granular access to VMware resources through Azure role-based access control (RBAC) and how to allow your teams to deploy and manage VMs.

## Prerequisites

- Your vCenter must be connected to Azure Arc.
- Your vCenter resources such as resource pools, clusters, hosts, networks, templates, and datastores are Arc-enabled.
- You have the User Access Administrator or Owner role at the scope (resource group or subscription) to assign roles to other users.


## Provide access to use Arc-enabled vSphere resources

To provision VMware VMs and change their size, add disks, change network interfaces, or delete them, your users need permissions on the compute, network, storage, and to the VM template resources that they'll use. The built-in **Azure Arc VMware Private Cloud User** role provides these permissions. 

Assign this role on individual resource pool (or cluster or host), network, datastore, and template that a user or a group needs to access.

1. Go to the [**VMware vCenters** list in Arc center](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/~/vCenter).

1. Search and select your vCenter. 

1. Navigate to the **Resourcepools/clusters/hosts** in **vCenter inventory** section in the table of contents.

1. Find and select resourcepool (or cluster or host). This selection takes you to the Arc resource representing the resourcepool.

1. Select **Access control (IAM)** in the table of contents.

1. Select **Add role assignments** on the **Grant access to this resource**.

1. Select **Azure Arc VMware Private Cloud User** role and select **Next**.

1. Select **Select members** and search for the Microsoft Entra user or group that you want to provide access.

8. Select the Microsoft Entra user or group name. Repeat this for each user or group to which you want to grant this permission.

1. Select **Review + assign** to complete the role assignment. 

1. Repeat steps 3-9 for each datastore, network, and VM template that you want to provide access to. 

If you organize your vSphere resources into a resource group, you can provide the same role at the resource group scope. 

Your users now have access to VMware vSphere cloud resources. However, your users also need permissions on the subscription or resource group where they want to deploy and manage VMs. 

## Provide access to subscription or resource group where VMs are deployed

In addition to having access to VMware vSphere resources through the **Azure Arc VMware Private Cloud User** role, your users must have permissions on the subscription and resource group where they deploy and manage VMs. 

The **Azure Arc VMware VM Contributor** role is a built-in role that provides permissions to conduct all VMware virtual machine operations. 

1. Go to the [Azure portal](https://portal.azure.com/).

1. Search and navigate to the subscription or resource group to which you want to provide access. 

1. Select **Access control (IAM)** in the table of contents on the left.

1. Select **Add role assignments** on the **Grant access to this resource**.

1. Select **Azure Arc VMware VM Contributor** role and select **Next**.

1. Select the option **Select members**, and search for the Microsoft Entra user or group that you want to provide access.

8. Select the Microsoft Entra user or group name. Repeat this for each user or group to which you want to grant this permission.

1. Select **Review + assign** to complete the role assignment. 


## Next steps

[Tutorial - Create a VM using Azure Arc-enabled vSphere](quick-start-create-a-vm.md).
