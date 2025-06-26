---
title: Role Based Access Control (RBAC) Guide for Workload Orchestration
description: Learn how to set up Role-Based Access Control (RBAC) when using workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: overview
ms.date: 05/03/2025
ms.custom:
  - build-2025
---

# Role Based Access Control (RBAC) guide for workload orchestration

This article describes how you can set up RBAC when using workload orchestration. Standard Azure RBAC is compatible with workload orchestration since all objects are ARM-based. You can use either the Azure Standard Built-in Roles (Owner, Reader, and Contributor) or a Custom Defined Role to support various combinations of actions suitable for their organization.

For more information, see [Azure Role-Based Access Control (RBAC)](/azure/role-based-access-control/custom-roles).

## Use standard built-in roles

Workload orchestration supports the following standard built-in roles. These roles are defined at the Resource Group level, and you can assign them to users, groups, or service principals. The following table describes the roles and their scopes:

|Role	|Scope	|Comments|
|---	|---	|---	|
|Owner	|Resource Group	|They can create/update/delete workload orchestration Resources in the Resource Group.|
|Reader	|Resource Group	|They can read all workload orchestration Resources in the Resource Group.|
|Contributor| Resource Group| They can create/update capabilities, solutions, sites, schemas, author configuration, deploy, and rollback.|

## Create custom roles 

You can define custom roles based on Azure RBAC guidelines to meet your organization's specific needs. Workload orchestration allows you to create custom roles by enabling the following actions on the associated resources for different objects. 

The following table provides a summary of the roles you can create and the actions you can assign to them:

|Role |Description|Actions|
|---|---|---|
|Admin|Full access to all workload orchestration resources and actions|Use Azure Core Contributor role|
|Onboarding admin| |`Microsoft.Edge/contexts/*`, `Microsoft.Edge/targets/write`, `Microsoft.Edge/targets/read`|
|Solution author|Read-write solutions and their versions| `Microsoft.Edge/contexts/read`, `Microsoft.Edge/solutionTemplates/*`,  `Microsoft.Edge/targets/solutions/*`|
|Solution viewer|Read solutions and their versions |`Microsoft.Edge/contexts/read`, `Microsoft.Edge/solutionTemplates/read`, `Microsoft.Edge/targets/solutions/read`, `Microsoft.Edge/targets/solutions/versions/read`, `Microsoft.Edge/solutionTemplates/versions/read`|
|Configuration author|<ul><li>Read site and target to view hierarchy </li><li> Read dynamic schemas and read-write shared schemas</li><li>Read-write all configuration RTs</li><li>Read, resolve, and publish solution binding</li><li>Read resolved and published configuration </li></ul>|`Microsoft.Edge/contexts/read`, `Microsoft.Edge/sites/read`, `Microsoft.Edge/configurationtemplates/*`, `Microsoft.Edge/schemaReferences/read`, `Microsoft.Edge/schemas/*`, `Microsoft.Edge/configurations/*`, `Microsoft.Edge/targets/read`, `Microsoft.Edge/targets/solutions/*`, `Microsoft.Edge/targets/reviewSolutionVersion/action`, `Microsoft.Edge/targets/publishSolutionVersion/action`|
|Configuration viewer|<ul><li>Read site and target to view hierarchy </li><li>Read dynamic and shared schemas</li><li>Read all configuration RTs</li><li>Read solution bindings</li><li>Read resolved and published configuration </li></ul>|`Microsoft.Edge/contexts/read`, `Microsoft.Edge/sites/read`, `Microsoft.Edge/configurationtemplates/read`, `Microsoft.Edge/schemaReferences/read`, `Microsoft.Edge/schemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/versions/read`, `Microsoft.Edge/configurations/read`, `Microsoft.Edge/configurations/dynamicConfigurations/read`, `Microsoft.Edge/configurations/dynamicConfigurations/versions/read`, `Microsoft.Edge/targets/solutions/read`, `Microsoft.Edge/targets/solutions/versions/read`, `Microsoft.Edge/targets/solutions/instances/read`, `Microsoft.Edge/targets/solutions/instances/histories/read`|
|Deployment admin|<ul><li>Read-write-delete targets</li><li>Read solution bindings</li><li>Read resolved and published configuration</li><li>Monitor published configuration </li></ul>|`Microsoft.Edge/contexts/read`, `Microsoft.Edge/sites/read`, `Microsoft.Edge/targets/write`,`Microsoft.ExtendedLocation/customLocations/read`, `Microsoft.Edge/configurationReferences/read`, `Microsoft.Edge/schemaReferences/read`, `Microsoft.Edge/schemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/versions/read`, `Microsoft.Edge/targets/solutions/read`, `Microsoft.Edge/targets/solutions/versions/read`, `Microsoft.Edge/targets/solutions/instances/read`, `Microsoft.Edge/targets/solutions/instances/histories/read`|
|Deployment manager|<ul><li>Read targets </li><li>Read solution bindings</li><li>Read resolved and published configuration</li><li>Deploy published configuration </li></ul>|`Microsoft.Edge/contexts/read`, `Microsoft.Edge/targets/read`, `Microsoft.Edge/configurationReferences/read`, `Microsoft.Edge/schemaReferences/read`, `Microsoft.Edge/schemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/read`, `Microsoft.Edge/schemas/dynamicSchemas/versions/read`, `Microsoft.Edge/targets/installSolution/action`, `Microsoft.Edge/targets/uninstallSolution/action`, `Microsoft.Edge/targets/solutions/read`, `Microsoft.Edge/targets/solutions/versions/read`, `Microsoft.Edge/targets/solutions/instances/read`, `Microsoft.Edge/targets/solutions/instances/histories/read`|
|Deployment viewer|<ul><li>Read targets </li><li>Read solution bindings</li><li>Read resolved and published configuration</li></ul>|`Microsoft.Edge/contexts/read`, `Microsoft.Edge/targets/read`, `Microsoft.Edge/targets/solutions/read`, `Microsoft.Edge/targets/solutions/versions/read`, `Microsoft.Edge/targets/solutions/instances/read`, `Microsoft.Edge/targets/solutions/instances/histories/read`|

