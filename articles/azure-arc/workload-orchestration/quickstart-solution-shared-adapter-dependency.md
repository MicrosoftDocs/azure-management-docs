---
title: Create a Solution with Shared Adapter Dependency with Workload Orchestration
description: Learn how to create a solution with shared adapter dependency using workload orchestration via CLI.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 05/07/2025
ms.custom:
  - build-2025
# Customer intent: "As a cloud architect, I want to create a solution that utilizes shared adapter dependencies through CLI, so that I can effectively manage data synchronization across different application targets."
---

# Tutorial: Create a solution with shared adapter dependency

In this tutorial, you use the workload orchestration via CLI to create a Factory Sensor Anomaly Detector (FSAD) solution which is dependent on a Shared Sync Adapter (SSA) solution. A shared sync adapter is a component used in various solutions to manage data synchronization between devices and servers.

The FSAD solution is deployed on a child target, while the SSA solution is deployed on a parent target. The FSAD solution uses the SSA solution to synchronize data between devices and servers.

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).


## Description of the user scenario

The following example illustrates the user scenario of creating a solution with shared adapter dependency.

1. A user creates an SSA instance, *ssa-instance-1*, using a portal or CLI. This instance is then added as a dependency to FSAD. This means FSAD relies on *ssa-instance-1* for its configuration or functionality.
1. When FSAD is deployed to a target, for example Chicago, the Chicago-specific configuration is injected into the SSA configuration of *ssa-instance-1*. If FSAD is later deployed to another target, for example Redmond, the Redmond-specific configuration is appended to the existing configuration of *ssa-instance-1*. SSA configurations are **target-specific**, which means they are not shared across different targets.
    - After deploying FSAD to Chicago, *ssa-instance-1* has Chicago-specific settings.
    - After deploying FSAD to Redmond, *ssa-instance-1* has both Chicago and Redmond configurations.

1. The user creates a second SSA instance, *ssa-instance-2*, and adds it as a dependency for FSAD for specific targets. During deployment, the service only looks for configurations of FSAD that depend on *ssa-instance-2*. It ignores configurations related to *ssa-instance-1.* Dependencies are **instance-specific**. If FSAD is configured to depend on *ssa-instance-2* for certain targets, only those targets inject configurations into *ssa-instance-2*.
1. When FSAD is uninstalled from Chicago, the Chicago-specific configuration is removed from the SSA instance it depends on, which is *ssa-instance-1*. Uninstalling FSAD from a target cleans up the configurations injected into the SSA instance for that target.
    - If FSAD is uninstalled from Chicago, the Chicago-specific settings are removed from *ssa-instance-1*.
    - If FSAD is uninstalled from Redmond, the Redmond-specific settings are removed from *ssa-instance-1*.


> [!NOTE]
> Service only updates the configmap of solution in K8s edge. It isn't restarting the FSAD if deployment is via Helm. If you want the solution to restart on every configuration change, you have to author the Helm chart in such a way to ensure application would restart on configuration update. For more information, see [Kubernetes K8 guide](https://www.baeldung.com/ops/kubernetes-restart-configmap-updates)


## Configuration templating for FSAD and SSA

A configuration template is a YAML file that defines the configuration parameters for a solution. The configuration template is used to create a configuration schema, which is a JSON file that defines the structure of the configuration data. 

> [!NOTE]
> Check out [Configuration template](configuring-template.md) and [Configuration schema](configuring-schema.md) for the list of rules used to define the template and schema for configurations and details to write conditional or nested expressions in the schema for custom validations.

Sample of a FSAD configuration template:

```yaml
dependencies:
    - solutionTemplateId: <solution template id of SSA>
      configsToBeInjected:
          - from: AppConfig
            to:dependentAppConfigs                 
configs:
  AppConfig:
     TargetName: ${{$val(LineName)}}
     TargetTag: ${{$val(LineTag)}}     
     AppName: FSAD

```

