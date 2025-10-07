---
title: Create custom roles with Azure Arc-enabled VMware vSphere
description: This article describes how to create custom roles using the Azure portal for Azure Arc-enabled VMware vSphere. 
ms.topic: how-to
ms.date: 06/25/2025
ms.service: azure-arc
ms.author: v-gajeronika
author: Jeronika-MS
ms.subservice: azure-arc-vmware-vsphere
---

# Create custom roles with Azure Arc-enabled VMware vSphere

If the built-in roles of Azure Arc-enabled VMware vSphere don't meet the specific needs of your organization, you can create your own custom roles to provide permissions at a granular level to your end users. 

Just like built-in roles, you can assign custom roles to users at subscription and resource group scopes to control access. Custom roles are stored in a Microsoft Entra directory and can be shared across subscriptions. Each directory can have up to 5,000 custom roles. Custom roles can be created using the Azure portal, Azure PowerShell, Azure CLI, or the REST API. This article describes how to create custom roles using the Azure portal for Azure Arc-enabled VMware vSphere.

To understand more about Azure custom roles, refer to the following:

- [Understand Azure role definitions](/azure/role-based-access-control/role-definitions)
- [Create or update Azure custom roles using the Azure portal](/azure/role-based-access-control/custom-roles-portal)
- [Steps to assign an Azure role](/azure/role-based-access-control/role-assignments-steps)

## Prerequisites

Ensure you have permissions to create custom roles, such as [Owner](/azure/role-based-access-control/built-in-roles#owner) or [User Access Administrator](/azure/role-based-access-control/built-in-roles#user-access-administrator).

## Create custom role

To create a custom role with Azure Arc-enabled VMware vSphere, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/#home), open the subscription where you want the custom role to be created, and then open **Access control (IAM)**.
2. Select **+ Add** and then select **Add custom role**. 
      :::image type="content" source="media/create-custom-roles/add-custom-roles.png" alt-text="Screenshot of Add custom roles screen.":::
3. On the **Basics** tab, fill the details such as Custom role name, Description, and choose the baseline permissions and then select **Next**.
4. On the **Permissions** tab, select **+ Add permissions** to add actions to your baseline permissions or **Exclude permissions** to remove actions from your baseline permissions. If you are creating a new role from scratch, select **Add permissions**.
5. On the **Add permissions** or **Exclude permissions** window, search *vmware vsphere* and select **Microsoft.ConnectedVMwarevSphere**.
      :::image type="content" source="media/create-custom-roles/add-permissions.png" alt-text="Screenshot of Add permissions screen.":::
6. On the **Microsoft.ConnectedVMwarevSphere** page, select the desired permissions to add or exclude and then select **Add**. 
7. Add permissions from other Resource Providers to this custom role, if needed, and select **Next**.
8. On the **Assignable scopes** tab, you can optionally choose additional subscription(s) and/or the resource group(s) in which this custom role can be available for assignment and then select **Next**.
9. On the **JSON** tab, you can optionally download the JSON format of the custom role to create more custom roles from a baseline permission set. Once done, select **Next**.
10. On the **Review + create** tab, select **Create** to create your custom role for Azure Arc-enabled VMware vSphere.
11. Once created, you can view, update, and delete the custom roles by following these steps:
     - [List custom roles](/azure/role-based-access-control/custom-roles-portal#list-custom-roles)
     - [Update a custom role](/azure/role-based-access-control/custom-roles-portal#update-a-custom-role)
     - [Delete a custom role](/azure/role-based-access-control/custom-roles-portal#delete-a-custom-role)

To manage custom roles using Azure PowerShell, Azure CLI, REST APIs, ARM, or Bicep templates, refer the [detailed documentation on Azure Role based Access Control](/azure/role-based-access-control/).

## Next Steps

- [Set up and manage self-service access to VMware resources through Azure RBAC](setup-and-manage-self-service-access.md).
- [Create a virtual machine on VMware vCenter using Azure Arc](quick-start-create-a-vm.md).
- [Perform VM operations on VMware VMs through Azure](perform-vm-ops-through-azure.md).
