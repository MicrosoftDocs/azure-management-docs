---
title: Manage workload orchestration resources as Bicep in Git
description: Learn how to manage Workload Orchestration resources (schemas, solution templates, and configuration templates) as Bicep templates in a Git repository, with automated validation and deployment via Azure Deployment Stacks.
author: nathmanish
ms.author: nathmanish
ms.topic: how-to
ms.date: 04/24/2026
ms.custom:
  - build-2025
# Customer intent: As a platform engineer, I want to manage workload orchestration resources as Bicep templates in Git, so that I can apply GitOps practices, enforce review through pull requests, and protect deployed resources from out-of-band changes.
---

# Manage workload orchestration resources in Git

Workload orchestration allows you to manage resources such as schemas, solution templates, and configuration templates as Bicep templates stored in a Git [repository](https://github.com/Azure/workload-orchestration-quickstart). Leveraging pre-configured GitHub Actions and [Azure Deployment Stacks](/azure/azure-resource-manager/bicep/deployment-stacks), you can validate changes through pull requests, deploy resources automatically on merge, and protect Git-managed resources from out-of-band changes.

This article describes how to set up your GitHub repository for managing the lifecycle of workload orchestration resources.

## How it works

Your GitHub repository uses the Bicep template `main.bicep` to declare all of your workload orchestration resources. Two GitHub Actions workflows enforce validation and synchronization between your repository and Azure account to manage the declared resources. The following are the steps in this workflow:

1. **Edit** `workload-orchestration/main.bicep` in a feature branch to add, update, or remove resource declarations.
1. **Open a pull request** to the `main` branch. The validation workflow runs automatically and:
   - Validates the Bicep templates against Azure.
   - Posts a validation result comment on the pull request.
1. **Merge** the pull request into `main`. The sync workflow deploys the resources to Azure by using a deployment stack with configurable deny settings.

## Repository structure

The repository is organized as follows:

```
workload-orchestration/
  config.yaml                  # Deployment stack settings (resource group, deny settings, and so on)
  main.bicep                   # Bicep template that declares all workload orchestration resources
.github/workflows/
  validate-bicep.yml           # Pull request gate: validates Bicep templates
  sync-bicep.yml               # Sync on merge to main, by using deployment stacks
```

The `main.bicep` file declares all workload orchestration resources&mdash;schemas, solution templates, configuration templates, and their versions&mdash;directly. Edit this file to define which resources to deploy.

## Prerequisites

- A GitHub repository where you store the Bicep templates and workflow files.
- An Azure subscription with permissions to create resource groups and deployment stacks.
- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](setup-workload-orchestration.md).

### Authenticate to Azure with OpenID Connect