Sample of an SSA configuration template:

```yaml            
configs:
  AppName: SSA
  DepedentAppConfigs: []   #It should be array or null

```

`AppConfig` of the FSAD configuration template changes at every target and target specific `AppConfig` gets injected into `DependantAppConfigs` of SSA configuration template.


## Define the variables for solution templating

Create the template and schema YAML files by referring to *shared-schema.yaml* and *app-config-template.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).

### [Bash](#tab/bash)

The following variables are used in the commands. You can change the values of these variables as per your requirements.

```bash
# Create variables for schema
# Enter schema name
schemaName="SSAschema"
app2SchemaName="FSADschema"
# Enter schema file name
schemaFile="ssa-schema.yaml"
app2SchemaFile="fsad-schema.yaml"
# Enter value in x.x.x format
schemaVersion="1.0.0"
app2SchemaVersion="1.0.1"

# Create variables for application/solution
# Enter name of application
appName1="SSA"
appName2="FSAD"

# Enter description for application
desc="To manage data synchronization between devices and servers"
desc2="Factory Sensor Anomaly Detector"
# Enter capabilities of application
appCapList1="[soap,conditioner]"
appCapList2="[soap]"
# Enter configuration template file name for the application 
appConfig="ssa-config-template.yaml"
appConfig2="fsad-config-template.yaml"
# Enter value in x.x.x format
appVersion="1.0.0"
app2Version="1.0.1"
```

### [PowerShell](#tab/powershell)

The following variables are used in the commands. You can change the values of these variables as per your requirements.

```powershell
# Create variables for schema
# Enter schema name
$schemaName = "SSAschema"
$app2SchemaName = "FSADschema"
# Enter schema file name
$schemaFile = "ssa-schema.yaml"
$app2SchemaFile = "fsad-schema.yaml"
# Enter value in x.x.x format
$schemaVersion = "1.0.0"
$app2SchemaVersion = "1.0.1"

# Create variables for application/solution
# Enter name of application
$appName1 = "SSA"
$appName2 = "FSAD"

# Enter description for application
$desc = "To manage data synchronization between devices and servers"
$desc2 = "Factory Sensor Anomaly Detector"
# Enter capabilities of application
$appCapList1 = "[soap,conditioner]"
$appCapList2 = "[soap]"
# Enter configuration template file name for the application 
$appConfig = "ssa-config-template.yaml"
$appConfig2 = "fsad-config-template.yaml"
# Enter value in x.x.x format
$appVersion = "1.0.0"
$app2Version = "1.0.1"
```

***


> [!NOTE]
> Variables need to be defined each time a new terminal is opened up.

## Create a target at parent level for SSA

### [Bash](#tab/bash)

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```bash
    customLocationName=$(az resource list --resource-type "Microsoft.ExtendedLocation/customLocations" --resource-group "$rg" --name "$clusterName-Location" --query "[].id" --output tsv)
    ```

1. Create a target at parent level. Ensure *custom-location.json* is updated with the created custom location's ID.

    ```bash
    parentName="<parent_name>"
    parentDesc="<parent_description>"
    level1="<hierarchy_level>"
    capParentList="<capabilities_list>"
    parentScope="$parentName-scope"

    az workload-orchestration target create \
        --resource-group "$rg" \
        --location "$l" \
        --name "$parentName" \
        --display-name "$parentName" \
        --hierarchy-level "$level1" \
        --capabilities "$capParentList" \
        --description "$parentDesc" \
        --solution-scope "$parentScope" \
        --target-specification "@targetspecs.json" \
        --extended-location "@custom-location.json" \
        --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextParentName"
    ```

