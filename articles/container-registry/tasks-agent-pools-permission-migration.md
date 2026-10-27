---
title: Migrate ACR task agent pool permissions
description: Identify affected Azure Container Registry task requests, assign agent pool read permission, validate access, and troubleshoot authorization failures.
ms.topic: how-to
author: KumudD
ms.author: kumud
ms.date: 10/19/2026
ms.custom: devx-track-azurecli
ms.service: azure-container-registry
ai-usage: ai-generated
# Customer intent: As a registry administrator, I want to identify and update callers that use dedicated task agent pools so that their requests continue to work after the authorization change.
---

# Migrate ACR task agent pool permissions

Starting **June 1, 2027**, Azure Container Registry (ACR) Tasks API requests that specify `agentPoolName` require the caller to have `Microsoft.ContainerRegistry/registries/agentpools/read` permission on the specified agent pool, in addition to the permissions already required for the task operation.

Use this article to identify affected requests, update permissions before enforcement, validate access, and troubleshoot failures. For information about dedicated agent pools, see [Run a task on a dedicated agent pool](tasks-agent-pools.md).

> [!IMPORTANT]
> No action is required for this change if the caller already has effective agent pool read permission on the selected pool, for example through **Container Registry Tasks Contributor** at registry scope. See [Built-in roles that already grant agent pool read](#built-in-roles-that-already-grant-agent-pool-read). Callers without effective read permission need an update before enforcement; otherwise, affected requests fail authorization.

## Understand which requests are affected

The following table lists the affected API operations and the fields to check in each JSON request body.

| Operation | Existing operation permission | Agent pool field to inspect |
| --- | --- | --- |
| Schedule a run. | `Microsoft.ContainerRegistry/registries/scheduleRun/action` | `agentPoolName` |
| Create or update a task run. | `Microsoft.ContainerRegistry/registries/taskRuns/write` | `properties.runRequest.agentPoolName` |
| Create or update a task. | `Microsoft.ContainerRegistry/registries/tasks/write` | `properties.agentPoolName` |

For each request that specifies a pool, identify the user, CI/CD service principal, or automation managed identity that submits the management API request. Use this identity when checking or updating agent pool read access. The task's runtime identity controls access to resources during execution.

The extra check applies only to requests that specify `agentPoolName`. Existing task operation and image access requirements continue to apply to all requests.

### Built-in roles that already grant agent pool read

The following built-in roles in the Azure public cloud grant `Microsoft.ContainerRegistry/registries/agentpools/read` through their control plane permissions, either explicitly or through a wildcard, after accounting for `notActions`. If your calling identity has an effective assignment of one of these roles at a scope that covers the selected pool, **no permission update is required for this change**.

| Built-in role | Control plane action granting agent pool read |
| --- | --- |
| Container Registry Tasks Contributor | `Microsoft.ContainerRegistry/registries/agentpools/read` |
| App Compliance Automation Administrator | `*/read` |
| App Compliance Automation Reader | `*/read` |
| Azure AI Administrator | `Microsoft.ContainerRegistry/registries/*` |
| Azure Resilience Management Drills Target Resource Administrator | `*/read` |
| Azure Resilience Management Drills Target Resource Contributor | `*/read` |
| Contributor | `*` |
| Log Analytics Contributor | `*/read` |
| Log Analytics Reader | `*/read` |
| Managed Application Contributor Role | `*/read` |
| Managed Application Operator Role | `*/read` |
| Managed Application Publisher Operator | `*/read` |
| Monitoring Contributor | `*/read` |
| Monitoring Policy Contributor | `*/read` |
| Monitoring Reader | `*/read` |
| Owner | `*` |
| Reader | `*/read` |
| Resource Policy Contributor | `*/read` |
| Role Based Access Control Administrator | `*/read` |
| User Access Administrator | `*/read` |

Assign pool read permission at the pool, registry, or a parent scope that covers the pool. When using **Reader** to add pool read access, retain the caller's existing operation permissions to schedule runs and create or update tasks or task runs.

Built-in role definitions can change and can differ between Azure clouds. Use the [pool-read check](#check-effective-permission-on-each-pool) to confirm effective access before deciding whether an update is needed. See the [Azure built-in roles reference](/azure/role-based-access-control/built-in-roles) for role details. Choose a role that matches the caller's task and agent pool requirements.

Review custom task roles for `Microsoft.ContainerRegistry/registries/agentpools/read` in their `actions`, accounting for wildcards and `notActions` exclusions. Update callers whose effective assignments lack this permission.

## Prerequisites

To inspect and update access, you need:

- The subscription, resource group, registry, and agent pool names.
- The object ID of each calling identity if you need to inspect or assign roles. For a service principal, use its object ID. For a managed identity, use its principal ID.
- Task read access to inventory stored tasks. Role inspection requires role assignment and definition read permissions. An access administrator can help inspect roles and apply updates.
- For custom role changes, `Microsoft.Authorization/roleDefinitions/write` on all assignable scopes of the role. For role assignments, `Microsoft.Authorization/roleAssignments/write` at the assignment scope. See [Create or update custom roles](/azure/role-based-access-control/custom-roles-cli) and [Assign Azure roles](/azure/role-based-access-control/role-assignments-cli).
- Azure CLI in a Bash environment, such as [Azure Cloud Shell](/azure/cloud-shell/quickstart).

Set the following variables for the registry and pool you're checking. You only need `principalObjectId` for role inspection or assignment:

```azurecli
subscriptionId="<subscription-id>"
resourceGroup="<resource-group>"
registryName="<registry-name>"
agentPoolName="<agent-pool-name>"
principalObjectId="<caller-object-id>"

az account set --subscription "$subscriptionId"

registryId="/subscriptions/$subscriptionId/resourceGroups/$resourceGroup/providers/Microsoft.ContainerRegistry/registries/$registryName"
agentPoolId="$registryId/agentPools/$agentPoolName"
```

Repeat the checks for each subscription, registry, pool, and calling identity your workflows use.

## Detect callers that need a permission update

Identify named-pool usage, then check each caller's effective permissions on the selected pool.

### Inventory task and request configuration

Review your scripts, CI/CD pipelines, SDK calls, and infrastructure-as-code definitions for the fields in the [affected requests table](#understand-which-requests-are-affected). Include Azure CLI commands that use `--agent-pool`, such as quick builds, quick runs, and task creation or updates.

As an administrator with task read access, list stored tasks that specify an agent pool:

```azurecli
az acr task list \
    --registry "$registryName" \
    --resource-group "$resourceGroup" \
    --query "[?agentPoolName].{task:name,agentPool:agentPoolName}" \
    --output table
```

Also inventory task-run resources:

```azurecli
az acr taskrun list \
    --registry "$registryName" \
    --resource-group "$resourceGroup" \
    --output json
```

Inspect the deployment templates or original request configuration for each task run to identify `properties.runRequest.agentPoolName`. Review direct `scheduleRun` request bodies to include quick runs and other on-demand requests in your inventory.

For each workflow, record the operation, target pool resource ID, and calling principal object ID. Include infrequent, scheduled, and deployment-time workflows. Review workflow definitions even when the stored task and task-run lists are empty.

### Check effective permission on each pool

Sign in as the identity that submits your task requests, or run this command in the pipeline or automation environment that uses that identity.

```azurecli
az acr agentpool show \
    --subscription "$subscriptionId" \
    --registry "$registryName" \
    --resource-group "$resourceGroup" \
    --name "$agentPoolName" \
    --query id \
    --output tsv
```

> [!NOTE]
> This Azure CLI command also requires `Microsoft.ContainerRegistry/registries/read` on the parent registry. **Container Registry Tasks Contributor** at registry scope includes both registry read and agent pool read. For a custom-role caller with a registry-read authorization error, ask an access administrator to check pool access through the role-assignment inspection described below.

| Result | Next step |
| --- | --- |
| The command succeeds and returns the selected pool's resource ID. | The caller has the required agent pool read access. Continue to [validate the workflow](#validate-the-migration). |
| The command fails with an authorization error for `Microsoft.ContainerRegistry/registries/agentpools/read` on the selected pool. | Have an authorized administrator [add the missing permission](#assign-agent-pool-read-permission), then repeat the check. |
| The command fails for another reason. | Resolve the reported error, verify the sign-in, subscription, registry, and pool names, and repeat the check to assess agent pool read access. |

If you can't test as the calling identity, ask an access administrator to [inspect its role assignments](/azure/role-based-access-control/role-assignments-list-cli) at the pool scope. Include inherited and group-based access, and account for deny assignments. Compare built-in roles with the [role list](#built-in-roles-that-already-grant-agent-pool-read); for custom roles, check `actions` and `notActions` for the required read permission. Use a complete view of effective access to determine whether a permission update is needed.

## Assign agent pool read permission

For callers that need a permission update, we recommend **Container Registry Tasks Contributor** at registry scope for ACR Tasks management. If you're comfortable managing custom roles and want to retain a narrower permission set, you can add the specific agent pool read permission to the caller's existing custom role.

Keep the caller's existing task permissions. Have an authorized administrator apply one of the following options.

### Option 1: Assign a built-in role

Assign the recommended built-in **Container Registry Tasks Contributor** role at registry scope. It includes task management permissions, `Microsoft.ContainerRegistry/registries/agentpools/read`, and `Microsoft.ContainerRegistry/registries/read`. The following command assigns it to the calling service principal or managed identity:

```azurecli
az role assignment create \
    --subscription "$subscriptionId" \
    --assignee-object-id "$principalObjectId" \
    --assignee-principal-type ServicePrincipal \
    --role "Container Registry Tasks Contributor" \
    --scope "$registryId"
```

For a user or group, use its object ID and change `--assignee-principal-type` to `User` or `Group`. Repeat for each registry the caller needs.

This role grants task and agent pool management permissions. Review that access before assigning it. Use [Option 2](#option-2-update-your-existing-custom-role) to retain a narrower custom permission set.

### Option 2: Update your existing custom role

If you're comfortable managing custom roles and already use a custom task role for the caller, add only `Microsoft.ContainerRegistry/registries/agentpools/read` to that role's control plane actions. Keep its other permissions unchanged. Export the current definition:

```azurecli
az role definition list \
    --subscription "$subscriptionId" \
    --name "<custom-role-name-or-id>" \
    --query "[0]" \
    --output json > acr-task-role.json
```

Check that `acr-task-role.json` contains the correct custom role, then add the action to its existing `permissions` block's `actions` array. Keep the role ID, other permissions, and assignable scopes unchanged. Check `notActions` to make sure the added action is allowed. Review how the change affects identities that already use this role before applying the update:

```azurecli
az role definition update \
    --subscription "$subscriptionId" \
    --role-definition acr-task-role.json
```

The caller's role assignment must cover the pool. For assignments scoped to a task or task run, use [Option 1](#option-1-assign-a-built-in-role) to grant access at registry scope.

For more information, see [ACR custom roles](container-registry-rbac-custom-roles.md).

## Validate the migration

Test by using the identity that sends the original request. Allow time for role and assignment changes to take effect, and then refresh the caller's sign-in or access token before trying again. For details, see [Troubleshoot Azure RBAC](/azure/role-based-access-control/troubleshooting).

1. Repeat the [pool-read check](#check-effective-permission-on-each-pool) after any permission update.
1. In a test environment, send the original request again by using the same caller and pool. Check that the request includes the `agentPoolName` field listed in the affected requests table. Repeat for each API operation your workflows use, including scheduling runs and creating or updating tasks or task runs.
1. Confirm that the operation is accepted and, for a run request, that the run proceeds on the intended pool. Resolve any separate task, image, source, or runtime permissions required by your workload.
1. Repeat the check for each calling identity and pool.

> [!NOTE]
> Before June 1, 2027, confirm agent pool read access by using the pool-read check and test the workflow to prepare for the new authorization requirement. After enforcement applies, repeat the affected requests to confirm end-to-end access.

Use these expected outcomes when validating after enforcement:

| Scenario | Expected result |
| --- | --- |
| A request in the affected requests table specifies a pool, and the caller has its existing operation permission but lacks read access to that pool. | The request fails authorization. Use an isolated test identity for this test and keep production access unchanged. |
| The same request's caller also has read access to the selected pool. | The additional agent pool authorization check passes. Other operation requirements still apply. |
| A request omits `agentPoolName`. | Existing operation requirements apply, with no additional agent pool read check. |

## Troubleshoot authorization failures

After enforcement, affected requests fail authorization when the caller lacks read access to the specified pool. Update the caller's permissions before retrying the request. Authorization failures prevent the requested task creation, task update, or run submission.

The [pool-read check](#check-effective-permission-on-each-pool) confirms the caller's read access to the agent pool.

If an affected request fails:

1. Record the returned HTTP status, error code, and complete error message. Check whether the failure identifies agent pool read access and the pool resource ID. An error for a different action or resource needs separate investigation.
1. Verify the principal object ID and tenant of the identity that submitted the request, along with the subscription and target pool.
1. Recheck direct, inherited, and group-based access on that pool, including role exclusions and deny assignments.
1. Have an authorized administrator add or correct the assignment. Allow time for the change to take effect, refresh the caller's credentials, and send the original request again.
1. If pool read succeeds and the operation still fails, review the original operation permissions and the returned error. Limit any permission update to the action and resource identified by the error.

### Contact support

If the failure persists, [open an ACR support request](https://aka.ms/acr/support/create-ticket). Include:

- The subscription, registry resource ID, and agent pool resource ID.
- The calling principal object ID and tenant ID.
- The API operation, API version, UTC failure time, and request or correlation ID from the response.
- The HTTP status, error code, and complete error message.
- A copy of the request with secrets removed, showing the agent pool field, the role definitions and assignment scopes you checked, and the steps you used to check access.

Remove access tokens, client secrets, registry passwords, source credentials, and other sensitive values before sharing this information with support.

## Next steps

- [Run a task on a dedicated agent pool](tasks-agent-pools.md).
- [Manage ACR custom roles](container-registry-rbac-custom-roles.md).
- [Review ACR built-in roles](container-registry-rbac-built-in-roles-directory-reference.md).
