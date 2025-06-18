---
title: Bulk Deployment with Workload Orchestration
description: Learn how to perform bulk deployment of workloads using workload orchestration in Azure Arc.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/04/2025
---

# Bulk deployment with workload orchestration

Workload orchestration allows you to deploy multiple workloads in bulk. With bulk deployment, you can publish a solution across multiple clusters. 

Currently bulk deployment is only supported via CLI.

## Prerequisites

Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Perform a bulk deployment

To perform a bulk deployment, run `bulk-deploy-solution` command in the Azure CLI.

```powershell
az workload-orchestration solution-template bulk-deploy-solution --targets "@target-file.json"  --solution-instance-name "<solution instance name>" --solution-version "<solution template version>" --solution-name "<solution-name>" --solution-dependencies "@dependencies-file.json" -g "<resource group>"
```

> [!NOTE]
> The `--solution-dependencies` parameter is only required if the solution has dependencies. 

You need to provide a *target-file.json* file that contains the list of targets where you want to deploy the solution. The file should be in the following format:

```json
[
    {
        "targetId": "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target1Name",
        "solutionInstanceName": "<instance name1>", // alphanumeric, small case, no spaces, no special characters
    },
    {
        "targetId": "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$target2Name",
        "solutionInstanceName": "<instance name2>" // alphanumeric, small case, no spaces, no special characters
    }
]
```

The `solutionInstanceName` parameter can be given for each target as part of target-file.json file or as part of CLI input. If you provide the solution instance name in both the target file and the CLI input, the solution instance name from the target file takes precedence. `solutionInstanceName` should be alphanumeric, in lowercase, and should not contain spaces or special characters.

> [!IMPORTANT]
> You need to have access to the target clusters on which the solution is deployed. 

If your solution has dependencies on other solutions, you also need to provide a *dependencies-file.json* file that contains the list of dependencies for the solution. The file should be in the following format:

```json
[
    {
        "solutionVersionId": "<solution version ID>",
    }
]
```

## Bulk deployment failures

There are two types of failures that can occur during bulk deployment: complete failure, when all the targets fail to deploy, and partial failure, when some targets succeed while others fail.

In the case of a partial failure, the CLI returns an error message indicating that the bulk deployment failed, a list of targets that succeeded and those that failed and their error messages. You can then review the error messages for the failed targets and retry the deployment for those specific targets.

In case of a complete failure, the CLI will return an error message indicating that all targets failed to deploy and their error messages. You can review the error messages and troubleshoot the issues before retrying the deployment.

## Related content

- [Diagnose of problems](diagnose-problems.md)
- [Service groups](service-group.md)