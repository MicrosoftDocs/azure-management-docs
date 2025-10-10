---
title: Azure Container Registry custom roles
description: Use Azure RBAC custom roles to create your own fine-grained roles for Azure Container Registry.
ms.topic: concept-article
author: rayoef
ms.author: rayoflores
ms.date: 04/24/2025
ms.service: azure-container-registry
# Customer intent: "As an IT administrator, I want to create custom roles for Azure Container Registry, so that I can manage fine-grained access permissions tailored to specific user needs and enhance security within my container management environment."
---

# Azure Container Registry custom roles

Azure Container Registry (ACR) supports [Azure role-based access control (RBAC)](/azure/role-based-access-control/) to manage access to your registry. If none of the [Azure Container Registry built-in roles](container-registry-rbac-built-in-roles-overview.md) suit your needs, you can create custom roles with fine-grained permissions tailored to your scenario. This article describes the steps to define, create, and assign custom roles for Azure Container Registry.

## Custom role permissions

A set of permissions (actions and data actions) define a custom role. The permissions defined in the custom role determine what operations users can perform on registry resources.

To determine which permissions (actions and data actions) should be defined in a custom role, you can:
* Review the JSON definition of [Azure built-in roles directory for Containers](/azure/role-based-access-control/built-in-roles/containers) which includes commonly used permissions (actions and data actions) that are used in ACR built-in roles,
* Review the complete list of `Microsoft.ContainerRegistry` resource provider permissions ([Azure Container Registry reference of actions and data actions](/azure/role-based-access-control/permissions/containers#microsoftcontainerregistry))

To programmatically list all available permissions (actions and data actions) for the `Microsoft.ContainerRegistry` resource provider, you can use the following Azure CLI or Azure PowerShell commands.

```azurecli
az provider operation show --namespace Microsoft.ContainerRegistry
```

```azurepowershell
Get-AzProviderOperation -OperationSearchString Microsoft.ContainerRegistry/*
```

## Example: Custom role to manage webhooks

For example, the following JSON defines the minimum permissions (actions and data actions) for a custom role that permits [managing ACR webhooks](container-registry-webhook.md).

```json
{
   "assignableScopes": [
     "/subscriptions/<optional, but you can limit the visibility to one or more subscriptions>"
   ],
   "description": "Manage Azure Container Registry webhooks.",
   "Name": "Container Registry Webhook Contributor",
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

To define a custom role with a JSON definition, see [steps to create a custom role](/azure/role-based-access-control/custom-roles#steps-to-create-a-custom-role).
You can create the custom role using [Azure CLI](/azure/role-based-access-control/custom-roles-cli), [Azure Resource Manager template](/azure/role-based-access-control/custom-roles-template), or [Azure PowerShell](/azure/role-based-access-control/custom-roles-powershell).

> [!NOTE]
> In tenants configured with [Azure Resource Manager private link](/azure/azure-resource-manager/management/create-private-link-access-portal), Azure Container Registry supports wildcard actions such as `Microsoft.ContainerRegistry/*/read` or `Microsoft.ContainerRegistry/registries/*/write` in custom roles, granting access to all matching actions.
> In a tenant without an ARM private link, don't use wildcards and specify all required registry actions individually in a custom role.

## Assigning a custom role

Add or remove role assignments for a custom role in the same way that you manage role assignments for built-in roles.
Learn more about assigning Azure roles to an Azure identity by using the [Azure portal](/azure/role-based-access-control/role-assignments-portal), the [Azure CLI](/azure/role-based-access-control/role-assignments-cli), [Azure PowerShell](/azure/role-based-access-control/role-assignments-powershell), or other Azure tools.

## Next steps

* For a high-level overview of these built-in roles—including supported role assignment identity types, steps to perform a role assignment, and recommended roles for common scenarios—see [Azure Container Registry RBAC built-in roles](container-registry-rbac-built-in-roles-overview.md).
* To perform role assignments with optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
* For a detailed reference of every ACR built-in role, including the permissions granted by each role, see the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md).
