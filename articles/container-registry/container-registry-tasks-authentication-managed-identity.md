---
title: Managed Identity in ACR Task
description: Enable a managed identity for Azure Resources in an Azure Container Registry task to access other Azure resources, without passing credentials.
services: container-registry
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
author: chasedmicrosoft
ms.author: doveychase
ms.date: 10/31/2023
# Customer intent: As a cloud developer, I want to configure managed identities in Azure Container Registry tasks, so that I can securely access other Azure resources without managing credentials.
---

# Use an Azure-managed identity in ACR Tasks 

Enable a [managed identity for Azure resources](/azure/active-directory/managed-identities-azure-resources/overview) in an [ACR task](container-registry-tasks-overview.md), so the task can access other Azure resources, without needing to provide or manage credentials. For example, use a managed identity to enable a task step to pull or push container images to another registry.

In this article, you learn how to use the Azure CLI to enable a user-assigned or system-assigned managed identity on an ACR task. You can use the Azure Cloud Shell or a local installation of the Azure CLI. If you'd like to use it locally, version 2.0.68 or later is required. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI][azure-cli-install].

For illustration purposes, the example commands in this article use [az acr task create][az-acr-task-create] to create a basic image build task that enables a managed identity. For sample scenarios to access secured resources from an ACR task using a managed identity, see:

* [Cross-registry authentication](container-registry-tasks-cross-registry-authentication.md)
* [Access external resources with secrets stored in Azure Key Vault](container-registry-tasks-authentication-key-vault.md)

## Why use a managed identity?

A managed identity for Azure resources provides selected Azure services with an automatically managed identity in Microsoft Entra ID. You can configure an ACR task with a managed identity so that the task can access other secured Azure resources, without passing credentials in the task steps.

Managed identities are of two types:

* *User-assigned identities*, which you can assign to multiple resources and persist for as long as you want. User-assigned identities are currently in preview.

* A *system-assigned identity*, which is unique to a specific resource such as an ACR task and lasts for the lifetime of that resource.

You can enable either or both types of identity in an ACR task. Grant the identity access to another resource, just like any security principal. When the task runs, it uses the identity to access the resource in any task steps that require access.

## Steps to use a managed identity

Follow these high-level steps to use a managed identity with an ACR task.

### 1. (Optional) Create a user-assigned identity

If you plan to use a user-assigned identity, use an existing identity, or create the identity using the Azure CLI or other Azure tools. For example, use the [az identity create][az-identity-create] command. 

If you plan to use only a system-assigned identity, skip this step. You create a system-assigned identity when you create the ACR task.

### 2. Enable identity on an ACR task

When you create an ACR task, optionally enable a user-assigned identity, a system-assigned identity, or both. For example, pass the `--assign-identity` parameter when you run the [az acr task create][az-acr-task-create] command in the Azure CLI.

To enable a system-assigned identity, pass `--assign-identity` with no value or `assign-identity [system]`. The following example command creates a Linux task from a public GitHub repository which builds the `hello-world` image and enables a system-assigned managed identity:

```azurecli
az acr task create \
    --image hello-world:{{.Run.ID}} \
    --name hello-world --registry MyRegistry \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
    --file Dockerfile \
    --commit-trigger-enabled false \
    --assign-identity
```

To enable a user-assigned identity, pass `--assign-identity` with a value of the *resource ID* of the identity. The following example command creates a Linux task from a public GitHub repository which builds the `hello-world` image and enables a user-assigned managed identity:

```azurecli
az acr task create \
    --image hello-world:{{.Run.ID}} \
    --name hello-world --registry MyRegistry \
    --context https://github.com/Azure-Samples/acr-build-helloworld-node.git#main \
    --file Dockerfile \
    --commit-trigger-enabled false
    --assign-identity <resourceID>
```

You can get the resource ID of the identity by running the [az identity show][az-identity-show] command. The resource ID for the ID *myUserAssignedIdentity* in resource group *myResourceGroup* is of the form: 

```
"/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myUserAssignedIdentity"
```

