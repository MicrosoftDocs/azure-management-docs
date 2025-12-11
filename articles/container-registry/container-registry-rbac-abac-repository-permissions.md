---
title: Azure attribute-based access control (Azure ABAC) repository permissions in Azure Container Registry
description: Use Azure ABAC to manage repository permissions for an Azure Container Registry.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 12/11/2025
ms.service: azure-container-registry
# Customer intent: As a container registry administrator, I want to configure attribute-based access control (ABAC) with role assignments in Azure Container Registry so that I can manage repository permissions more granularly and enhance security for my organization's container images.
---

# Azure attribute-based access control (Azure ABAC) repository permissions in Azure Container Registry

Azure Container Registry (ACR) supports [Azure attribute-based access control (Azure ABAC)](/azure/role-based-access-control/conditions-overview) for managing repository permissions. This feature enhances security by enabling more granular permissions management to container registry repositories. 

Azure ABAC builds upon Azure role-based access control (Azure RBAC) by introducing repository-specific conditions in role assignments. With Azure ABAC, you can grant a security principal access to a repository resource based on attributes.

This article describes how to enable Azure ABAC for repository permissions in your container registry, and how to assign role assignments with ABAC conditions to scope permissions to specific repositories.

## Configure role assignment permissions mode setting

To use Azure ABAC to manage repository permissions, set the **Role assignment permissions mode** for your registry to **RBAC Registry + ABAC Repository Permissions**. This mode allows you to use optional ABAC conditions to scope role assignments to specific repositories, in addition to RBAC role assignments using [ACR built-in roles](container-registry-rbac-built-in-roles-overview.md).

