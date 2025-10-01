---
title: Bulk Review, Publish, and Deploy with Workload Orchestration
description: Learn how to perform bulk review, publish, and deployment of workloads using workload orchestration in Azure Arc.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 09/30/2025
---

# Bulk review, publish, and deploy with workload orchestration

Workload orchestration allows you to bulk review, publish, and deploy a solution to multiple targets within the same cluster. If [external validation](external-validation.md) and [staging](how-to-stage.md) are enabled, they are automatically triggered as part of the bulk process. 

## Prerequisites

Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Perform bulk review

You can review solutions across multiple targets and apply target-specific configurations by running the bulk-review command in Azure CLI by running the following command:

```powershell
az workload-orchestration solution-template bulk-review --name "<solution-template-name>" --version "<solution-template-version>" --targets "@targets.json" --dependencies "@dependencies.json" --solution-instance-name <instance name> --solution-configuration "@configuration.yaml"
```

You need to provide a *targets.json* file that contains the list of targets that need to be reviewed. The file should be in the following format:

```json
[
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
        "solutionInstanceName": "<instance name1>" // alphanumeric, small case, no spaces, no special characters
    },
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
        "solutionInstanceName": "<instance name2>",
        "solutionConfiguration": "<configuration file path>.yaml"                
    }
]
```

The *configuration.yaml* file is used to set the configuration values for a target or multiple targets. For example, you can set the `temperature` and `errorThreshold` parameters for a target:

```yaml
temperature: 100
errorThreshold: 5
```

The `--solutionInstanceName` and `--solutionConfiguration` parameters can be given for each target as part of targets.json file or as part of CLI input. If specified in CLI input, it applies to all targets that don't have the parameter defined in targets.json. If no instance name is provided, the service defaults to using the solution template name as the instance name.

The `--dependencies` parameter is optional and only required if the solution is a shared app with dependencies. The *dependencies.json* file is in the following format:

```json
[
    {
        "solutionVersionId": "/subscriptions/$subId/resourceGroup/$rg/providers/microsoft.edge/targets/<target>/solutions/<solution name>/versions/<version>",
    }
]
```

### Bulk review output

On successful review, the CLI returns the list of reviewed targets under `reviewedTargets`, as shown:

```json
{
    "solutionTemplateVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/<solution-template>/versions/<version>",
    "reviewedTargets": [
      {
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<version>",
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/<target1>"
      },
      {
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/<target2>/solutions/<solution>/versions/<version>",
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/<target2>"
      }
    ]
}
```

In case of failure, the CLI returns the list of targets that succeeded (if any) under `reviewedTargets` and the targets that failed under `failedTargets`, along with target ID and error details.

## Perform bulk publishing

You can perform bulk publishing of a solution to multiple targets using the Azure CLI or the workload orchestration portal.

### [CLI](#tab/cli)

To perform bulk publishing of reviewed targets, run the following command in Azure CLI:

```powershell
az workload-orchestration solution-template bulk-review --name "<solution-template-name>" --version "<solution-template-version>" --targets "@targets.json" --dependencies "@dependencies.json"
```

You need to provide a *targets.json* file that contains the list of targets along with reviewed solution version ID:

```json
[
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name/solutions/<solution name>/versions/<version>"
    },
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name/solutions/<solution name>/versions/<revision>"                
    }
]
```

To perform a bulk-publish operation on a bunch of targets which contains a mix of targets that have been reviewed as well as targets for which you wish to bypass the review process, the command is as follows:

```powershell
az workload-orchestration solution-template bulk-review --name "<solution-template-name>" --version "<solution-template-version>" --targets "@targets.json" --dependencies "@dependencies.json" --solution-instance-name <instance name> --solution-configuration "@configuration.json"
```

The targets.json file in this case has the following format:

```json
[
    // Not reviewed
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
        "solutionInstanceName": "<instance-name>"
    },
    
    // Reviewed
    {
        "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
        "solutionVersionId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name/solutions/<solution name>/versions/<revision>"                
    }
]
```

All the other inputs for `bulk-publish` command in case of both reviewed and non-reviewed targets are the same as those of `bulk-review` command.

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

### Bulk deployment script

The script [bulk_deployment.ps1](https://github.com/Azure/workload-orchestration/blob/main/bulk_deployment.ps1) enables you to deploy an application to multiple targets in a single step, eliminating the need to run multiple commands for bulk review, publishing, and deployment. 

To use the script, provide the following details in an *input.json* file:

```json
{
    "resourceGroup": "$rg",
    "subscriptionId": "$subId",
    "solutionTemplateName": "<solution-template-name>",
    "solutionTemplateVersion": "<solution-template-version>",
    "targets": [
        {
            "targetId": "
/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
            "solutionInstanceName": "<instance name1>",   
        },
        {            
            "targetId": "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
            "solutionInstanceName": "<instance name2>",
            "solutionConfiguration": "<target2 configuration file path>.yaml"   
        }        
    ],
    "solutionInstanceName": "<instance name>",
    "solutionConfiguration": "<configuration for all targets>.yaml"    
    "skipReview": true
}
```

The `skipReview` flag can be used to bypass the review process and publish the solution directly. In case this parameter is set to `false`, the script executes the review command and downloads the reviewed targets to a file. This allows inspecting the targets before proceeding further.

Once the input file is prepared, run the script using:

```powershell
bulk_deployment.ps1 "input.json"
```

## Related content

- [Diagnose of problems](diagnose-problems.md)
- [External validation](external-validation.md)
- [Stage solution before deployment](how-to-stage.md)
- [Service groups](service-group.md)
