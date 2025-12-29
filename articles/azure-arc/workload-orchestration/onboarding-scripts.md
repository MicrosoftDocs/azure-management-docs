---
title: Onboarding Scripts for Workload Orchestration
description: The onboarding scripts are designed to help you set up the necessary infrastructure and resources for workload orchestration in Azure Arc.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: install-set-up-deploy
ms.date: 06/10/2025
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to automate the setup of infrastructure and resources for workload orchestration, so that I can efficiently provision a Kubernetes cluster and related services without manual intervention."
---

# Onboarding scripts for workload orchestration

The onboarding PowerShell scripts are designed to help you set up the necessary infrastructure and resources for workload orchestration in Azure Arc. The scripts automate the process of creating a Kubernetes cluster, deploying on the cluster, creating custom location and site, and installing the workload orchestration CLI extension.

> [!TIP]
> If you prefer to not use the scripts and want to do the setup manually, you can follow the instruction in [Prepare the environment for workload orchestration](initial-setup-environment.md) and [Setup workload orchestration](initial-setup-configuration.md).

## Prerequisites

- Run `winget install -e --id Microsoft.AzureCLI` and `winget install -e --id Kubernetes.kubectl`.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. From the compressed folder, you find the following files:
    - Download the workload-orchestration CLI extension.
    - Download the JSON files for site-address and site content, schemas, configs, *onboarding-data.json* and *mock-data.json*.
    - Edit the `onboarding-data.json` file. You can find mock data in `mock-data.json`.

## Common variables in input JSON

The following fields are common to both the `cmOnboarding` and `infraOnboarding` sections in the input JSON file. These fields are used to define the common resource group, subscription ID, and location for the resources being created. If you want to use a common resource group, subscription ID, or location for both CM and infra resources, you can set them in the common section. Otherwise, you can override them in the `cmOnboarding` or `infraOnboarding` sections.

- `subscriptionId`: (Optional) When using a common subscription ID, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `resourceGroup`: (Optional) When using a common resource group, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `location`: (Optional) When using a common location, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `customLocationFile`: Automatically added by the onboarding script, don't modify it when running the custom location onboarding script.

## Infra onboarding

The infra setup script helps you onboard to the infrastructure needed for workload orchestration, such as creating an AKS cluster, deploying TCO on the cluster, creating custom location and site, and finally installing the workload orchestration CLI extension.

Open a PowerShell terminal and run the following command.

- **Command**: `infra_onboarding.ps1 mock-data.json`
- **Arguments**: 
    - `-skipResourceGroupCreation`: Skip creation of the resource group.
    - `-skipAzLogin`: Skip Azure login, use when you are already logged in.
    - `-skipAzExtensions`: Skip installing/updating `connectedk8s`, `k8s-extension` and `customlocation` extensions.
    - `-skipAksCreation`: Skip creation of AKS cluster, use when the cluster is already created.
    - `-skipTcoDeployment`: Skip connecting AKS to Arc and creation of TCO extension, use when TCO has been deployed already.
    - `-skipCustomLocationCreation`: Skip creation of CustomLocation, use when it has been created before.
    - `-skipConnectedRegistryDeployment`: Skip connected registry deployment. By default, this step is skipped. Set to false when user need to deploy the connected registry on AKS cluster for staging. 
    - `-skipSiteCreation`: Skip creation of Site and SiteAddress, use when it has been created before.
    - `-skipAutoParsing`: Skip auto-creation of custom location file and auto-parsing of site file. By default, you don't need to set the "addressResourceId" field in the site file and do not need to pass a customLocationFile in the target data section. Set to `$true` if you want to assign your own custom location (not created via onboarding script) or your own site address (not created via onboarding script).
    - `-enableWODiagnostics`: Enable workload orchestration extension user-facing logs, use when you want to collect workload orchestration extension user audits and user diagnostics logs. For more information, see [Diagnose edge-related logs and errors](diagnose-problems.md).
    - `-enableContainerInsights`: Enable `Container.Insights` on arc cluster to collect container logs and k8s events. Use when you want to collect container logs or k8s events. For more information, see [Diagnose edge-related logs and errors](diagnose-problems.md).

> [!NOTE]
>  All arguments are boolean which take `$true`/`$false` as values and the default value is `$false`, except for `-skipAzLogin` and `-skipConnectedRegistryDeployment` which are `$true` by default. 