You can configure the **Role assignment permissions mode** when creating your registry, or you can update the setting on any existing registry. This configuration can be done either through the [Azure portal](container-registry-get-started-portal.md#create-a-container-registry) or the [Azure CLI](container-registry-get-started-azure-cli.md#configure-role-assignment-permissions-mode).

If you change the setting for an existing registry, be sure to understand the effects on existing role assignments and tasks as described in the following sections.

> [!NOTE]
> Ensure that you have the latest version of the Azure CLI installed by running the Azure CLI command `az upgrade`. Additionally, if you have previously participated in the private preview of this feature, you may have installed a custom private preview extension to manage ACR ABAC. This custom extension is no longer needed and should be uninstalled (to avoid conflicts) by running the Azure CLI command `az extension remove --name acrabac`.

### Effect on existing role assignments

If you configure a registry to use **RBAC Registry + ABAC Repository Permissions**, be aware that some existing role assignments aren't honored, or have different effects, because a different set of ACR built-in roles apply to ABAC-enabled registries.

For example, the `AcrPull`, `AcrPush`, and `AcrDelete` roles aren't honored in an ABAC-enabled registry.
Instead, in ABAC-enabled registries, use the `Container Registry Repository Reader`, `Container Registry Repository Writer`, and `Container Registry Repository Contributor` roles to grant either registry-wide or repository-specific image permissions.

Additionally, privileged roles such as `Owner`, `Contributor`, and `Reader` will have different effects in an ABAC-enabled registry. In these repositories, these privileged roles grant only control plane permissions to create, update, and delete the registry itself, without granting data plane permissions to the repositories and images in the registry.

For more information on these roles based on your scenario and registry role assignment permissions mode, see [Azure Container Registry  permissions and role assignments overview](container-registry-rbac-built-in-roles-overview.md). Alternatively, consult the [ACR built-in roles reference](container-registry-rbac-built-in-roles-directory-reference.md) for an in-depth description of each role.

### Effect on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs

If you configure a registry to use **RBAC Registry + ABAC Repository Permissions**, new and existing ACR Tasks, as well as Quick Tasks, Quick Builds, and Quick Runs, will be affected. They will no longer have default data plane access to an ABAC-enabled source registry and its content.

To understand the effects of this change—and how to grant data plane access for ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs in ABAC-enabled source registries—see [Appendix: Effects of Enabling ABAC on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs](#appendix-effects-of-enabling-abac-on-acr-tasks-quick-tasks-quick-builds-and-quick-runs).

### Create a registry with ABAC enabled

#### [Azure portal](#tab/azure-portal)

When creating a [new registry through Azure portal](container-registry-get-started-portal.md), select **RBAC Registry + ABAC Repository Permissions** for **Role assignment permissions mode**.

:::image type="content" source="media/container-registry-get-started-portal/configure-container-registry-options.png" alt-text="Screenshot showing the container registry creation settings in the portal":::

#### [Azure CLI](#tab/azure-cli)

When running `az acr create`, use the `--role-assignment-mode` parameter to [set the role assignment permissions mode](container-registry-get-started-azure-cli.md#configure-role-assignment-permissions-mode). Specify `rbac-abac` to create a registry with Azure ABAC enabled:

```azurecli
az acr create --name <registry-name> --resource-group <resource-group> --sku <sku> --role-assignment-mode 'rbac-abac' --location <location>
```

---

### Update an existing registry to enable ABAC

> [!IMPORTANT]
> Changing the role assignment permissions mode of an existing registry can affect existing role assignments and tasks, as described in the previous sections. Be sure to review these effects before changing the setting.

#### [Azure portal](#tab/azure-portal)

To view the existing role assignment permissions mode of a registry, go to the registry in the Azure portal. In the service menu, under **Settings**, select **Properties**, then check the **Role assignment permissions mode** setting.

To enable Azure RBAC, select **RBAC Registry + ABAC Repository Permissions**, then select **Save.**

#### [Azure CLI](#tab/azure-cli)

When running `az acr update`, use the `--role-assignment-mode` parameter to set the registry role assignment permissions mode. Specify `rbac-abac` to create a registry with **RBAC Registry + ABAC Repository Permissions** enabled.

To show the current role assignment permissions mode of a registry, run the following command:

```azurecli
az acr show --name <registry-name> --resource-group <resource-group> --query "roleAssignmentMode"
```

To update an existing registry to enable Azure ABAC, run the following command:

```azurecli
az acr update --name <registry-name> --resource-group <resource-group> --role-assignment-mode 'rbac-abac'
```

---

## Assigning Microsoft Entra ABAC repository permissions

You can use either the Azure portal or Azure CLI to assign Microsoft Entra ABAC conditions to scope role assignments to specific repositories.
This section provides examples of how to add ABAC conditions for a specific repository, a repository prefix (wildcard), or multiple repository prefixes (multiple wildcards).

### ABAC-enabled built-in roles

The following ACR built-in roles are ABAC-enabled roles.
You can specify optional ABAC conditions to the following roles to optionally scope role assignments to specific repositories.

* `Container Registry Repository Reader` - ABAC-enabled role that grants permissions to **read** images, tags, and metadata within repositories in a registry.
* `Container Registry Repository Writer` - ABAC-enabled role that grants permissions to **read, write, and update** images, tags, and metadata within repositories in a registry.
* `Container Registry Repository Contributor` - ABAC-enabled role that grants permissions to **read, write, update, and delete** images, tags, and metadata within repositories in a registry.

Take note that these roles **don't support catalog listing permissions to list repositories** in a registry.
To list all repositories in a registry (without granting permissions to read repository content), you must additionally assign the `Container Registry Repository Catalog Lister` role.
This separate role **does not support ABAC conditions** and will always have permissions to **list all repositories** in a registry.

> [!IMPORTANT]
> **If you assign an ABAC-enabled role without ABAC conditions, the role assignment won't be scoped to repositories**.
> This means that a role assignment without ABAC conditions will be treated as a registry-wide role assignment, granting permissions to all repositories in the registry.
> **To scope a role assignment to specific repositories, you must include ABAC conditions when assigning an ABAC-enabled role**.

For more information on the role based on your scenario and registry role assignment permissions mode, see [scenarios for ACR built-in roles](container-registry-rbac-built-in-roles-overview.md).
Alternatively, consult the [ACR built-in roles reference](container-registry-rbac-built-in-roles-directory-reference.md) for an in-depth description of each role.

## Scope role assignment to a specific repository

In this example, we assign the `Container Registry Repository Reader` role to grant pull permissions to a single repository.
By adding ABAC conditions, this role assignment lets the identity pull images, view tags, and read metadata only from the specified repository, preventing access to other repositories in the registry.

### [Azure portal](#tab/azure-portal)

Navigate to the registry's "Access control (IAM)" blade. Click "Add" and select "Add role assignment."

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-03-add-role-assignment.png" alt-text="Screenshot of adding a role assignment." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-03-add-role-assignment.png":::

Select `Container Registry Repository Reader` as the role.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-04-select-role.png" alt-text="Screenshot of selecting a role to assign." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-04-select-role.png":::

Continue by selecting the identity to assign the role to.

Afterwards, continue to the "Conditions" tab. Select the "Add condition" button to add a new ABAC condition to restrict the role assignment scope.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-05-add-conditions-tab.png" alt-text="Screenshot of adding conditions for role assignment." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-05-add-conditions-tab.png":::

Select the "Visual" editor option in the ABAC condition builder.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-06-add-condition-detail.png" alt-text="Screenshot of selecting Visual editor option." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-06-add-condition-detail.png":::

Select the actions (permissions) to grant in this repository-scoped role assignment.
For most use cases, select all actions (permissions) belonging to the role you selected earlier, ensuring that identities can only perform these actions within the repository scope.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-07-select-actions.png" alt-text="Screenshot of selecting actions and permissions to grant." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-07-select-actions.png":::

Add an expression for the ABAC condition to restrict the role assignment to a specific repository.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-08-build-expression.png" alt-text="Screenshot of adding expression for the ABAC condition." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-08-build-expression.png":::

Configure the following options for the expression to scope the ABAC condition to a specific repository:
  * Attribute source: `Request`
  * Attribute: `Repository name`
  * Operator: `StringEqualsIgnoreCase`
  * Value: `<repository-name>` - the full name of the repository.
    * For example, if the full repository name is `nginx`, enter `nginx`.
    * If the full repository name is `backend/nginx`, enter `backend/nginx`.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-09-expression-specific-repository.png" alt-text="Screenshot of configuring expression to scope the ABAC condition to a specific repository." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-09-expression-specific-repository.png":::

Click "Save" to save the ABAC condition.

Review the role assignment ABAC condition.
The review page includes a code expression of the ABAC condition, which can be used to perform the same role assignment with the same ABAC condition using Azure CLI.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-10-finished-condition-specific-repository.png" alt-text="Screenshot of reviewing ABAC condition to scope to a specific repository." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-10-finished-condition-specific-repository.png":::

Perform the role assignment by clicking "Review + assign."

Once the role assignment is created, you can view, edit, or delete the role assignment.
Navigate to the registry's "Access control (IAM)" and select the "Role assignments" tab to view the list of existing role assignments that apply to the registry.

### [Azure CLI](#tab/azure-cli)

To add a role assignment with an ABAC condition scoped to a specific repository, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create).
The [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create) command includes the following parameters related to ABAC conditions.

| Parameter | Type | Description |
| --- | --- | --- |
| `role` | String | The name of ID of the role to assign, for example `Container Registry Repository Reader`. |
| `scope` | String | The full resource ID of the registry. This parameter should be the registry resource ID (without the repository name), even for role assignments scoped to a specific repository, with the format `/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.ContainerRegistry/registries/{registry-name}`. You can also specify the resource group or subscription scope to perform this role assignment across all registries in the resource group or subscription. |
| `assignee` | String | The email address or object ID of the Microsoft Entra identity to assign the role to. For example, you can specify `user@contoso.com` for a user assignee. You can also specify the object ID of the identity. |
| `assignee-object-id` | String | Use this parameter instead of '--assignee' to bypass Graph API invocation if there's insufficient privileges. This parameter only works with object IDs for users, groups, service principals, and managed identities. For managed identities, use the principal ID. For service principals, use the object ID and not the application ID. |
| `description` | String | A description for the role assignment to help identify the purpose of the role assignment. This parameter is optional. |
| `condition` | String | A code expression which defines the ABAC condition for the role assignment. |
| `condition-version` | String | Version of the condition syntax. If `--condition` is specified without `--condition-version`, the version is set to the default value of `2.0`. |

The ACR team recommends using the visual editor in the Azure portal role assignment experience to create the expression for the ABAC condition.

Use the following expression for the ABAC condition to restrict the role assignment to the repository `nginx`:

```
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringEqualsIgnoreCase 'nginx'
 )
)
```

You can use store this ABAC condition into a shell variable named `condition` for use in the `az role assignment create` command.

```azurecli
condition=$(cat <<'EOF' | tr -d '\n'
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringEqualsIgnoreCase 'nginx'
 )
)
EOF
)
```

You can inspect the `condition` variable to verify that it's correct.
Ensure that you use double quotes (`"`) whenever you use the `condition` variable.

```azurecli
echo "$condition"
```

Before performing the role assignment, you must also query the registry resource ID for the `--scope` parameter.
The `--scope` parameter should always be the registry resource ID (without the repository name), even for role assignments scoped to a specific repository.
You can also specify the resource group or subscription scope to perform this role assignment across all registries in the resource group or subscription.

```azurecli
scope=$(az acr show --name <registry-name> --resource-group <resource-group> --query "id" -o tsv)
```

Finally, to perform the role assignment, run the following command:

```azurecli
az role assignment create \
  --role "Container Registry Repository Reader" \
  --scope "$scope" \
  --assignee "user@contoso.com" \
  --description "Read access to a specific repository" \
  --condition "$condition" \
  --condition-version "2.0"
```

The following shows an example of the output:

```json
{
    "canDelegate": null,
    "condition": "( (  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})  AND  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'}) ) OR  (  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringEqualsIgnoreCase 'nginx' ))",
    "conditionVersion": "2.0",
    "description": "{description}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ContainerRegistry/registries/{registryName}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

To update or delete the role assignment, you can use [`az role assignment update`](/cli/azure/role/assignment#az-role-assignment-update) or [`az role assignment delete`](/cli/azure/role/assignment#az-role-assignment-delete).

---

## Scope role assignment to multiple repositories using repository prefix (wildcard)

In this example, we assign the `Container Registry Repository Reader` role to grant pull permissions to multiple repositories with a common prefix (wildcard).
By adding ABAC conditions, this role assignment lets the identity pull images, view tags, and read metadata only from the repositories with a common prefix, preventing access to other repositories in the registry.

### [Azure portal](#tab/azure-portal)

If you followed the previous example to assign the `Container Registry Repository Reader` role to a specific repository, you must delete that role assignment (by navigating to the "Access control (IAM)" blade and selecting the "Role assignments" tab), before creating a new one with an ABAC condition scoped to a repository prefix.

Follow the same steps as in the [previous example](#scope-role-assignment-to-a-specific-repository) to perform a role assignment with ABAC conditions.

In the step to add an expression for the ABAC condition, configure an expression for an ABAC condition to scope the role assignment to multiple repositories with a common prefix (wildcard).
Configure the following options:
  * Attribute source: `Request`
  * Attribute: `Repository name`
  * Operator: `StringStartsWithIgnoreCase`
  * Value: `<repository-prefix>` - the prefix of the repositories, including the trailing slash `/`.
    * For example, to grant permissions to all repositories with the prefix `backend/`, such as `backend/nginx` and `backend/redis`, enter `backend/`.
    * To grant permissions to all repositories with the prefix `frontend/js/`, such as `frontend/js/react` and `frontend/js/vue`, enter `frontend/js/`.

> [!IMPORTANT]
> The trailing slash `/` is required in the `Value` field for the expression of the ABAC condition.
> If you don't include the trailing slash `/`, you may unintentionally grant permissions to other repositories that don't match the prefix.
> For example, if you enter `backend` without the trailing slash `/`, the role assignment grants permissions to all repositories with the prefix `backend`, such as `backend/nginx`, `backend/redis`, `backend-infra/k8s`, `backend-backup/store`, `backend`, and `backendsvc/containers`.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-11-expression-repository-prefix.png" alt-text="Screenshot of configuring expression to scope the ABAC condition to a repository prefix." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-11-expression-repository-prefix.png":::

Click "Save" to save the ABAC condition.

Review the role assignment ABAC condition.
The review page includes a code expression of the ABAC condition, which can be used to perform the same role assignment with the same ABAC condition using Azure CLI.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-12-finished-condition-repository-prefix.png" alt-text="Screenshot of reviewing ABAC condition to scope to a repository prefix." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-12-finished-condition-repository-prefix.png":::

Perform the role assignment by clicking "Review + assign."

Once the role assignment is created, you can view, edit, or delete the role assignment.
Navigate to the registry's "Access control (IAM)" and select the "Role assignments" tab to view the list of existing role assignments that apply to the registry.

### [Azure CLI](#tab/azure-cli)

If you followed the previous example to assign the `Container Registry Repository Reader` role to a specific repository, you must delete that role assignment (using [`az role assignment delete`](/cli/azure/role/assignment#az-role-assignment-delete)), before creating a new one with an ABAC condition scoped to a repository prefix.

To add a role assignment with an ABAC condition scoped to multiple repositories under a prefix, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create).

The ACR team recommends using the visual editor in the Azure portal role assignment experience to create the expression for the ABAC condition.

Use the following expression for the ABAC condition to restrict the role assignment to the repository `nginx`:

```
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/'
 )
)
```

You can use store this ABAC condition into a shell variable named `condition` for use in the `az role assignment create` command.

```azurecli
condition=$(cat <<'EOF' | tr -d '\n'
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/'
 )
)
EOF
)
```

You can inspect the `condition` variable to verify that it's correct.
Ensure that you use double quotes (`"`) whenever you use the `condition` variable.