> [!NOTE]
> If you are using an [ABAC-enabled source registry](container-registry-rbac-abac-repository-permissions.md), you must explicitly attach and set a managed identity that will be used by the Task to authenticate with an ABAC-enabled source registry using the new `--source-acr-auth-id` flag. Afterwards, you must perform a separate role assignment (with optional ABAC conditions) to grant this identity permissions to an ABAC-enabled source registry.
>
> For more information, see [effects of enabling ABAC on ACR Tasks, Quick Tasks, Quick Builds, and Quick Runs](container-registry-rbac-abac-repository-permissions.md#appendix-effects-of-enabling-abac-on-acr-tasks-quick-tasks-quick-builds-and-quick-runs).

### 3. Grant the identity permissions to access other Azure resources

Depending on the requirements of your task, grant the identity permissions to access other Azure resources. Examples include:

* Assign the managed identity a role with pull, push and pull, or other permissions to a target container registry in Azure. For a complete list of registry roles, see [Azure Container Registry Entra permissions and roles overview](container-registry-rbac-built-in-roles-overview.md). 
* Assign the managed identity a role to read secrets in an Azure key vault.

Use the [Azure CLI](/azure/role-based-access-control/role-assignments-cli) or other Azure tools to manage role-based access to resources. For example, run the [az role assignment create][az-role-assignment-create] command to assign the identity a role to the resource. 

The following example assigns a managed identity the permissions to pull from a container registry. The command specifies the *principal ID* of the task identity and the *resource ID* of the target registry.

The correct role to use in the role assignment depends on whether the registry is [ABAC-enabled or not](container-registry-rbac-abac-repository-permissions.md).

```azurecli
ROLE="Container Registry Repository Reader" # For ABAC-enabled registries. For non-ABAC registries, use AcrPull.
az role assignment create \
  --assignee <principalID> \
  --scope <registryID> \
  --role "$ROLE"
```

### 4. (Optional) Add credentials to the task

If your task needs credentials to pull or push images to another custom registry, or to access other resources, add credentials to the task. Run the [az acr task credential add][az-acr-task-credential-add] command to add credentials, and pass the `--use-identity` parameter to indicate that the identity can access the credentials. 

For example, to add credentials for a system-assigned identity to authenticate with the Azure container registry *targetregistry*, pass `use-identity [system]`:

```azurecli
az acr task credential add \
    --name helloworld \
    --registry myregistry \
    --login-server targetregistry.azurecr.io \
    --use-identity [system]
```

To add credentials for a user-assigned identity to authenticate with the registry *targetregistry*, pass `use-identity` with a value of the *client ID* of the identity. For example:

```azurecli
az acr task credential add \
    --name helloworld \
    --registry myregistry \
    --login-server targetregistry.azurecr.io \
    --use-identity <clientID>
```

You can get the client ID of the identity by running the [az identity show][az-identity-show] command. The client ID is a GUID of the form `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx`.

### 5. Run the task

After configuring a task with a managed identity, run the task. For example, to test one of the tasks created in this article, manually trigger it using the [az acr task run][az-acr-task-run] command. If you configured additional, automated task triggers, the task runs when automatically triggered.

## Next steps

In this article, you learned how to enable and use a user-assigned or system-assigned managed identity on an ACR task. For scenarios to access secured resources from an ACR task using a managed identity, see:

* [Cross-registry authentication](container-registry-tasks-cross-registry-authentication.md)
* [Access external resources with secrets stored in Azure Key Vault](container-registry-tasks-authentication-key-vault.md)


<!-- LINKS - Internal -->
[az-role-assignment-create]: /cli/azure/role/assignment#az_role_assignment_create
[az-identity-create]: /cli/azure/identity#az_identity_create
[az-identity-show]: /cli/azure/identity#az_identity_show
[az-acr-task-create]: /cli/azure/acr/task#az_acr_task_create
[az-acr-task-run]: /cli/azure/acr/task#az_acr_task_run
[az-acr-task-credential-add]: /cli/azure/acr/task/credential#az_acr_task_credential_add
[azure-cli-install]: /cli/azure/install-azure-cli