### Available objects and their actions

The following sections list the available objects and their actions that you can use to create custom roles for workload orchestration.

#### Context

- `Microsoft.Edge/contexts/write`: Allows writing context information.
- `Microsoft.Edge/contexts/read`: Allows reading context information.
- `Microsoft.Edge/contexts/siteReferences/write`: Allows writing site references within contexts.
- `Microsoft.Edge/contexts/siteReferences/read`: Allows reading site references within contexts.

#### Schema

- `Microsoft.Edge/schemas/write`: Allows writing schema information.
- `Microsoft.Edge/schemas/read`: Allows reading schema information.
- `Microsoft.Edge/schemas/versions/write`: Allows writing schema versions.
- `Microsoft.Edge/schemas/versions/read`: Allows reading schema versions.
- `Microsoft.Edge/schemas/dynamicSchemas/read`: Allows reading dynamic schemas.
- `Microsoft.Edge/schemas/dynamicSchemas/versions/read`: Allows reading versions of dynamic schemas.
- `Microsoft.Edge/schemaReferences/read`: Allows reading schema references.

#### Configuration Templates

- `Microsoft.Edge/configurationtemplates/write`: Allows writing configuration templates.
- `Microsoft.Edge/configurationtemplates/read`: Allows reading configuration templates.
- `Microsoft.Edge/configurationtemplates/versions/read`: Allows reading versions of configuration templates.
- `Microsoft.Edge/configurationtemplates/createVersion/action`: Allows creating a new version of a configuration template.
- `Microsoft.Edge/configurationtemplates/removeVersion/action`: Allows removing a version of a configuration template.

#### Configurations

- `Microsoft.Edge/configurations/read`: Allows reading configurations.
- `Microsoft.Edge/configurations/dynamicConfigurations/read`: Allows reading dynamic configurations.
- `Microsoft.Edge/configurations/dynamicConfigurations/versions/read`: Allows reading versions of dynamic configurations.
- `Microsoft.Edge/configurations/dynamicConfigurations/versions/write`: Allows writing versions of dynamic configurations.
- `Microsoft.Edge/configurationreferences/read`: Allows reading configuration references.

#### Solution Templates

- `Microsoft.Edge/solutionTemplates/write`: Allows writing solution templates.
- `Microsoft.Edge/solutionTemplates/read`: Allows reading solution templates.
- `Microsoft.Edge/solutionTemplates/versions/read`: Allows reading versions of solution templates.
- `Microsoft.Edge/solutionTemplates/createVersion/action`: Allows creating a new version of a solution template.
- `Microsoft.Edge/solutionTemplates/removeVersion/action`: Allows removing a version of a solution template.

#### Targets

- `Microsoft.Edge/targets/write`: Allows writing target information.
- `Microsoft.Edge/targets/read`: Allows reading target information.
- `Microsoft.Edge/targets/reviewSolutionVersion/action`: Allows reviewing a solution version for a target.
- `Microsoft.Edge/targets/publishSolutionVersion/action`: Allows publishing a solution version for a target.
- `Microsoft.Edge/targets/removeRevision/action`: Allows removing a revision for a target.
- `Microsoft.Edge/targets/installSolution/action`: Allows installing a solution on a target.
- `Microsoft.Edge/targets/uninstallSolution/action`: Allows uninstalling a solution from a target.
- `Microsoft.Edge/targets/solutions/read`: Allows reading solutions associated with a target.
- `Microsoft.Edge/targets/solutions/versions/read`: Allows reading versions of solutions associated with a target.
- `Microsoft.Edge/targets/solutions/instances/read`: Allows reading instances of solutions associated with a target.
- `Microsoft.Edge/targets/solutions/instances/histories/read`: Allows reading histories of solution instances associated with a target.