```azurecli
echo "$condition"
```

Before performing the role assignment, you must also query the registry resource ID for the `--scope` parameter.
The `--scope` parameter should always be the registry resource ID (without the repository name), even for role assignments scoped to a repository prefix.
You can also specify the resource group or subscription scope to perform this role assignment across all registries in the resource group or subscription.

```azurecli
scope=$(az acr show --name <registry-name> --resource-group <resource-group> --query "id" -o tsv)
```

Finally, to perform the role assignment, run the following command:

```azurecli
az role assignment create \
  --role "Container Registry Repository Reader" \
  --scope "$scope" \
  --assignee "user@contoso.com" \
  --description "Read access to a multiple repositories under a repository prefix" \
  --condition "$condition" \
  --condition-version "2.0"
```

The following shows an example of the output:

```json
{
    "canDelegate": null,
    "condition": "( (  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})  AND  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'}) ) OR  (  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/' ))",
    "conditionVersion": "2.0",
    "description": "{description}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ContainerRegistry/registries/{registryName}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

To update or delete the role assignment, you can use [`az role assignment update`](/cli/azure/role/assignment#az-role-assignment-update) or [`az role assignment delete`](/cli/azure/role/assignment#az-role-assignment-delete).

---

## Scope role assignment to multiple repositories using multiple repository prefixes (multiple wildcards)

In this example, we assign the `Container Registry Repository Reader` role to grant pull permissions to multiple repositories under two different prefixes (multiple wildcards).
By adding ABAC conditions, this role assignment lets the identity pull images, view tags, and read metadata only from the specified repository, preventing access to other repositories in the registry.

### [Azure portal](#tab/azure-portal)

If you followed the previous example to assign the `Container Registry Repository Reader` role to a specific repository, you must delete that role assignment (by navigating to the "Access control (IAM)" blade and selecting the "Role assignments" tab), before creating a new one with an ABAC condition scoped to a repository prefix.

Follow the same steps as in the [previous example](#scope-role-assignment-to-a-specific-repository) to perform a role assignment with ABAC conditions.

In the step to add an expression for the ABAC condition, configure two expressions to scope the role assignment to multiple repositories under two prefixes: `backend/` and `frontend/js/` (multiple wildcards).

For the first expression, configure the following options:
  * Attribute source: `Request`
  * Attribute: `Repository name`
  * Operator: `StringStartsWithIgnoreCase`
  * Value: `<repository-prefix>` - the prefix of the repositories, including the trailing slash `/`.
    * For example, to grant permissions to all repositories with the prefix `backend/`, such as `backend/nginx` and `backend/redis`, enter `backend/`.
    * To grant permissions to all repositories with the prefix `frontend/js/`, such as `frontend/js/react` and `frontend/js/vue`, enter `frontend/js/`.

Click "Add expression."
**Ensure that the boolean operator is set to "Or"**.
You can optionally select "Group" to group expressions together and control the order of evaluation.
The visual editor also supports multiple boolean operators including "And," "Or," hierarchical grouping, and negation.

For the second expression, configure the following options:
  * Attribute source: `Request`
  * Attribute: `Repository name`
  * Operator: `StringStartsWithIgnoreCase`
  * Value: `<repository-prefix>` - the prefix of the repositories, including the trailing slash `/`.
    * For example, to grant permissions to all repositories with the prefix `backend/`, such as `backend/nginx` and `backend/redis`, enter `backend/`.
    * To grant permissions to all repositories with the prefix `frontend/js/`, such as `frontend/js/react` and `frontend/js/vue`, enter `frontend/js/`.

> [!IMPORTANT]
> The trailing slash `/` is required in the `Value` field for the expression of the ABAC condition.
> If you don't include the trailing slash `/`, you may unintentionally grant permissions to other repositories that don't match the prefix.
> For example, if you enter `backend` without the trailing slash `/`, the role assignment grants permissions to all repositories with the prefix `backend`, such as `backend/nginx`, `backend/redis`, `backend-infra/k8s`, `backend-backup/store`, `backend`, and `backendsvc/containers`.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-13-expression-multiple-repository-prefixes.png" alt-text="Screenshot of configuring expression to scope the ABAC condition to multiple repository prefixes." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-13-expression-multiple-repository-prefixes.png":::

Click "Save" to save the ABAC condition.

Review the role assignment ABAC condition.
The review page includes a code expression of the ABAC condition, which can be used to perform the same role assignment with the same ABAC condition using Azure CLI.

:::image type="content" source="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-14-finished-condition-multiple-prefixes.png" alt-text="Screenshot of reviewing ABAC condition to scope to multiple repository prefixes." lightbox="media/container-registry-rbac-abac-repository-permissions/rbac-abac-repository-permissions-14-finished-condition-multiple-prefixes.png":::

Perform the role assignment by clicking "Review + assign."

Once the role assignment is created, you can view, edit, or delete the role assignment.
Navigate to the registry's "Access control (IAM)" and select the "Role assignments" tab to view the list of existing role assignments that apply to the registry.

### [Azure CLI](#tab/azure-cli)

If you followed the previous example to assign the `Container Registry Repository Reader` role to a specific repository, you must delete that role assignment (using [`az role assignment delete`](/cli/azure/role/assignment#az-role-assignment-delete)), before creating a new one with an ABAC condition scoped to a repository prefix.

To add a role assignment with an ABAC condition scoped to multiple repositories under multiple prefixes, use [az role assignment create](/cli/azure/role/assignment#az-role-assignment-create).

The ACR team recommends using the visual editor in the Azure portal role assignment experience to create the expression for the ABAC condition.

Use the following expression for the ABAC condition to restrict the role assignment to the repository `nginx`:

```
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/'
  OR
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'frontend/js/'
 )
)
```

You can use store this ABAC condition into a shell variable named `condition` for use in the `az role assignment create` command.

```azurecli
condition=$(cat <<'EOF' | tr -d '\n'
(
 (
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})
  AND
  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'})
 )
 OR 
 (
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/'
  OR
  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'frontend/js/'
 )
)
EOF
)
```

You can inspect the `condition` variable to verify that it's correct.
Ensure that you use double quotes (`"`) whenever you use the `condition` variable.

