---
title: Bulk Review, Publish, and Deploy with Workload Orchestration
description: Learn how to perform bulk review, publish, and deployment of workloads using workload orchestration in Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 09/30/2025
---

# Bulk deployment with workload orchestration

Workload orchestration allows you to deploy a solution to multiple targets within the same or different Azure Arc-connected clusters. If [external validation](external-validation.md) and [staging](how-to-stage.md) are enabled, they are automatically triggered as part of the bulk process.

## Prerequisites

Set up the required Azure resources for workload orchestration by referring to [Set up workload orchestration](set-up-workload-orchestration.md).

## Perform bulk review

The Review step ensures the configuration values obey all schema rules and generates a solution version based on the solution template. This step is optional and you can directly [publish the solution](bulk-deployment.md#perform-bulk-publishing).

> [!NOTE]
> You can directly deploy an application to intended targets in a single step using the [bulk_deployment.ps1](https://github.com/Azure/workload-orchestration/blob/main/bulk_deployment.ps1) script, eliminating the need to run multiple commands for bulk review, publishing, and deployment.

To review a solution across multiple targets and apply target-specific configurations, run the following command:

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

## Perform bulk publishing

The publishing step generates the final configuration of the solution after it's validated and approved, created by combining the schema, configuration template, and solution Helm chart.

You can perform bulk publishing of a solution to multiple targets using the Azure CLI or the workload orchestration portal.

### [CLI](#tab/cli)

To perform bulk publishing of reviewed targets, run the following command in Azure CLI:

```powershell
az workload-orchestration solution-template bulk-publish --name "<solution-template-name>" --version "<solution-template-version>" --targets "@targets.json" --dependencies "@dependencies.json"
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

To perform a bulk-publish operation on a mix of targets involving targets that have been reviewed as well as targets for which you wish to bypass the review process, the command is as follows:

```powershell
az workload-orchestration solution-template bulk-publish --name "<solution-template-name>" --version "<solution-template-version>" --targets "@targets.json" --dependencies "@dependencies.json" --solution-instance-name <instance name> --solution-configuration "@configuration.json"
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


### [Portal](#tab/portal)

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Click on **Configure Solutions** on the left. The **Solutions** tab shows the status of all solutions deployed or pending deployment in your environment.

     :::image type="content" source="./media/configure-solutions.png" alt-text="Screenshot of the Configure tab showing how to apply filters1." lightbox="./media/configure-solutions.png":::

1. Search and select the name of your solution with configuration status *Configuration pending* and click on **Configure and publish**. 

    :::image type="content" source="./media/configure-solution-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to select a solution to configure it." lightbox="./media/configure-solution-1.png":::

1. Select the targets for which you want to set the solution configuration. Click on **Next**.

    :::image type="content" source="./media/configure-solution-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure a solution and disable autopublish." lightbox="./media/configure-solution-2.png":::

1. In the **Configure target** step, you can set common configurations for all targets or click on the **custom target value** icon to set custom configuration values for selective targets. You can also click on **Previous Versions** to view configuration for previously deployed versions of this solution. Once done, click on **Next**.

    :::image type="content" source="./media/configure-solution-3.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the parameters to configure the targets." lightbox="./media/configure-solution-3.png":::
    :::image type="content" source="./media/configure-solution-3-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the parameters to configure the targets1." lightbox="./media/configure-solution-3-1.png":::

1. Review the final configurations and click on **Publish** to create a new revision of configuration values for the selected targets. Once completed, the new solution version (or revision) is published for each target where the configurations were resolved successfully.

    :::image type="content" source="./media/configure-solution-7.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to publish the configuration of a solution target." lightbox="./media/configure-solution-7.png":::

***

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

Once publish succeeds, you can find the dependencies of solution using:

```azurecli
az rest -u "/subscriptions/$subId/resourceGroups/$rg/Microsoft.Edge/targets/<target1>/solutions/<solution>/versions/<revision>?api-version=2025-06-01" -m GET 
```

In the output, search for the property called `solutionDependencies`, which contains all the dependency solutions.

## Perform bulk deployment

Follow these steps to deploy a solution to multiple targets:

### [CLI](#tab/cli)

Run the following command: 

```azurecli
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

### [Portal](#tab/portal)

1. Click on the **Deploy** tab and switch to solution view to view the applicable solutions and their statuses.

    :::image type="content" source="./media/deploy-1.png" alt-text="Screenshot of the Deploy tab showing how to click on a target2." lightbox="./media/deploy-1.png":::

1. Select a solution which has 1 or more targets available to deploy to.

    :::image type="content" source="./media/deploy-2.png" alt-text="Screenshot of the Deploy tab showing how to click on a target3." lightbox="./media/deploy-2.png":::

1. Choose the targets in **Publish Completed** state and click on **Deploy Solution**. If asked to confirm, click on **Confirm** to proceed.

    :::image type="content" source="./media/deploy-3.png" alt-text="Screenshot of the Deploy tab showing how to deploy a solution." lightbox="./media/deploy-3.png":::

1. To view the detailed status of your deployment, click on the notification icon at the top right and click on **Show in event logs**.

    :::image type="content" source="./media/deploy-6.png" alt-text="Screenshot of the Deploy tab showing the deployment status." lightbox="./media/deploy-6.png":::

1. Click on the respective **Event name** to open the **Status details** pane showing all the intermediate steps of the operation, along with date and time of completion and the user who initiated it. Details of shared app dependencies associated with the current deployment, if any, also show up on this side-pane.

    :::image type="content" source="./media/deploy-7.png" alt-text="Screenshot of the Deploy tab showing the deployment status details." lightbox="./media/deploy-7.png":::

***

In case any of the bulk operations fail partially or for all targets, you can retry the operation for the failed targets with the same parameters.

## Bulk deployment script

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

- [External validation](external-validation.md)
- [Stage solution before deployment](how-to-stage.md)
