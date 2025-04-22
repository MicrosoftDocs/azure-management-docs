---
title: Azure Container Registry custom roles
description: Use Azure RBAC custom roles to create your own fine-grained roles for Azure Container Registry.
ms.topic: conceptual
author: rayoef, johnsonshi
ms.author: rayoflores, johsh
ms.date: 04/21/2025
ms.service: azure-container-registry
---

# Azure Container Registry custom roles

Azure Container Registry (ACR) supports [Azure role-based access control (RBAC)](/azure/role-based-access-control/) to manage access to your registry. You can assign [Azure Container Registry built-in roles](/articles/container-registry/container-registry-rbac-built-in-roles.md) or create [custom roles](/azure/role-based-access-control/custom-roles) with fine-grained permissions tailored to your specific needs. This article describes the steps to define, create, and assign custom roles for Azure Container Registry.

## Custom role permissions

Custom roles are defined by a set of permissions (actions and data actions). The permissions defined in the custom role determine what operations users can perform on registry resources.

To determine which permissions (actions and data actions) should be defined in a custom role, you can:
* review the list of `Microsoft.ContainerRegistry` [actions](/azure/role-based-access-control/permissions/containers#microsoftcontainerregistry),
* review the actions packaged in [Azure built-in roles for Containers](/azure/role-based-access-control/built-in-roles/containers),
* or run the following command using either Azure CLI or Azure PowerShell to list all available actions for the `Microsoft.ContainerRegistry` resource provider.

### [Azure CLI](#tab/azure-cli)

```azurecli
az provider operation show --namespace Microsoft.ContainerRegistry
```

To define a custom role, see [Steps to create a custom role](/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role).

> [!NOTE]
> In tenants configured with [Azure Resource Manager private link](/azure/azure-resource-manager/management/create-private-link-access-portal), Azure Container Registry supports wildcard actions such as `Microsoft.ContainerRegistry/*/read` or `Microsoft.ContainerRegistry/registries/*/write` in custom roles, granting access to all matching actions. In a tenant without an ARM private link, specify all required registry actions individually in a custom role.

### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Get-AzProviderOperation -OperationSearchString Microsoft.ContainerRegistry/*
```

To define a custom role, see [Steps to create a custom role](/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role).

> [!NOTE]
> In tenants configured with [Azure Resource Manager private link](/azure/azure-resource-manager/management/create-private-link-access-portal), Azure Container Registry supports wildcard actions such as `Microsoft.ContainerRegistry/*/read` or `Microsoft.ContainerRegistry/registries/*/write` in custom roles, granting access to all matching actions. In a tenant without an ARM private link, specify all required registry actions individually in a custom role.

## Example: Custom role to import images

For example, the following JSON defines the minimum actions for a custom role that permits [managing webhooks for registries and geo-replications](container-registry-import-images.md).

```json
{
   "assignableScopes": [
     "/subscriptions/<optional, but you can limit the visibility to one or more subscriptions>"
   ],
   "description": "Manage Azure Container Registry webhooks.",
   "Name": "Container Registry Webhook Manager",
   "permissions": [
     {
       "actions": [
         "Microsoft.ContainerRegistry/registries/webhooks/read",
         "Microsoft.ContainerRegistry/registries/webhooks/write",
         "Microsoft.ContainerRegistry/registries/webhooks/delete"
       ],
       "dataActions": [],
       "notActions": [],
       "notDataActions": []
     }
   ],
   "roleType": "CustomRole"
 }
```

## Creating or updating a custom role

To create or update a custom role using the JSON description, use the [Azure CLI](/azure/role-based-access-control/custom-roles-cli), [Azure Resource Manager template](/azure/role-based-access-control/custom-roles-template), [Azure PowerShell](/azure/role-based-access-control/custom-roles-powershell), or other Azure tools.

## Assigning a custom role

Add or remove role assignments for a custom role in the same way that you manage role assignments for built-in Azure roles.
Learn more about assigning Azure roles to an Azure identity by using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), the [Azure CLI](/azure/role-based-access-control/role-assignments-cli), [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell), or other Azure tools.