## User scenarios for CLI access

The following table describes some of the user scenarios and the associated custom-defined roles that you can assign to users for CLI access:

|Scenario| Associations|
|---|---|
| Users want to Author new Solutions for Consumption (IT)| Assign "Solution Author" at RG level|
| Users want to View new Solutions for Consumption (IT)  | Assign "Solution Viewer" at RG /solution level  |  
| Users want to Onboard New Targets (SiteAdmin)  | Assign "Deployment Admin" at RG level  |  
| Users want to Configure Hierarchy (Area/BU Leader)  | Assign "Configuration Author" at RG level  |  
| Users want to View Configured Hierarchy (Equipment Owner)  | Assign "Configuration Viewer" at RG level/configuration Level  |  
| Users want to Configure a specific Target (Line Leader)  | See “Scenarios at Target Level” Section  |  
| Users want to View a specific Target (Equipment Owner)  | See “Scenarios at Target Level” Section  |  
| Users want to Manage Applications (Equipment Owner)  | Assign "Deployment Manager" at RG level /solution level  |  
| Users want to View Applications (Equipment Owner)  | Assign "Deployment Viewer" at RG level/ solution level  |  
| Users want to Manage Applications on specific Target (Line Owner)  | See “Scenarios at Target Level” Section  |  
| Users want to View Applications on specific Target (Line Owner)  | See “Scenarios at Target Level” Section  |  


## User scenarios for Azure portal access

The following table describes some of the user scenarios and the associated custom-defined roles that you can assign to users for Azure portal access:

|Scenario| Associations|
|---|---|
|IT Admin| Full Access via Contributor or Assign Onboarding Admin + Solution Author + Configuration Author + Deployment Admin|
|Equipment Owner|Assign Solution Viewer + Deployment Viewer + Configuration Author|
|Line Operator|Assign Solution Viewer + Deployment Manager|
|Model Owner|Assign Solution Viewer|
|Platform Owner | Assign Deployment Admin|


### RBAC for service groups

The following table shows recommended actions for different user roles when using a custom-defined role at a service group level. For more information about service groups, see [Service groups for workload orchestration](service-group.md).

#### Service group contributor​

A service group contributor​ is the admin user who creates the service group and is responsible for setting up the entire service group and Sites hierarchy​

|Description|Actions|
|---|---|
|Create and manage service groups \*| `Microsoft.Management/serviceGroups/read​`, `Microsoft.Management/serviceGroups/write​`, `Microsoft.Management/serviceGroups/delete​`|
|Create and manage relationships on top of `DeploymentTargets`| `Microsoft.Relationships/serviceGroupMember/read` on `DeploymentTarget` + Target service group​, `Microsoft.Relationships/serviceGroupMember/write` on `DeploymentTarget` + Target service group​, `Microsoft.Relationships/serviceGroupMember/delete` on `DeploymentTarget` + Target service group​|
|View the resources under the service group|Ensure that the user has READ access to the `DeploymentTarget` resources that are part of the service group. For this, the user can either have access at the resource group/subscription level or directly for the particular `DeploymentTarget` resource.​|

\* User needs to be a registered user on the Tenant. Once a user creates a service group, they automatically create the following permissions on the service group. 

#### Service group reader

A service group reader is any user who can view the service groups and sites hierarchy​.

|Description|Actions|
|---|---|
|View service groups by service group contributor​ | `Microsoft.Management/serviceGroups/read​`|
|View already existing relationships on top of `DeploymentTargets`| `Microsoft.Relationships/serviceGroupMember/read` on `DeploymentTarget` + Target service group​|
|View the resources under the service group|Ensure that the user has READ access to the `DeploymentTarget` resources that are part of the service group. For this, the user can either have access at the resource group/subscription level or directly for the particular `DeploymentTarget` resource.​|

#### Service group admin​

A service group admin is any user who can view, edit, and delete the service groups created by the service group contributor​​.

|Description|Actions|
|---|---|
|Create and manage service groups| `Microsoft.Management/serviceGroups/read​`, `Microsoft.Management/serviceGroups/write​`, `Microsoft.Management/serviceGroups/delete​`|
|Create and manage relationships on top of `DeploymentTargets`| `Microsoft.Relationships/serviceGroupMember/read` on `DeploymentTarget` + Target service group​, `Microsoft.Relationships/serviceGroupMember/write` on `DeploymentTarget` + Target service group​, `Microsoft.Relationships/serviceGroupMember/delete` on `DeploymentTarget` + Target service group​|
|View the resources under the service group| Ensure that the user has READ access to the `DeploymentTarget` resources that are part of the service group. For this, the user can either have access at the resource group/subscription level or directly for the particular `DeploymentTarget` resource.​|


