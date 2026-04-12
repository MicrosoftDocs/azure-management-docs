---
title: Onboarding Scripts for Workload Orchestration
description: The onboarding scripts are designed to help you set up the necessary infrastructure and resources for workload orchestration in Azure Arc.
author: sethmanheim
ms.author: sethm
ms.topic: install-set-up-deploy
ms.date: 06/10/2025
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to automate the setup of infrastructure and resources for workload orchestration, so that I can efficiently provision a Kubernetes cluster and related services without manual intervention."
---

# Onboarding scripts for workload orchestration

The onboarding scripts are designed to help you set up the necessary infrastructure and resources for workload orchestration in Azure Arc. The scripts automate the process of creating a Kubernetes cluster, deploying on the cluster, creating custom location and site, installing the workload orchestration CLI extension and other resources necessary to deploy your 1st application. The scripts are available in 3 variants - PowerShell, Python and Bash, all of which are functionally equivalent.

> [!TIP]
> If you prefer to not use the scripts and want to do the setup manually, you can follow the instruction in [Prepare the environment for workload orchestration](initial-setup-environment.md) and [Setup workload orchestration](initial-setup-configuration.md).

## Prerequisites

- Run `winget install -e --id Microsoft.AzureCLI` and `winget install -e --id Kubernetes.kubectl`.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder.
- Fill the `onboarding-data.json` file with your details, or directly edit the file `mock-data.json` containing the mock data. The files are identical and either of them can be used as input while running the scripts. Instructions about various properties are provided below.

### Additional prerequisites by platform