### [PowerShell](#tab/powershell)

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```powershell
    $customLocationName = az resource list --resource-type "Microsoft.ExtendedLocation/customLocations" --resource-group $rg --name "$clusterName-Location" --query "[].id" --output tsv

    # Create a target at parent level. Ensure custom-location.json is updated with the created custom location's ID.

    $parentName = "<parent_name>"
    $parentDesc = "<parent_description>"
    $level1 = "<hierarchy_level>"
    $capParentList = "<capabilities_list>"
    $parentScope = "$parentName-scope"

    az workload-orchestration target create `
        --resource-group $rg `
        --location $l `
        --name $parentName `
        --display-name $parentName `
        --hierarchy-level $level1 `
        --capabilities $capParentList `
        --description $parentDesc `
        --solution-scope $parentScope `
        --target-specification "@targetspecs.json" `
        --extended-location "@custom-location.json" `
        --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextParentName
    ```


***

## Create a schema for Shared Sync Adapter (SSA)

Create the shared solution schema. The following command takes version input from CLI argument:

### [Bash](#tab/bash)

```bash
az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --version "$schemaVersion" --schema-file "$schemaFile"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration schema create --resource-group $rg --location $l --schema-name $schemaName --version $schemaVersion --schema-file $schemaFile
```

***

## Create a Shared Sync Adapter (SSA) solution template

### [Bash](#tab/bash)

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the previous step.
1. Create the SSA solution template using the following command. The following command takes version input from CLI argument:

    ```bash
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName1" --description "$desc" --capabilities "$appCapList1" --config-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

### [PowerShell](#tab/powershell)

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the previous step.
1. Create the SSA solution template using the following command. The following command takes version input from CLI argument:

    ```powershell
    az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName1 --description $desc --capabilities $appCapList1 --config-template-file $appConfig --specification "@specs.json" --version $appVersion
    ```

***

## Create a target at child level for FSAD

Create a target at child level. Ensure *custom-location.json* is updated with the created custom location's ID.

### [Bash](#tab/bash)

```bash	
az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description "$childDesc" --solution-scope "$childName-scope" --target-specification '@targetspecs.json' --extended-location '@custom-location.json' --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextChildName"
```

> [!NOTE]
> Update the parameter `solutionTemplateId` under dependencies section of FSAD configuration template and schema with the SSA solution template ID.
> The ``solution-template create` command displays the ID along with solution template version.
>
> ```bash
> solutionTemplateId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutiontemplates/$appName1"
> ```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description "$childDesc" --solution-scope "$childName-scope" --target-specification '@targetspecs.json' --extended-location '@custom-location.json' --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextChildName
```

> [!NOTE]
> Update the parameter `solutionTemplateId` under dependencies section of FSAD configuration template and schema with the SSA solution template ID.
> The ``solution-template create` command displays the ID along with solution template version.
>
> ```powershell
> $solutionTemplateId = "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutiontemplates/$appName1"
> ```

***

## Create a schema for FSAD

Create the shared solution schema. The following command takes version input from CLI argument:

### [Bash](#tab/bash)

```bash
az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$app2SchemaName" --version "$app2SchemaVersion" --schema-file "$app2SchemaFile"
```

### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration schema create --resource-group $rg --location $l --schema-name $app2SchemaName --version $app2SchemaVersion --schema-file $app2SchemaFile
```

***

## Create a FSAD solution template

### [Bash](#tab/bash)

1. Create a *fsad-specs.json* file by referring to *fsad-specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *fsad-specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
1. Create the FSAD solution template using the following command:

    ```bash
    # Any modifications to solution files will necessitate version update.

    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName2" --description "$desc2" --capabilities "$appCapList2" --config-template-file "$appConfig2" --specification "@fsad-specs.json" --version "$app2Version"
    ```

### [PowerShell](#tab/powershell)

1. Create a *fsad-specs.json* file by referring to *fsad-specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *fsad-specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
1. Create the FSAD solution template using the following command:

    ```powershell
    # Any modifications to solution files will necessitate version update.

    az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName2 --description $desc2 --capabilities $appCapList2 --config-template-file $appConfig2 --specification "@fsad-specs.json" --version $app2Version
    ```

***

