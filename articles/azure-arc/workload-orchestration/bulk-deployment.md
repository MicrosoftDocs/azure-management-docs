---
title: Bulk Deployment with Workload Orchestration
description: Learn how to perform bulk deployment of workloads using workload orchestration in Azure Arc.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/24/2025
---

# Bulk publish and deploy with workload orchestration

Workload orchestration allows you to bulk publish and deploy a solution to multiple targets within the same cluster. If [external validation](external-validation.md) and [staging](how-to-stage.md) are enabled, they are automatically triggered as part of the bulk process. 

## Prerequisites

Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Perform bulk publishing

You can perform bulk publishing of a solution to multiple targets using the Azure CLI or the workload orchestration portal.

### [CLI](#tab/cli)

To perform a bulk publishing, run `bulk-publish` command in the Azure CLI.

```powershell
az workload-orchestration solution-template bulk-publish -g $rg --targets "@targets.json" --name "<solution-template-name>" --version "<solution-template-version>" --solution-dependencies "@dependencies.json"
```

> [!NOTE]
> The `--solution-dependencies` parameter is only required if the solution has dependencies. See the [Bulk publishing with dependencies](#bulk-publishing-with-dependencies) section.

You need to provide a *targets.json* file that contains the list of targets where you want to publish the solution. The file should be in the following format:

```json
[
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
        "solutionInstanceName": "<instance name1>" // alphanumeric, small case, no spaces, no special characters
    },
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
        "solutionInstanceName": "<instance name2>" // alphanumeric, small case, no spaces, no special characters
    }
]
```

The `solutionInstanceName` parameter can be given for each target as part of target-file.json file or as part of CLI input. If you provide the solution instance name in both the target file and the CLI input, the solution instance name from the target file takes precedence. `solutionInstanceName` should be alphanumeric, in lowercase, and should not contain spaces or special characters.

> [!IMPORTANT]
> You need to have access to the target clusters on which the solution is deployed. 

### [WO portal](#tab/woportal)

You can perform bulk publishing of a solution to multiple targets using the **Configure tab** in workload orchestration portal. 

- In the **Solutions subtab**, when you configure the parameters of the solution, select the targets where you want to publish the solution. For more information, see [Configure solution parameters](configure.md#configure-solution-parameters).
- In the **Published Solutions subtab**, you can publish a solution to more targets after the solution is published. For more information, see [Publish a solution to more targets](configure.md#publish-a-solution-to-more-targets).

***

### Bulk publishing output

If the bulk publish is successful, the CLI returns the list of published targets under `publishedTargets`. If any target has [external validation enabled](external-validation.md), the CLI also returns the list of targets that are pending external validation under `externalValidationPending`. These targets can't be deployed until external validation is completed. The CLI output looks like this:

```json
{
"solutionTemplateVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/<solution-template>/versions/<version>",
  "publishedTargets": [
    {
      "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<instance>",
      "targetId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>"
    }
],
"externalValidationPending": [
    {
      "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target2>/solutions/<solution>/versions/<instance>",
      "targetId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target2>"
    },
      ]
}
```

If the bulk publish isn't successful, there are two types of failures: complete failure, when all the targets fail to publish, and partial failure, when some targets succeed to publish while others fail. In a partial failure, the CLI returns the list of targets that succeeded, `publishedTargets` and the targets that failed, `failedTargets`. In a complete failure, the CLI returns a message indicating that all targets failed to publish. 

### Bulk publishing with dependencies

If your solution has dependencies on other solutions, you also need to provide a *dependencies.json* file that contains the list of dependencies for the solution. The file should be in the following format:

```json
[
    {
        "solutionVersionId": "/subscriptions/$subId/resourceGroup/$rg/providers/microsoft.edge/targets/<target>/solutions/sharedApp/versions/shared-app-1.0.0.1",
    }
]
```

In this case, workload orchestration creates a new revision of the dependency and publishes it. For example, if the solution has the dependency on shared-app-1.0.0.1, when bulk publishing triggers, workload orchestration creates new revision shared-app-1.0.0.2 and use it for publishing.

Once publish succeeds, you can find the dependencies of solution by using `az rest` command:

```powershell
az rest -u "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<revision>?api-version=2025-06-01" -m GET 
```

In the output, search for the property called `solutionDependencies`, which contains all the dependency solutions.

## Perform bulk deployment

Currently bulk deployment is only supported via CLI. To perform a bulk deployment, run `bulk-deploy` command in the Azure CLI. 

```powershell
az workload-orchestration solution-template bulk-deploy --targets "@target.json" --version "<solution template version>" --name "<solution-name>" -g $rg --solution-dependencies "@dependencies.json"
```

You need to provide a *targets.json* file that contains the list of targets where you want to publish the solution. The file should be in the following format:

```json
[
    {
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<instance>"
    }
]
```

> [!NOTE]
> The `--solution-dependencies` parameter is only required if the solution has dependencies. For more information, see the [previous section](#bulk-publishing-with-dependencies) on how to create the *dependencies.json* file.

### Bulk deployment output

If the bulk publish is successful, the CLI returns the list of published targets under `deployedTargets`. 

```json
{
"solutionTemplateVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/<solution-template>/versions/<version>",
  "deployedTargets": [
    {
      "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<instance>",
      "targetId": "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>"
    }
]
}
```

If the bulk deployment isn't successful, there are two types of failures: complete failure, when all the targets fail to deploy, and partial failure, when some targets succeed to deploy while others fail. In a partial failure, the CLI returns the list of targets that succeeded, `deployedTargets` and the targets that failed, `failedTargets`. In a complete failure, the CLI returns a message indicating that all targets failed to deploy. You can retry the deployment for the failed targets by running the `bulk-deploy` command again with the same parameters.

## Related content

- [Diagnose of problems](diagnose-problems.md)
- [External validation](external-validation.md)
- [Stage solution before deployment](how-to-stage.md)
- [Service groups](service-group.md)
