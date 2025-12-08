---
title: Release Notes for Workload Orchestration
description: Release notes for Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: release-notes
ms.date: 11/04/2025
---

# Release notes for workload orchestration

This article provides the latest and past release notes for workload orchestration in Azure Arc. It includes new features, improvements, and bug fixes.

## November 2025 release

### New features

- **Bulk deployment across multiple clusters**: Workload orchestration now supports deploying a solution to targets residing across multiple clusters in a single operation. This enhancement delivers greater flexibility and scalability for hybrid and multi-cloud environments.
- Schemas will no longer support the **editableAt** property for application parameters, that was used to specify the hierarchy level at which those parameters need to be configured. Configuration templates referring to them will now need to be explicitly linked to a hierarchy level, and all solution templates will be automatically linked to the target where the corresponding solution is deployed. This enforces stricter control over configuration changes, avoiding ambiguity and reducing accidental edits at unintended levels.
- Users can now configure, publish and deploy solutions to multiple targets, a capability previously accessible only through CLI, via the workload orchestration portal interface. Targets can be filtered by name, hierarchy level, capability tags, parent site, etc. and users can choose to configure common parameter values for all targets or set custom values for each.

### Improvements in Portal

- The **Configure** tab that enabled operations related to configuration and publishing of solutions and targets, has now been split into **Configure hierarchy** and **Configure solutions**, which offers clear segregation between hierarchy and leaf targets, and solutions.
- Users can now filter and group targets by their parent site, in addition to existing parameters, in the **Monitor**, **Configure hierarchy** and **Deploy** tabs of portal.

### Improvements in CLI

- The time taken for target and solution template creation has been significantly reduced, resulting in faster response time and enhanced CLI experience.
- Users can now view the schema validation rules while configuring solution parameters for a target.

## October 2025 release

### New features

- The status of solutions and targets shown to users in the workload orchestration portal, related to completion of configuration, publishing and deployment, are standardized across all screens. The distinct number of statuses are reduced for simplification and enhanced readability. Users can click on the status of any item to open a context pane showing details of all operations performed on it so far, along with timestamp and name of user who performed the action.
- The first party app **EdgeConfigurationManagerApp** access requirement is reduced from **Contributor** to **Reader** role.

## September 2025 release

### New features

- [Bulk configuration and review](bulk-deployment.md): You can set configurations for multiple targets and review all at once through CLI, with the flexibility of applying the same configuration to all or individual configurations per target. Upon command execution, the output lists the completion status for each target, along with detailed error logs for troubleshooting.
- [Workload Orchestration SDK](workload-orchestration-sdk.md): The Azure Workload Orchestration SDK is now available in 4 languages - Python, Java, JavaScript and Golang, enabling developers to automate tasks programmatically. Key capabilities include:
    - Creation and management of context, targets, schema and solution templates
    - Configuration, publishing and deployment of solutions
    - Support for asynchronous operations and automatic retries
    - Comprehensive logging for enhanced observability

### Improvements in CLI

- The Azure CLI Workload Orchestration extension has been updated to version **4.0.0**, introducing bulk capabilities for review and configuration tasks, along with additional flexibility for bulk publishing operations. It also utilizes the latest API version 2025-08-01. To update to the latest version of extension, simply run:

```bash
az extension update --name workload-orchestration
```

- The existing bulk publish CLI functionality is upgraded to allow users to bypass the review step and achieve configuration and publishing of a solution to multiple targets in a single command. The list of targets can include a mix of those that have been already reviewed and those for which they intend to bypass the review process.

### Bug fixes

- Updated the Workload Orchestration portal to appropriately mask site details from users having access to child targets but not the respective parent site.
- Improved configuration status accuracy across portal views.

## August 2025 release

### New features

- *Granular visibility of deployment operations*: Users can now view detailed real‑time progress in the workload orchestration portal while deploying solutions. This includes intermediate steps along with timestamps, and the user who initiated the deployment. In case of a failure, users can see the exact stage of operations at which it occurred and the associated error message for troubleshooting.

- *Event Log for improved monitoring*: A new **Event Log** tab under the **Monitor** pane in the portal provides a chronological view of all user‑initiated actions across solutions and targets. Users can filter or group the view based on relevant fields, and further drill down into details of individual executions within an operation. This feature acts as an audit trail and improves transparency in shared environments.

### Improvements in workload orchestration portal

- Eliminated delays between the live status of a user-initiated action displayed on portal and its corresponding notifications, presenting a more accurate and consistent view of operations.

- The portal now has improved accessibility. Details can be found [here](https://www.microsoft.com/accessibility/conformance-reports).

### Improvements in CLI

- **Azure CLI Workload Orchestration Extension 3.0.0** is now available with new capabilities and improvements. To install it, run:

    `az extension add --name workload-orchestration`

    To update your existing installation to the latest version, run:

    `az extension update --name workload-orchestration`

- The extension now provides more robust capabilities for context and solution management, specifically the ability to set and view current context, view all solution instances and revisions within a target, and display parameter values configured for all previous versions of a configuration template

- CLI persistently stores information related to the current context used, allowing automatic fallback to default context in a new session and enhanced context ID validation.

- Improved user experience for context creation and management through enhanced output readability and more actionable error messages.

### Bug fixes

Resolved an issue related to the timestamp not being displayed correctly for few portal notifications.
 
## July 2025 release

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

- [Four-level hierarchy support:](service-group.md#service-groups-at-different-hierarchy-levels) You can now create a hierarchy of up to four levels, which allows you to organize your resources such as region, city, factory, and line. 
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


