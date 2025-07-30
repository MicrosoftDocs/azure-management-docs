---
title: Release Notes for Workload Orchestration
description: Release notes for Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: release-notes
ms.date: 06/24/2025
---

# Release notes for workload orchestration

This article provides the latest and past release notes for workload orchestration in Azure Arc. It includes new features, improvements, and bug fixes.

[!INCLUDE [cli-version-note](includes/cli-version-note.md)]

## July 2025 release (latest)

### Improvements

- Performance improvements: This release introduces advanced caching mechanisms designed to boost performance through acceleration of long-running operations, particularly for:

  - Creation and review of solution templates and targets
  - Publishing and deployment of workflows

- Reliability and enhanced user experience: The improved caching logic increases reliability for deployments in environments with complex hierarchies or significant numbers of targets. This release ensures smoother progress and fewer interruptions when managing resources during periods of high activity or when working with large configurations.

### Bug fixes

- Resolved an issue that enabled deletion of previously deployed versions of solutions.
- Boosted reliability of solution template creation process by fixing race condition that was leading to occasional random failures.

## June 2025 GA release

Workload orchestration is now generally available (GA) in Azure Arc. This release includes all the features and improvements that were previously in preview, along with additional enhancements.

To migrate your existing workload orchestration target resources to the GA version, see [Migration script for workload orchestration](migration-script.md).

### New features

- End-to-end support for Tanzu environment: Enable users to onboard and manage solution deployments across Tanzu-based Kubernetes clusters.
- [Bulk-publish to multiple targets via CLI:](bulk-deployment.md#perform-bulk-publishing) Enables users to publish an application to multiple targets within the same cluster. External validation and staging (if enabled) are automatically triggered as part of the process. 

### Improvements in workload orchestration portal

- Improved error messaging: Users now receive clearer guidance when required context is missing, making it easier to resolve issues.
- Updated column name in the Solution subtab of the Configure tab: "Lines Published To" is now labeled as "Targets Published To."
- Enhanced filter logic: Filtering now supports hierarchies beyond two levels, providing better support for complex configurations.
- Refined rollback experience: The rollback flow now displays only solutions with matching instance names, improving clarity and usability.
- Breadcrumb navigation is now available on all details screens, providing improved navigation throughout the portal.
- Configure flows are now fully responsive, ensuring a seamless experience across devices.
- Enhanced retry logic for solution reviews: Users can now more easily retry reviewing solutions after a failure.
- Additional accessibility improvements have been implemented to further enhance the user experience.

### Improvements in CLI

- Added the `--context-id` parameter to the `target create` command, enabling users to specify the full ARM ID for precise targeting.
- Updated arguments across multiple commands for improved clarity and consistency. See the table below for a summary of these changes:

|Command|Old arguments|New argument|
|---|---|---|
|`az workload-orchestration target review`|`--solution-name`, `--solution-version`|`--solution-template-version-id`|
|`az workload-orchestration target publish`|`--review-id`, `--solution-name`, `--solution-version`|`--solution-version-id`|
|`az workload-orchestration target install`|`--solution-name`, `--solution-version`|`--solution-version-id`|
|`az workload-orchestration target uninstall`|`--solution-name`|`--solution-template-id`|
|`az workload-orchestration target remove-revision`|`--solution-name`|`--solution-template-id`|

### Extension updates

- Cloud extension **workload-orchestration 1.1.8**: Now available and officially part of the Azure CLI extension index. To install it, simply run: 

    ```bash
    az extension add —name workload-orchestration 
    ```

- Arc extension **Microsoft.workloadorchestration 2.1.2**: Delivers new capabilities along with critical bug fixes.

### Bug fixes

- Fixed issue with bulk deployment where the CLI response only listed one target in the `DeployedTargets` array, even when multiple targets were successfully deployed. The CLI now correctly lists all deployed targets.

## June 2025 preview release

### New features

- [Four-level hierarchy support:](service-group.md#service-groups-at-different-hierarchy-levels) You can now create a hierarchy of up to four levels, which allows you to organize your resources such as country, region, factory, and line. 
- [Predeployment staging:](how-to-stage.md) Staging is introduced as a predeployment step to allow container images to be downloaded and synced to edge devices in advance, ensuring reliable deployments within limited maintenance windows.
- [Bulk deployment:](bulk-deployment.md) You can now deploy a solution to multiple targets within the same Kubernetes cluster. 
- [Diagnostic of K8 errors:](diagnose-problems.md) Diagnostic logs are now available for workload orchestration, which allows you to diagnose edge-related errors including container issues and telemetry logs. 

### Improvements

- Instance name: You can add the instance name in the [configuring flow](configure.md#configure-solution-parameters).
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