## Context creation script (only if there is no existing context)

Context creation is a one-time operation. If you have already created a context, you can skip this step. If you need to create a new context, use the following command:

```powershell
# One context per tenant 
az workload-orchestration context create `
 --resource-group <Context-ResourceGroup> `
 --location <Location> `
 --name <ContextName> `
 --capabilities [0].name="$resourcePrefix-soap2" [0].description="$resourcePrefix-Soap2" [1].name="$resourcePrefix-shampoo2" [1].description="$resourcePrefix-Shampoo2" `
 --hierarchies [0].name=factory [0].description=Factory [1].name=line [1].description=Line
```

Once you create the context, add the required data in *onboarding-data.json* file. The context creation script doesn't run again, so you need to add the context details in the onboarding data JSON file.

```json
        "contextResourceGroup": "",
        "contextName": "",
        "contextSubscriptionId": "",
        "contextLocation": "",  
```

### Onboarding data JSON

The infra-related properties fall under the `infraOnboarding` section in this file.

- `subscriptionId`: (Optional) If you want to override the common section's subscription ID.
- `resourceGroup`: (Optional) If you want to override the common section's resource group.
- `location`: (Optional, default: `eastus`) If you want to override the common section's location.
- `arcLocation`: (Optional, default: `eastus`) Azure region where the Arc-enabled Kubernetes cluster resource will reside.
- `aksClusterIdentity`: (Optional, default: `$resourceGroup-Cluster-Identity`) Name of the managed identity used by the AKS cluster.
- `aksClusterName`: (Optional, default: `$resourceGroup-Cluster`) Name of the AKS cluster to be created or used.
- `customLocationName`: (Optional, default: `$resourceGroup-Location`) Name for the Custom Location resource created on top of the Arc-enabled AKS cluster.
- `customLocationNamespace`: (Optional, default: `contoso`) Kubernetes namespace associated with the Custom Location. Should be lowercase.
- `workloadOrchestrationWHL`: (Required) File path to the downloaded Workload Orchestration CLI extension `.whl` file.
- `contextResourceGroup`: (Required) Resource group where the Workload Orchestration Context exists (for example, "Contoso"). This is used for setting up capabilities and site references.
- `contextName`: (Required) Name of the Workload Orchestration Context (for example, "Contoso-Context").
- `contextSubscriptionId`: (Required) Subscription ID where the Workload Orchestration Context exists.
- `contextLocation`: (Required) Azure region where the Workload Orchestration Context exists (for example, "eastus2").
- `diagInfo`: (Optional) An array defining the diagnostic configurations.
    - `diagnosticWorkspaceId`: (Optional) The ARM resource ID of log analytics workspace.
    - `diagnosticResourceName`: (Optional) Name of the diagnostic resource.
    - `diagnosticSettingName`: (Optional) Name of the diagnostic settings.
- `acrName`: (Optional) Name of the Azure container registry.
- `connectedRegistryName`: (Optional) Name of the connected registry.
- `connectedRegistryIp`: (Required if `skipConnectedRegistryDeployment=$false`) Available IP address to host the connected registry service.
- `connectedRegistryClientToken`: (Optional) Name of the connected registry client token secret.
- `storageSizeRequest`: (Optional) Size of the storage used for connected registry.
- `siteHierarchy`: (Optional) An array defining the site structure and associated deployment targets.
    - `siteName`: (Required) Name of the site resource to be created. Avoid adding trailing numbers in the name.
    - `parentSite`: (Optional) Name of the parent site in the hierarchy. Set to `null` for top-level sites.
    - `configuration` (Required): Nested object containing data about site configuration
        - `name` (Required): name of site config
        - `location` (Required): location to create site config
    - `level`: (Required) The hierarchy level this site represents (for example, "factory", "line"). Must match a level defined in the Context.
    - `capabilityList`: (Optional) Defines capabilities to be added to the Context if this site node is processed for capability setup.
        - `capabilities`: (Required) An array of capability names (strings) to add/update in the Context.
    - `hierarchyLevels`: (Optional) Defines hierarchy levels to be added to the Context if this site node is processed for capability setup.
        - `levels`: (Required) An array of hierarchy level names (strings) to add/update in the Context.
    - `deploymentTargets`: (Optional) Defines deployment targets associated with this site.
        - `rbac`: (Optional) Default RBAC settings for targets under this site. It can be overridden per target.
            - `role`: (Required) Azure role to assign (for example, "Contributor").
            - `userGroup`: (Required) Object ID of the user or group to assign the role to.
        - `capabilities`: (Optional) Default capabilities for targets under this site. It can be overridden per target. Array of strings.
        - `hierarchyLevel`: (Optional) Default hierarchy level for targets under this site. It can be overridden per target. String.
        - `namespace`: (Optional) Default Kubernetes namespace for targets under this site. (Note: Currently informational, not directly used in target creation command). It can be overridden per target. String.
        - `targetSpecFile`: (Required if 'targets' defined) File path to the JSON file containing the target specification.
        - `customLocationFile`: (Optional) File path to the JSON file containing the custom location ID. If not specified here or per target, it falls back to the file created during Arc cluster creation.
        - `targets`: (Required) An array of deployment target definitions.
            - `name`: (Required) Name of the deployment target resource.
            - `displayName`: (Required) Display name for the deployment target.
            - `capabilities`: (Optional) Overrides the parent `deploymentTargets.capabilities`. Array of strings.
            - `hierarchyLevel`: (Optional) Overrides the parent `deploymentTargets.hierarchyLevel`. String.
            - `namespace`: (Optional) Overrides the parent `deploymentTargets.namespace`. String.
            - `rbac`: (Optional) Overrides the parent `deploymentTargets.rbac`. Object with `role` and `userGroup`.
            - `customLocationFile`: (Optional) Overrides the parent `deploymentTargets.customLocationFile`. File path string.
            - `targetSpecFile`: (Optional) Overrides the parent `deploymentTargets.targetSpecFile`. File path string. (Less common to override per target).


You execute the script with the following command:

```powershell
infra_onboarding.ps1 mock-data.json -skipAzExtensions $True -skipCustomLocationCreation $True
```

## Workload orchestration resources onboarding

The workload orchestration setup script creates application specific resources such solutions, configurations, and schemas.

Open a PowerShell terminal and run the following command.

- **Command**: `cm_onboarding.ps1 mock-data.json`
- **Argument**: 
    - `-skipResourceGroupCreation` (default `$false`): Skip creation of resource group

### Onboarding data JSON

The workload orchestration related properties fall under the `cmOnboarding` section in this file.

- `resourceGroup`: (Optional) Defines the resource group for creating CM resources. Overrides the common one.
- `subscriptionId`: (Optional) Defines the subscription for creating CM resources. Overrides the common one.
- `location`: (Optional, default: `eastus`) Defines the location for creating CM resources. Overrides the common one.
- `contextResourceGroup`: (Required) Specifies the resource group where the workload orchestration context is stored. In Microsoft tenant, this is typically "Contoso".
- `contextName`: (Required) Name of the workload orchestration context. In Microsoft tenant, this is typically "Contoso-Context".
- `contextSubscriptionId`: (Required) Subscription ID where the context resource exists. This may differ from your main deployment subscription.
- `contextLocation`: (Required) Azure region where the context resource is deployed. Must be a region that supports workload orchestration.

- `schemas`: (Optional) Defines the schemas to be created.
    - `name`: (Required) Name of the schema.
    - `version`: (Required) Version of the schema.
    - `schemaFile`: (Required) File path of the schema definition.

- `configs`: (Optional) Defines the configurations to be created.
    - `name`: (Required) Name of the configuration.
    - `versionName`: (Required) Version name of the configuration.
    - `configFile`: (Required) File path of the configuration file.

- `solutions`: (Optional) Defines the solutions to be created.
    - `name`: (Required) Name of the solution.
    - `description`: (Required) Description of the solution.
    - `capabilities`: (Required) Array of capabilities for the solution.
    - `version`: (Required) Version of the solution.
    - `specificationFile`: (Required) File path to Specification File.
    - `configTemplate`: (Required) Configuration template for the solution.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

## Related content

- [Service groups with workload orchestration](service-group.md)
- [RBAC guide](rbac-guide.md)
- [Configuration template](configuring-template.md)
- [Configuration schema](configuring-schema.md)