| Platform | Requirements |
|----------|-------------|
| **PowerShell** | Azure CLI, kubectl (installed via winget above) |
| **Python** | Python 3.8+, Azure CLI, kubectl. Install via `winget install -e --id Python.Python.3.12` |
| **Bash (Shell)** | Git Bash (Windows) or native Bash (Linux/macOS), Azure CLI, kubectl, `jq`. On Windows, the shell scripts auto-detect `jq.exe` in the `tools/` directory — download it from [jq releases](https://github.com/jqlang/jq/releases) if not already present. |

## Common variables in input JSON

The following fields are common to both the `cmOnboarding` and `infraOnboarding` sections in the input JSON file. These fields are used to define the common resource group, subscription ID, and location for the resources being created. If you want to use a common resource group, subscription ID, or location for both CM and infra resources, you can set them in the common section. Otherwise, you can override them in the `cmOnboarding` or `infraOnboarding` sections.

- `subscriptionId`: (Optional) When using a common subscription ID, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `resourceGroup`: (Optional) When using a common resource group, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `location`: (Optional) When using a common location, don't override this field in the `cmOnboarding` or `infraOnboarding` section.
- `customLocationFile`: Automatically added by the onboarding script, don't modify it when running the custom location onboarding script.

## Step 1: Infra onboarding

The infra setup script helps you onboard to the infrastructure needed for workload orchestration, such as creating an AKS cluster, deploying TCO on the cluster, creating custom location and site, and finally installing the workload orchestration CLI extension.

> [!IMPORTANT]
> The Service Group name should be unique across tenants. So the site name input must be chosen carefully.

Open a terminal and run the following command.

### [PowerShell](#tab/powershell)

```powershell
.\powershell\infra_onboarding.ps1 mock-data.json
```

### [Python](#tab/python)

```python
python python/infra_onboarding.py mock-data.json
```

### [Bash](#tab/bash)

```bash
bash shell/infra_onboarding.sh mock-data.json
```
---

### Arguments

All of them are boolean arguments. PowerShell uses `$true`/`$false`, Python and Bash use `--flag-name` style.

| PowerShell flag | Python / Bash flag | Default | Description |
|-----------------|--------------------|---------|-------------|
| `-skipAzLogin $true` | `--skip-az-login` (default on) / `--no-skip-az-login` | `true` | Skip `az login`. |
| `-skipAzExtensions $true` | `--skip-az-extensions` | `false` | Skip installing/updating connectedk8s, k8s-extension & customlocation extensions. |
| `-skipResourceGroupCreation $true` | `--skip-resource-group-creation` | `false` | Skip creation of resource group. |
| `-skipAksCreation $true` | `--skip-aks-creation` | `false` | Skip creation of AKS cluster. Use when the cluster is already created. |
| `-skipTcoDeployment $true` | `--skip-tco-deployment` | `false` | Skip connecting AKS to Arc and creation of TCO extension. Use when TCO has been deployed already. |
| `-skipCustomLocationCreation $true` | `--skip-custom-location-creation` | `false` | Skip creation of CustomLocation. Use when it has been created before. |
| `-skipConnectedRegistryDeployment $false` | `--no-skip-connected-registry-deployment` | `true` | Skip connected registry deployment. By default, this step is skipped. Set to false when user needs to deploy the connected registry on AKS cluster for staging. |
| `-skipSiteCreation $true` | `--skip-site-creation` | `false` | Skip creation of Site and SiteAddress. Use when it has been created before. |
| `-skipAutoParsing $true` | `--skip-auto-parsing` | `false` | Skip auto-creation of custom location file and auto-parsing of site file. Set to true if you want to assign your own custom location or site address. |
| `-skipRelationshipCreation $true` | `--skip-relationship-creation` | `false` | Skip creation of serviceGroupMember relationships. |
| `-enableWODiagnostics $true` | `--enable-wo-diagnostics` | `false` | Enable workload orchestration extension user-facing logs. |
| `-enableContainerInsights $true` | `--enable-container-insights` | `false` | Enable Container.Insights on arc cluster to collect container logs and k8s events. |


### Input properties

The properties being used in this step fall under the `infraOnboarding` section in the `onboarding-data.json` file.

- `subscriptionId [Optional]` : If you want to override the common section's sub.
- `resourceGroup [Optional]` : If you want to override the common section's RG.
- `location [Optional]` (default: `eastus`) : If you want to override the common section's location.
- `arcLocation [Optional]` (default: `eastus`): Azure region where the Arc-enabled Kubernetes cluster resource will reside.
- `aksClusterIdentity [Optional]` (default: `$resourceGroup-Cluster-Identity`): Name of the managed identity used by the AKS cluster.
- `aksClusterName [Optional]` (default: `$resourceGroup-Cluster`): Name of the AKS cluster to be created or used.
- `customLocationName [Optional]` (default: `$resourceGroup-Location`): Name for the Custom Location resource created on top of the Arc-enabled AKS cluster.
- `customLocationNamespace [Optional]` (default: `mehoopany`): Kubernetes namespace associated with the Custom Location. Should be lowercase.
- `contextResourceGroup [Required]`: Resource group where the Workload Orchestration Context exists (e.g., "Mehoopany"). This is used for setting up capabilities and site references.
- `contextName [Required]`: Name of the Workload Orchestration Context (e.g., "Mehoopany-Context").
- `contextSubscriptionId [Required]`: Subscription ID where the Workload Orchestration Context exists.
- `contextLocation [Required]`: Azure region where the Workload Orchestration Context exists (e.g., "eastus2euap").
- `diagInfo [Optional]`: An array defining the diagnostic configurations.
    - `diagnosticWorkspaceId [Optional]`: The ARM resource id of log analytics workspace.
    - `diagnosticResourceName [Optional]`: Name of the diagnostic resource.
    - `diagnosticSettingName [Optional]`: Name of the diagnostic settings.
- `acrName [Optional]`: Name of the Azure container registry.
- `connectedRegistryName [Optional]`: Name of the connected registry.
- `connectedRegistryIp [Required if skipConnectedRegistryDeployment=$false]`: Available IP address to host the connected registry service.
- `connectedRegistryClientToken [Optional]`: Name of the connected registry client token secret.
- `customLocationOid [Optional]`: Object ID of the `Microsoft.ExtendedLocation` resource provider service principal. Used when enabling custom-locations feature on the Arc-connected cluster. If not provided, the script attempts to resolve it via Azure AD (`az ad sp show`). If that fails due to insufficient privileges, it falls back to the well-known Microsoft tenant OID (`51dfe1e8-70c6-4de5-a08e-e18aff23d815`). You can look up the OID via `az ad sp show --id bc313c14-388c-4e7d-a58e-70017303ee3b --query id -o tsv` from an account with Azure AD read permissions.
- `storageSizeRequest [Optional]`: Size of the storage used for connected registry.
- `siteHierarchy [Optional]`: An array defining the site structure and associated deployment targets.
    - `siteName [Required]`: Name of the site resource to be created. Avoid adding trailing Numbers in name
    - `isRGSite [Optional]` (default: `false`): When set to `true`, the site is created as a Resource Group-scoped site instead of a Service Group-scoped site. RG-based sites do not require a Service Group, do not support `parentSite`, and skip the `serviceGroupMember` relationship creation. The site is created via `PUT /subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Edge/sites/{siteName}`. **Note:** Only one RG-based site is allowed per Resource Group scope. If you need multiple sites, use separate Resource Groups or use SG-based sites.
    - `parentSite [Optional]`: Name of the parent site in the hierarchy. Set to `null` for top-level sites. Ignored when `isRGSite` is `true`.
    - configuration [Required]: Nested object containing data about site configuration
        - name [Required]: name of site config
        - location [Required]: location to create site config
    - `level [Required]`: The hierarchy level this site represents (e.g., "factory", "line"). Must match a level defined in the Context.
    - `capabilityList [Optional]`: Defines capabilities to be added to the Context if this site node is processed for capability setup.
        - `capabilities [Required]`: An array of capability names (strings) to add/update in the Context.
    - `hierarchyLevels [Optional]`: Defines hierarchy levels to be added to the Context if this site node is processed for capability setup.
        - `levels [Required]`: An array of hierarchy level names (strings) to add/update in the Context.
    - `deploymentTargets [Optional]`: Defines deployment targets associated with this site.
        - `rbac [Optional]`: Default RBAC settings for targets under this site. Can be overridden per target.
            - `role [Required]`: Azure role to assign (e.g., "Contributor").
            - `userGroup [Required]`: Object ID of the user or group to assign the role to.
        - `capabilities [Optional]`: Default capabilities for targets under this site. Can be overridden per target. Array of strings.
        - `hierarchyLevel [Optional]`: Default hierarchy level for targets under this site. Can be overridden per target. String.
        - `namespace [Optional]`: Default Kubernetes namespace for targets under this site. (Note: Currently informational, not directly used in target creation command). Can be overridden per target. String.
        - `targetSpecFile [Required if 'targets' defined]`: File path to the JSON file containing the target specification.
        - `customLocationFile [Optional]`: File path to the JSON file containing the custom location ID. If not specified here or per target, it falls back to `created file while arc cluster creation`.
        - `targets [Required]`: An array of deployment target definitions.
            - `name [Required]`: Name of the deployment target resource.
            - `displayName [Required]`: Display name for the deployment target.
            - `capabilities [Optional]`: Overrides the parent `deploymentTargets.capabilities`. Array of strings.
            - `hierarchyLevel [Optional]`: Overrides the parent `deploymentTargets.hierarchyLevel`. String.
            - `namespace [Optional]`: Overrides the parent `deploymentTargets.namespace`. String.
            - `rbac [Optional]`: Overrides the parent `deploymentTargets.rbac`. Object with `role` and `userGroup`.
            - `customLocationFile [Optional]`: Overrides the parent `deploymentTargets.customLocationFile`. File path string.
            - `targetSpecFile [Optional]`: Overrides the parent `deploymentTargets.targetSpecFile`. File path string. (Less common to override per target).

## Pre-requisite for next step: Context creation (only if you are not using an existing context)

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

Once you create the context, add the required data in `onboarding-data.json` file.

```json
        "contextResourceGroup": "",
        "contextName": "",
        "contextSubscriptionId": "",
        "contextLocation": "",  
```

## Step 2: Workload orchestration resources onboarding

The workload orchestration setup script creates application specific resources such as capabilities, hierarchy lists, deployment targets, solutions, configs, and schemas.

Open a terminal and run the following command.

### [PowerShell](#tab/powershell)

```powershell
.\powershell\cm_onboarding.ps1 -onboardingFile mock-data.json
```

### [Python](#tab/python)

```python
python python/cm_onboarding.py mock-data.json
```

### [Bash](#tab/bash)

```bash
bash shell/cm_onboarding.sh mock-data.json
```
---

### Arguments

| PowerShell flag | Python / Bash flag | Default | Description |
|-----------------|--------------------|---------|-------------|
| `-skipResourceGroupCreation $true` | `--skip-resource-group-creation` | `false` | Skip creation of resource group |

### Input properties

The properties being used in this step fall under the `cmOnboarding` section in the `onboarding-data.json` file.

- `resourceGroup [Optional]`: Defines the resource group for creating CM resources. Overrides the common one.
- `subscriptionId [Optional]`: Defines the subscription for creating CM resources. Overrides the common one.
- `location [Optional]` (default: `eastus`) : Defines the location for creating CM resources. Overrides the common one.

- `schemas [Optional]`: Defines the schemas to be created.
  - `name [Required]`: Name of the schema.
  - `version [Required]`: Version of the schema.
  - `schemaFile [Required]`: File path of the schema definition.

- `configs [Optional]`: Defines the configurations to be created.
  - `name [Required]`: Name of the configuration.
  - `versionName [Required]`: Version name of the configuration.
  - `configFile [Required]`: File path of the configuration file.

- `solutions [Optional]`: Defines the solutions to be created.
  - `name [Required]`: Name of the solution.
  - `description [Required]`: Description of the solution.
  - `capabilities [Required]`: Array of capabilities for the solution.
  - `version [Required]`: Version of the solution.
  - `specificationFile[Required]` : File path to Specification File
  - `solutionTemplate [Required]`: Configuration template file path for the solution.

## FAQs

- `rbac.userGroup` should be the object ID of the user/group. For a user, this can be looked up on the Portal via the Entra ID tab.

## Known Issues

### Custom Location Creation Error with Azure CLI 2.70.0

When running the infrastructure onboarding script, you may encounter the following error during custom location creation:

```
'CredentialAdaptor' object has no attribute 'signed_session'

```

This issue specifically affects Azure CLI version 2.70.0 and occurs when using the `az customlocation create` command.

### Workaround

####  Create the Custom Location via Azure portal

1. Navigate to the [Azure portal](https://portal.azure.com)
2. Click "+ Create a resource" and search for "custom location"
3. In the "Basics" tab:
   - Select your subscription and resource group
   - Enter a name for your custom location
   - Select your Arc-enabled cluster
   - Select the appropriate extension (either `microsoft.testsymphonyex` or `microsoft.workloadorchestreation`)
   - Specify your namespace (the same value you'd use in the script)
4. Complete the creation process

After creating the custom location through the portal, run the script with `-skipCustomLocationCreation $true` to skip this step.

#### Additional Script Parameters

The infrastructure script supports skipping various components if they have already been created or if you're troubleshooting specific parts:

### [PowerShell](#tab/powershell)
```powershell
.\powershell\infra_onboarding.ps1 -onboardingFile "your-file.json" `
    -skipAzExtensions $true `
    -skipResourceGroupCreation $true `
    -skipAksCreation $true `
    -skipTcoDeployment $true `
    -skipCustomLocationCreation $true
```

### [Python](#tab/python)
```python
python python/infra_onboarding.py your-file.json \
    --skip-az-extensions \
    --skip-resource-group-creation \
    --skip-aks-creation \
    --skip-tco-deployment \
    --skip-custom-location-creation
```

### [Bash](#tab/bash)
```bash
bash shell/infra_onboarding.sh your-file.json \
    --skip-az-extensions \
    --skip-resource-group-creation \
    --skip-aks-creation \
    --skip-tco-deployment \
    --skip-custom-location-creation
```

---

Use these parameters as needed based on which components you have already created or want to skip.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

## Related content

- [Service groups with workload orchestration](service-group.md)
- [RBAC guide](rbac-guide.md)
- [Configuration template](configuring-template.md)
- [Configuration schema](configuring-schema.md)