## Set the configuration values for the solution

### [Bash](#tab/bash)

1. View parameters of FSAD at child level.

    ```bash
    az workload-orchestration configuration show --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName2"
    ```

1. Edit parameters of FSAD at child level.

    ```bash
    az workload-orchestration configuration set --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName2"
    ```

### [PowerShell](#tab/powershell)

1. View parameters of FSAD at child level.

    ```powershell
    az workload-orchestration configuration show --resource-group $rg --target-name $childName --solution-template-name $appName2
    ```

1. Edit parameters of FSAD at child level.

    ```powershell
    az workload-orchestration configuration set --resource-group $rg --target-name $childName --solution-template-name $appName2
    ```

***

> [!TIP]
> You can also set the configuration values for the solution using the [Configure tab in Workload orchestration portal](configure.md)

## Review the configuration values

### [Bash](#tab/bash)

1. Run `target review` command for SSA with dependencies.

    ```bash
    # Fetch the appVersion from SSA solution create command
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName1" --solution-version "$appVersion" --target-name "$parentName"
    ```

1. Update the solution version ID, `Id`, from the output of previous command in the *dependencies.json* file. 
1. Run `target review` command for FSAD with dependencies. Ensure that the FSAD solution version has dependencies listed in the CLI output. 

    ```bash
    # Fetch the appVersion from FSAD solution create command
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName2" --solution-version "$app2Version" --target-name "$childName" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `solutionTemplateVersionId` from `solutionDependencies` in the output. Execute the `az rest` command and ensure FSAD configuration is injected.

    ```bash
    solutionTemplateVersionId="<solutionTemplateVersionId>"
    az rest --uri "$solutionTemplateVersionId?api-version=2025-01-01-preview" --method GET
    ```

1. In the CLI response, check `reviewId` and `name` values. The name displays the new solution template version.

### [PowerShell](#tab/powershell)

1. Run `target review` command for SSA with dependencies.

    ```powershell
    # Fetch the appVersion from SSA solution create command
    az workload-orchestration target review --resource-group $rg --solution-name $appName1 --solution-version $appVersion --target-name $parentName
    ```

1. Update the solution version ID, `Id`, from the output of previous command in the *dependencies.json* file. 
1. Run `target review` command for FSAD with dependencies. Ensure that the FSAD solution version has dependencies listed in the CLI output. 

    ```powershell
    # Fetch the appVersion from FSAD solution create command
    az workload-orchestration target review --resource-group $rg --solution-name $appName2 --solution-version $app2Version --target-name $childName --solution-dependencies "@dependencies.json"
    ```

1. Copy the `solutionTemplateVersionId` from `solutionDependencies` in the output. Execute the `az rest` command and ensure FSAD configuration is injected.

    ```powershell
    $solutionTemplateVersionId = "<solutionTemplateVersionId>"
    az rest --uri "$solutionTemplateVersionId?api-version=2025-01-01-preview" --method GET
    ```

1. In the CLI response, check `reviewId` and `name` values. The name displays the new solution template version.

***

## Deploy the solution

### [Bash](#tab/bash)

1. Run `target publish` to publish FSAD solution with dependencies. The command publishes the FSAD solution with dependencies to SSA. Enter the `reviewId` from the previous command response.

    ```bash
    reviewId="<reviewId>"
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version 
    ```

1. Run `target install` command to deploy the solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version
    ```

### [PowerShell](#tab/powershell)

1. Run `target publish` to publish FSAD solution with dependencies. The command publishes the FSAD solution with dependencies to SSA. Enter the `reviewId` from the previous command response.

    ```powershell
    $reviewId = "<reviewId>"
    az workload-orchestration target publish --resource-group $rg --target-name $childName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version
    ```

1. Run `target install` command to deploy the solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $childName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version
    ```

***

> [!TIP]
> You can also deploy the solution using the [Deploy tab in Workload orchestration portal](deploy.md)

