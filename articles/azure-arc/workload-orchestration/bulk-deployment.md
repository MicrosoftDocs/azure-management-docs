---
title: Bulk Deployment with Workload Orchestration
description: Learn how to perform bulk deployment of workloads using workload orchestration in Azure Arc.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/02/2025
---

# Bulk deployment with workload orchestration

Workload orchestration allows you to deploy multiple workloads in bulk. With bulk deployment, you can publish a solution across multiple clusters. 

Currently bulk deployment is only supported via CLI.

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

## Perform a bulk deployment

To perform a bulk deployment, run `bulk-deploy-solution` command in the Azure CLI.

```powershell
az workload-orchestration solution-template bulk-deploy-solution --targets "@target-file.json"  --solution-instance-name "<solution instance name>" --solution-version "<solution template version>" --solution-name "<solution-name>" --dependencies "@dependencies-file.json" -g "<resource group>"
```

You need to provide a *target-file.json* file that contains the list of targets where you want to deploy the solution. An instance name for the solution is optional. The file should be in the following format:

```json
[
    {
        "targetId": "<target resource ID>",
        "solutionInstanceName": "<instance name>",
    },
    {
        "targetId": "<target resource ID>",
    }
]
```

> [!IMPORTANT]
> You need to have access to the target clusters on which the solution is deployed. 

You also need to provide a *dependencies-file.json* file that contains the list of dependencies for the solution. The file should be in the following format:

```json
[
    {
        "solutionVersionId": "<solution version ID>",
    }
]
```

## Bulk deployment failures

There are two types of failures that can occur during bulk deployment: complete failure, when all the targets fail to deploy, and partial failure, when some targets succeed while others fail.

When a bulk deployment fails, the CLI will return an error message indicating the failed and deployed targets in case of a partial failure. In case of a complete failure, the CLI will return an error message indicating that all targets failed to deploy.

## Related content

- [Diagnose of problems](diagnose-problems.md)
- [Service groups](service-group.md)