```azurecli
echo "$condition"
```

Before performing the role assignment, you must also query the registry resource ID which for the `--scope` parameter.
The `--scope` parameter should always be the registry resource ID (without the repository name), even for role assignments scoped to multiple repository prefixes.
You can also specify the resource group or subscription scope to perform this role assignment across all registries in the resource group or subscription.

```azurecli
scope=$(az acr show --name <registry-name> --resource-group <resource-group> --query "id" -o tsv)
```

Finally, to perform the role assignment, run the following command:

```azurecli
az role assignment create \
  --role "Container Registry Repository Reader" \
  --scope "$scope" \
  --assignee "user@contoso.com" \
  --description "Read access to a multiple repositories under multiple repository prefixes" \
  --condition "$condition" \
  --condition-version "2.0"
```

The following shows an example of the output:

```json
{
    "canDelegate": null,
    "condition": "( (  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/content/read'})  AND  !(ActionMatches{'Microsoft.ContainerRegistry/registries/repositories/metadata/read'}) ) OR  (  @Request[Microsoft.ContainerRegistry/registries/repositories:name] StringStartsWithIgnoreCase 'backend/' ))",
    "conditionVersion": "2.0",
    "description": "{description}",
    "id": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Authorization/roleAssignments/{roleAssignmentId}",
    "name": "{roleAssignmentId}",
    "principalId": "{userObjectId}",
    "principalType": "User",
    "resourceGroup": "{resourceGroup}",
    "roleDefinitionId": "/subscriptions/{subscriptionId}/providers/Microsoft.Authorization/roleDefinitions/{roleDefinitionId}",
    "scope": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ContainerRegistry/registries/{registryName}",
    "type": "Microsoft.Authorization/roleAssignments"
}
```

