---
title: Set up and manage self-service access to SCVMM resources
description: This article describes how to use built-in roles to manage granular access to SCVMM resources through Azure Role-based Access Control (RBAC).
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.topic: how-to
ms.date: 02/09/2026
keywords: "VMM, Arc, Azure"
ms.custom:
  - build-2025
# Customer intent: As an IT administrator, I want to manage user access to SCVMM resources through Azure Role-based Access Control, so that my teams can effectively deploy and manage VMs with the appropriate permissions.
---

# Set up and manage self-service access to SCVMM resources

After you enable your SCVMM resources in Azure, provide your teams with the required access for a self-service experience. This article describes how to use built-in roles to manage granular access to SCVMM resources through Azure Role-based Access Control (RBAC) and allow your teams to deploy and manage VMs.

## Prerequisites

- Your SCVMM instance is connected to Azure Arc.
- Your SCVMM resources such as virtual machines, clouds, VM networks, and VM templates are Azure enabled.
- You have the **User Access Administrator** or **Owner** role at the scope (resource group or subscription) to assign roles to other users.

## Provide access to use Azure Arc-enabled SCVMM resources

To provision SCVMM VMs and change their size, add disks, change network interfaces, or delete them, your users need permission on the compute, network, storage, and to the VM template resources that they'll use. The built-in Azure Arc SCVMM Private Cloud User role provides these permissions.

Assign this role to an individual cloud, VM network, and VM template that a user or a group needs to access.

1. Go to the [SCVMM management servers](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/~/scVmmManagementServer) list in Arc center.
1. Search and select your SCVMM management server.
1. Navigate to the **Clouds** in **SCVMM inventory** section in the table of contents.
1. Find and select the cloud for which you want to assign permissions. 
     This selection takes you to the Arc resource representing the SCVMM Cloud.
1. Select **Access control (IAM)** in the table of contents.
1. Select **Add role assignments** on the **Grant access to this resource**.
1. Select **Azure Arc ScVmm Private Cloud User** role and select **Next**.
1. Select **Select members** and search for the Microsoft Entra user or group that you want to provide access to.
1. Select the Microsoft Entra user or group name. Repeat this step for each user or group to which you want to grant this permission.
1. Select **Review + assign** to complete the role assignment.
1. Repeat steps 3-9 for each VM network and VM template that you want to provide access to.

If you organize your SCVMM resources into a resource group, you can provide the same role at the resource group scope.

Your users now have access to SCVMM cloud resources. However, your users also need permission on the subscription or resource group where they deploy and manage VMs.

## Provide access to subscription or resource group where VMs are deployed

In addition to having access to SCVMM resources through the **Azure Arc ScVmm Private Cloud User** role, your users must have permissions on the subscription and resource group where they deploy and manage VMs.

The **Azure Arc ScVmm VM Contributor** role is a built-in role that provides permissions to conduct all SCVMM virtual machine operations.

1. Go to the [Azure portal](https://portal.azure.com/).
1. Search and navigate to the subscription or resource group to which you want to provide access.
1. Select **Access control (IAM)** from the table of contents on the left.
1. Select **Add role assignments** on the **Grant access to this resource**.
1. Select **Azure Arc ScVmm VM Contributor** role and select **Next**.
1. Select the option **Select members**, and search for the Microsoft Entra user or group that you want to provide access to.
1. Select the Microsoft Entra user or group name. Repeat this step for each user or group to which you want to grant this permission.
1. Select **Review + assign** to complete the role assignment.

## Next steps

[Create an Azure Arc VM](create-virtual-machine.md).
