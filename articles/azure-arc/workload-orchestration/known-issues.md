---
title: Known Issues for Workload Orchestration
description: This article provides a list of known issues for workload orchestration in Azure Arc, including workarounds.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: troubleshooting-known-issue
ms.date: 06/08/2025
---

# Known issues for workload orchestration

This article provides a list of known issues for workload orchestration in Azure Arc. It includes workarounds and solutions for each issue. Known issues are updated as new issues are discovered and resolved.


## Bulk deployment response lists a single deployed target 

When executing a [bulk deployment](bulk-deployment.md), the CLI response only lists one target in the `DeployedTargets` array, even though all the targets are successfully deployed. 

In the case of deployment failure, the CLI returns an error message and only lists one target under both `DeployedTargets` and `FailedTargets` arrays, even though the deployment fails. 

The message in the CLI response is a known issue and doesn't reflect the actual deployment status of the targets. Check the [workload orchestration portal](deploy.md) to verify the application status for each target. 

## Related content

- [Troubleshooting workload orchestration](troubleshooting.md)
- [Release notes for workload orchestration](release-notes.md)


 