To update or delete the role assignment, you can use [`az role assignment update`](/cli/azure/role/assignment#az-role-assignment-update) or [`az role assignment delete`](/cli/azure/role/assignment#az-role-assignment-delete).

---

## Maximum number of ABAC conditions

The Azure portal supports a limited number of ABAC conditions per role assignment.

To add more than the Azure portal limit of ABAC conditions, you can use the Azure CLI to create the role assignment with more ABAC conditions.

## Appendix: Effects of Enabling ABAC on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs

### Effect on ACR Tasks

> [!IMPORTANT]
> When a registry is configured with **"RBAC Registry + ABAC Repository Permissions"**, new and existing ACR Tasks no longer have default data plane access to the source registry, including its images and artifacts within repositories. You must explicitly attach and set an identity that will be used by new and existing ACR Tasks to authenticate to an ABAC-enabled source registry. Afterwards, grant the attached identity the appropriate ACR built-in role to enable authentication and access to registry repositories.

#### Attaching an Identity to a New ACR Task for Source Registry Access

For ABAC-enabled source registries, use `az acr task create` with the `--source-acr-auth-id` flag to specify the identity the Task will use to authenticate to the source registry:

- `--source-acr-auth-id [system]` – Attaches a system-assigned managed identity.
- `--source-acr-auth-id $resourceId` – Attaches a user-assigned managed identity using its resource ID.

Once the identity is attached, assign it an ACR built-in role (optionally with ABAC conditions) to grant appropriate permissions.

#### Updating an Existing ACR Task to use an Identity for Source Registry Access

For ABAC-enabled source registries, use `az acr task update` with the same `--source-acr-auth-id` options to assign or change the identity used by the Task to authenticate to the source registry. As with new Tasks, follow up with the appropriate role assignment.

#### Removing Source Registry Access from an ACR Task

To explicitly prevent an ACR Task from authenticating with an ABAC-enabled source registry and block the Task's permissions to the source registry, set `--source-acr-auth-id none`. This disables the task's ability to perform any operations on the source registry and content within it.

> [!NOTE]
> Note: The `--auth-mode` flag is deprecated for ABAC-enabled registries and no longer supported for controlling ACR Task access.

### Effect on Quick Tasks, Quick Builds, and Quick Runs

> [!IMPORTANT]
> When a registry is configured with **"RBAC Registry + ABAC Repository Permissions"**, new Quick Tasks such as [Quick Builds](/cli/azure/acr#az-acr-build) and [Quick Runs](/cli/azure/acr#az-acr-run) will no longer have default data plane access to the source registry, including its images and artifacts within repositories. For Quick Tasks, you must use the caller's identity to authenticate the Quick Task to the source registry. The caller's identity must have the appropriate ACR built-in role to enable authentication and access to registry repositories.

#### Running a Quick Task with the Caller Identity for Source Registry Access

For ABAC-enabled source registries, use `az acr build` or `az acr run` with the `--source-acr-auth-id [caller]` option to specify the caller's identity as the identity the Quick Task will use to authenticate to the source registry. Before running the Quick Task, ensure that the caller's identity has the appropriate ACR built-in role to enable authentication and access to the source registry and its repositories.

#### Running a Quick Task without Source Registry Access

To run a Quick Task without any source registry access, use `az acr build` or `az acr run` with the `--source-acr-auth-id none` option. This disables the Quick Task's ability to perform any operations on the source registry and content within it.

## Next steps

* For a high-level overview of these built-in roles—including supported role assignment identity types, steps to perform a role assignment, and recommended roles for common scenarios—see [Azure Container Registry RBAC built-in roles](container-registry-rbac-built-in-roles-overview.md).
* To perform role assignments with optional Microsoft Entra ABAC conditions to scope role assignments to specific repositories, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).
* For a detailed reference of every ACR built-in role, including the permissions granted by each role, see the [Azure Container Registry roles directory reference](container-registry-rbac-built-in-roles-directory-reference.md).
* For more information on creating custom roles that meet your specific needs and requirements, see [Azure Container Registry custom roles](container-registry-rbac-custom-roles.md).
