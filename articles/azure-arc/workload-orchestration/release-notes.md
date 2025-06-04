---
title: Release Notes for Workload Orchestration
description: Release notes for Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: release-notes
ms.date: 06/01/2025
---

# Release notes for workload orchestration

This article provides the latest and past release notes for workload orchestration in Azure Arc. It includes new features, improvements, and bug fixes.

[!INCLUDE [cli-version-note](includes/cli-version-note.md)]

## June 2025 release

### New features

- [Four-level hierarchy support:](service-group.md#service-groups-at-different-hierarchy-levels) You can now create a hierarchy of up to four levels, which allows you to organize your resources such as country, region, factory, and line. 
- [Pre-deployment staging:](how-to-stage.md) Staging is introduced as a pre-deployment step to allow container images to be downloaded and synced to edge devices in advance, ensuring reliable deployments within limited maintenance windows.
- [Bulk deployment:](bulk-deployment.md) You can now deploy a solution to multiple targets within the same Kubernetes cluster. 
- [Diagnostic of K8 errors:](diagnose-problems.md) Diagnostic logs are now available for workload orchestration, which allows you to diagnose edge-related errors including container issues and telemetry logs. 

### Improvements

- Faster Performance: API response times is reduced by up to ~30% for all actions.  
- Instance Name: You can add the instance name in the configure flow.
- Grouping by version: When configuring shared applications in the portal, choose the instance from the dropdown, which is now grouped by solution version.

## May 2025 release

### Improvements

- IT users can set configurations from file via CLI. 

    ```powershell
    az workload-orchestration configuration set `
      --resource-group $rg `
      --solution-template-name $solutionName `
      --target-name $targetName `
      --file <filepath>
    ```

- IT users can download configuration files for a particular solution template and target via CLI.

    ```powershell
    az workload-orchestration configuration download `
      --resource-group $rg `
      --solution-template-name $solutionName `
      --target-name $targetName
    ```


