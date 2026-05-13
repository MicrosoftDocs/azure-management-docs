---
title: Delete Resources in Workload Orchestration
description: Learn how to delete the resources created with workload orchestration
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 06/24/2025
ms.custom:
  - build-2025
# Customer intent: As an IT administrator, I want to delete workload orchestration resources using CLI commands, so that I can manage and free up my cloud environment effectively.
---


# Delete resources in workload orchestration

This article details how to delete or uninstall any workload orchestration resources in Azure, and its cascading impact on other dependent resources, if any.

## Delete all resources

Deleting a resource group results in deletion of all Azure resources within the same.

## Uninstall a solution

Any installed solution can be uninstalled from a target using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration target uninstall --resource-group "$rg"  --target-name "$targetName" --solution-template-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target uninstall --resource-group $rg --target-name $targetName --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName
```

***

## Delete a solution revision

Delete a solution revision for a target if it's not installed using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration target remove-revision --resource-group "$rg" --solution-template-version "$version" --target-name "$targetName" --solution-template-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName" --solution-version 1.0.0
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target remove-revision --resource-group $rg --solution-template-version $version --target-name $targetName --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName --solution-version 1.0.0
```

***

## Delete a target

Delete a target and all its child resources using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration target delete --resource-group "$rg" --target-name "$childName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target delete --subscription $subId --resource-group $rg --target-name $childName
```
***

Target deletion automatically uninstalls all running workloads (instances, solutions, namespaces, and the target itself) from the cluster. The corresponding cloud resources (Target, TargetSolution, TargetSolutionVersion and Instances) are also deleted successfully.

A deleted target can be recreated with the same name successfully. Once the target is re-linked to the relevant common config templates, users can once again start deploying solutions post re-configuring the parameter values.

> [!NOTE]
> Target deletion without the --force-delete flag fails when underlying cluster is in the *stopped* state, due to cluster connectivity issues. Add the argument `--force-delete true` to successfully delete the target along with all associated cloud resources. Workloads running on the cluster won't be impacted.

***

## Delete a solution template version

Delete solution template version across all targets using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template remove-version --resource-group "$rg" --solution-template-name "$appName1" --version "$appVersion"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template remove-version --subscription $subId --resource-group $rg --solution-template-name $appName1 --version $appVersion
```
***

The deletion succeeds even if the solution template version is being referenced by a target solution version. The solution version remains unaffected.
You can verify the deletion of the solution template version by listing all revisions of a solution deployed on a target.

### [Bash](#tab/bash)

```bash
az workload-orchestration target solution-revision-list --target-name "$childName" --resource-group "$rg" --solution-template-name "<solution-version-id>"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target solution-revision-list --target-name $childName --resource-group $rg --solution-template-name "<solution-version-id>"
```

***

## Delete a solution template

Delete a solution template across all targets using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration solution-template delete --resource-group "$rg" --solution-template-name "$appName1"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration solution-template delete --subscription $subId --resource-group $rg --solution-template-name $appName1
```

***

Deletion of solution template succeeds even if deployed instances refer to its solution template version. All associated solution template versions also get deleted automatically. Any solutions already reviewed or published, referring to the deleted solution template, can still be deployed. No running workloads are impacted.

Any deleted solution template can be recreated successfully with the same name, referencing the same or a different schema. However, solution parameters need to be re-configured to deploy solutions based on the template.

***

## Delete a schema

Delete a schema and all its versions using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration schema delete --subscription "$subId" --resource-group "$rg" --schema-name "$schemaName"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration schema delete --subscription $subId --resource-group $rg --schema-name $schemaName
```

Deleting a schema removes all associated schema versions, and any solution templates referencing the schema can't be reviewed.


***

## Delete a configuration template

Delete a configuration template and all its versions using the following command:

### [Bash](#tab/bash)

```bash
az workload-orchestration config-template delete --subscription "$subId" --resource-group "$rg" --config-template-name "$appConfig"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration config-template delete --subscription $subId --resource-group $rg --config-template-name $appConfig
```

***

Deleting a configuration template removes all its versions and associated hierarchy linkages. A deleted template can be recreated successfully with the same name, referencing the same or a different schema. However, it needs to be linked to hierarchies or targets and configured again. If the recreated configuration template references a different schema and a solution template inherits that schema, the `az workload-orchestration target review` command is expected to fail since the schema properties changed.

***