The workflows authenticate to Azure by using [OpenID Connect (OIDC)](https://learn.microsoft.com/azure/developer/github/connect-from-azure) with a user-assigned managed identity that has federated credentials configured for GitHub Actions.

To configure authentication:

1. Create a user-assigned managed identity in Azure.
1. Configure federated credentials on the managed identity for both the `main` branch and pull requests in your GitHub repository. For step-by-step guidance, see [Use GitHub Actions to connect to Azure](https://learn.microsoft.com/azure/developer/github/connect-from-azure).
1. Store the following values as GitHub repository secrets:

   | Secret name | Value |
   |---|---|
   | `AZURE_CLIENT_ID` | The client ID of the user-assigned managed identity. |
   | `AZURE_TENANT_ID` | The Microsoft Entra tenant ID. |
   | `AZURE_SUBSCRIPTION_ID` | The target Azure subscription ID. |

### Assign Azure RBAC roles

Assign one of the following roles to the managed identity, based on the deny settings you intend to use:

| Role | When to use |
|---|---|
| **Azure Deployment Stack Owner** | Required when `denySettingsMode` is `denyWriteAndDelete` or `denyDelete`. This role can manage deployment stacks, including creating and deleting deny assignments. |
| **Azure Deployment Stack Contributor** | Use when `denySettingsMode` is `none` (the default). This role can manage deployment stacks but cannot create or delete deny assignments. |

For more information about deployment stack roles, see [Deployment stacks built-in roles](/azure/azure-resource-manager/bicep/deployment-stacks).

## Get started

To set up Git-based management of workload orchestration resources:

1. **Fork this [repository](https://github.com/Azure/workload-orchestration-quickstart)** into your GitHub account or organization.
1. **Set up Azure authentication** by following [Use GitHub Actions to connect to Azure](https://learn.microsoft.com/azure/developer/github/connect-from-azure). Then store `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, and `AZURE_SUBSCRIPTION_ID` as repository secrets.
1. **Configure deployment settings** in `workload-orchestration/config.yaml`:

   ```yaml
   resourceGroup: my-resource-group
   denySettingsMode: none
   denySettingsExcludedActions:
     - Microsoft.Edge/configTemplates/linkToHierarchies/action
     - Microsoft.Edge/configTemplates/unLinkFromHierarchies/action
   actionOnUnmanageResources: detach
   actionOnUnmanageResourceGroups: detach
   ```

   For details on each setting, see [Customize resource management](#customize-resource-management).

1. **Author your resources** in `workload-orchestration/main.bicep`. Declare schemas, solution templates, configuration templates, and their versions directly as Bicep resources.
1. **Push a branch** and open a pull request. The validation workflow runs automatically.
1. **Merge the pull request** into `main`. The sync workflow triggers and synchronizes your resources to Azure.

## Customize resource management

The deployment stack protects managed resources from out-of-band changes and controls their lifecycle. All settings are configured in `workload-orchestration/config.yaml`.

```yaml
resourceGroup: my-resource-group
denySettingsMode: none
denySettingsExcludedActions:
  - Microsoft.Edge/configTemplates/linkToHierarchies/action
  - Microsoft.Edge/configTemplates/unLinkFromHierarchies/action
actionOnUnmanageResources: detach
actionOnUnmanageResourceGroups: detach
```

### resourceGroup

The target Azure resource group for deployment. This setting is required.

### denySettingsMode

Controls whether Azure blocks direct, out-of-band changes to resources managed by the stack.

| Value | Behavior |
|---|---|
| `denyWriteAndDelete` | Blocks both modifications and deletions of managed resources outside the stack. |
| `denyDelete` | Blocks deletions but allows modifications. Use this option to allow operational changes (for example, scaling) while preventing accidental deletions. |
| `none` | Default. No restrictions. Resources can be freely modified or deleted outside the stack. |

### denySettingsExcludedActions

A list of Azure RBAC actions that are exempt from the deny assignment. These actions can be performed on managed resources even when deny settings are active.

The default value includes actions required for configuration template hierarchy operations. Git-based hierarchy linking isn't yet supported, so these actions must be allowlisted to allow linking and unlinking configuration templates to hierarchies from outside the deployment stack (for example, by using the Azure CLI or the portal):

```yaml
denySettingsExcludedActions:
  - Microsoft.Edge/configTemplates/linkToHierarchies/action
  - Microsoft.Edge/configTemplates/unLinkFromHierarchies/action
```

You can add more actions to the list as needed:

| Action | Reason to exclude |
|---|---|
| `Microsoft.Edge/configTemplates/linkToHierarchies/action` | Required. Git-based linking isn't yet supported and must be performed outside the stack. |
| `Microsoft.Edge/configTemplates/unLinkFromHierarchies/action` | Required. Git-based unlinking isn't yet supported and must be performed outside the stack. |
| `Microsoft.Resources/tags/write` | Allow tagging resources without going through the stack. |
| `Microsoft.Authorization/locks/write` | Allow adding resource locks directly. |
| `Microsoft.Insights/diagnosticSettings/write` | Allow configuring diagnostic settings outside the stack. |

### actionOnUnmanageResources

Controls what happens to *resources* when they're removed from the Bicep template and the stack is redeployed.

| Value | Behavior |
|---|---|
| `detach` | Default. Resources remain in Azure but are no longer tracked by the stack. |
| `delete` | Resources are deleted from Azure. |

### actionOnUnmanageResourceGroups

Controls what happens to *resource groups* when they're removed from the template.

| Value | Behavior |
|---|---|
| `detach` | Default. Resource groups remain in Azure but are no longer tracked. |
| `delete` | Resource groups are deleted from Azure. |

## Included Modules

The `modules/` folder contains optional, reusable Bicep modules that provide helper functions for defining Workload Orchestration resources. You can use them, extend them, or remove them as needed.

### `solutionTemplate.bicep`

Exports a `HelmChart` function that builds the component structure for a Helm-based solution template. Pass in a chart repo URL and version, and it returns the correctly shaped specification object.

**Usage:**
```bicep
import { HelmChart } from 'modules/solutionTemplate.bicep'

resource solutionTemplateVersion 'Microsoft.Edge/solutionTemplates/versions@2026-03-01' = {
  parent: solutionTemplate
  name: '1.0.0'
  properties: {
    configurations: $$'''
      schema:
        name: $${schema.name}
        version: $${schemaVersion.name}
      configs:
        ErrorThreshold: ${{$val(ErrorThreshold)}}
    '''
    specification: HelmChart('<repo url>', '<version>')
  }
}
```

You can add more helper functions to this module or create new modules for schemas and config templates as your project grows.

## Resource deployment scope

By default, the deployment stack is created at **resource group** scope and targets the resource group specified in `workload-orchestration/config.yaml`. All resources in `main.bicep` are deployed into this single resource group. The workflows use the [`azure/bicep-deploy@v2`](https://github.com/azure/bicep-deploy) action with `type: deploymentStack`.

You can change the scope to suit your requirements.

> [!NOTE]
> Deny settings (resource protection) only apply at the level of the deployment stack scope. For example, a resource-group-scoped stack only blocks changes to resources within that resource group. If you need protection across multiple resource groups or the entire subscription, use a higher scope accordingly.

The following table summarizes the configuration changes required for each scope:

| Scope | Bicep `targetScope` | `scope` in `bicep-deploy` action | `scope` on resources | Authentication change |
|---|---|---|---|---|
| **Resource group** (default) | *(none&mdash;default)* | `resourceGroup` | *(none needed&mdash;deploys directly)* | `subscription-id` in login |
| **Subscription** | `subscription` | `subscription` | `resourceGroup('<rg-name>')` | `subscription-id` in login |
| **Tenant** | `tenant` | `tenant` | `resourceGroup('<sub-id>', '<rg-name>')` | `allow-no-subscriptions: true` in login |
| **Management group** | `managementGroup` | `managementGroup` | `resourceGroup('<sub-id>', '<rg-name>')` | `allow-no-subscriptions: true` in login |

To change the scope, update the following:

1. The `targetScope` in `workload-orchestration/main.bicep`.
1. The resource `scope` in `main.bicep`. Add the resource group and subscription ID as needed.
1. The `scope:` value in the `azure/bicep-deploy@v2` steps in all workflow files.
1. For **subscription** scope, remove `resource-group-name` from workflow deploy steps. The resource group is set in the Bicep resource `scope` instead.
1. For **tenant** or **management group** scopes:
   - Add `management-group-id` to workflow deploy steps (for management group scope).
   - Change `azure/login` to use `allow-no-subscriptions: true` instead of `subscription-id`.

### Resource group scope (default)

No `targetScope` is needed. Resources deploy directly into the resource group defined in `config.yaml`. No workflow changes are required.

**main.bicep:**

```bicep
// no targetScope (defaults to resourceGroup)

resource schema 'Microsoft.Edge/schemas@2026-03-01' = {
  name: '<your-schema-name>'
  location: '<location>'
  properties: {}
}
```

**Workflow step (default):**

```yaml
- uses: azure/bicep-deploy@v2
  with:
    type: deploymentStack
    operation: create
    scope: resourceGroup
    resource-group-name: ${{ steps.config.outputs.rg }}
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    template-file: ./workload-orchestration/main.bicep
    # ... other inputs
```

### Subscription scope

**main.bicep:**

```bicep
targetScope = 'subscription'

resource schema 'Microsoft.Edge/schemas@2026-03-01' = {
  name: '<your-schema-name>'
  scope: resourceGroup('my-resource-group')
  location: '<location>'
  properties: {}
}
```

**Workflow step.** Change `scope` to `subscription` and remove `resource-group-name`:

```yaml
- uses: azure/bicep-deploy@v2
  with:
    type: deploymentStack
    operation: create
    scope: subscription
    subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}
    template-file: ./workload-orchestration/main.bicep
    # ... other inputs (no resource-group-name)
```

### Tenant scope

**main.bicep:**

```bicep
targetScope = 'tenant'

resource schema 'Microsoft.Edge/schemas@2026-03-01' = {
  name: '<your-schema-name>'
  scope: resourceGroup('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'my-resource-group')
  location: '<location>'
  properties: {}
}
```

**Workflow step.** Change `scope` to `tenant` and remove `resource-group-name` and `subscription-id`:

```yaml
- uses: azure/bicep-deploy@v2
  with:
    type: deploymentStack
    operation: create
    scope: tenant
    template-file: ./workload-orchestration/main.bicep
    # ... other inputs (no resource-group-name or subscription-id)
```

**Azure login.** Add `allow-no-subscriptions`:

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    allow-no-subscriptions: true
```

### Management group scope

**main.bicep:**

```bicep
targetScope = 'managementGroup'

resource schema 'Microsoft.Edge/schemas@2026-03-01' = {
  name: '<your-schema-name>'
  scope: resourceGroup('xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', 'my-resource-group')
  location: '<location>'
  properties: {}
}
```

**Workflow step.** Change `scope` to `managementGroup`, add `management-group-id`, and remove `resource-group-name` and `subscription-id`:

```yaml
- uses: azure/bicep-deploy@v2
  with:
    type: deploymentStack
    operation: create
    scope: managementGroup
    management-group-id: <your-management-group-id>
    template-file: ./workload-orchestration/main.bicep
    # ... other inputs (no resource-group-name or subscription-id)
```

**Azure login.** Add `allow-no-subscriptions`:

```yaml
- uses: azure/login@v2
  with:
    client-id: ${{ secrets.AZURE_CLIENT_ID }}
    tenant-id: ${{ secrets.AZURE_TENANT_ID }}
    allow-no-subscriptions: true
```

## Related content

- [Use GitHub Actions to connect to Azure](https://learn.microsoft.com/azure/developer/github/connect-from-azure)
- [Azure Deployment Stacks overview](/azure/azure-resource-manager/bicep/deployment-stacks)
- [Set up workload orchestration](setup-workload-orchestration.md)
