---
title: Create a Solution with Multiple Dependencies with Workload Orchestration
description: Learn how to create a solution with multiple shared adapter dependencies using workload orchestration via CLI.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 05/07/2025
ms.custom:
  - build-2025
# Customer intent: As a cloud solution architect, I want to create a multi-dependency solution using workload orchestration via CLI so that I can efficiently manage data synchronization across various production lines in a factory environment.
---

# Create a solution with multiple shared adapter dependencies

In this tutorial, you create a solution with multiple shared adapter dependencies using workload orchestration via CLI. You will create a Factory Sensor Anomaly Detector (FSAD) solution that depends on a Shared Sync Adapter (SSA) solution. The FSAD solution is deployed on a child target, while the SSA solution is deployed on a parent target. The FSAD solution uses the SSA solution to synchronize data between devices and servers.

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).

## Description of the user scenario

The following scenario describes the use case for creating a solution with multiple shared adapter dependencies:

- FSAD instance needs to inject its configurations into SSA instance during deployment. This means SSA must be configured differently depending on which FSAD instance it serves.
- There are two instances of SSA, *ssa-instance-a* and *ssa-instance-b*. These instances are deployed at the factory level, meaning they are shared across multiple production lines in the factory. There are three production lines (targets): *Line01*, *Line02*, and *Line03*. 
- Configurations for FSAD at *Line01* and *Line02* must be injected into *ssa-instance-a* while the configurations for FSAD at *Line03* must be injected into *ssa-instance-b*.
- All instances of FSAD and SSA must be deployed in the same Kubernetes namespace. This ensures that they can communicate with each other and share resources within the same logical boundary.
- There is no change in custom location. You can use the custom location from the previous scenarios.
- The `solutionScope` variable is set to `"factory-namespace"`, which is the namespace where all these components will be deployed. If you want to rename this namespace, you must ensure it adheres to Kubernetes naming conventions (lowercase letters, numbers, and hyphens).

> [!TIP]
> You can see the quickstart on [Create a solution with shared adapter dependency](quickstart-solution-shared-adapter-dependency.md), which is a simplified version of this scenario.

> [!NOTE]
> Both SSA and FSAD have configuration called `DeploymentName` at line level. This is introduced to isolate instances of both SSA and FSAD from each other since all these instances are deployed in same name space. Ensure `DeploymentName` is unique and follows K8s object naming convention.
>
> You have to provide solution instance name for SSA Review command. Ensure it's unique and follows K8s object naming convention.

## Define the variables for solution templating

Create the template and schema YAML files by referring to *shared-schema.yaml* and *app-config-template.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).

### [Bash](#tab/bash)

The following variables are used in the commands. In this example, the factory level target is named *Redmond*, and the line level targets are named *Line01*, *Line02*, and *Line03*. You can change the values of these variables as per your requirements.

```bash
solutionScope="factory-namespace"

# Create targets
factory="Redmond"
line01="Line01"
line02="Line02"
line03="Line03"

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

The following variables are used in the commands. In this example, the factory level target is named *Redmond*, and the line level targets are named *Line01*, *Line02*, and *Line03*. You can change the values of these variables as per your requirements.

```powershell
$solutionScope = "factory-namespace"

# Create targets
$factory = "Redmond"
$line01 = "Line01"
$line02 = "Line02"
$line03 = "Line03"

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

## Create a target at factory and line levels

### [Bash](#tab/bash)

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```bash
    customLocationName=$(az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group $rg --name "$clusterName-Location" --query "[].id" --output tsv)
    ```

1. Create a target at parent level *Redmond*. Ensure *custom-location.json* is updated with the created custom location's ID.

    ```bash
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$factory" --display-name "$factory" --hierarchy-level "$level1" --capabilities "$capParentList" --description "$parentDesc" --solution-scope "$solutionScope" --target-specification "@targetspecs.json" --extended-location "@custom-location.json" --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

1. Create targets at line level *Line01*, *Line02*, and *Line03*. Ensure *custom-location.json* is updated with the created custom location's ID.

    ```bash
    # Create target at line level (Line01)
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$line01" --display-name "$line01" --hierarchy-level "$level2" --capabilities "$capChildList" --description "$childDesc" --solution-scope "$solutionScope" --target-specification "@targetspecs.json" --extended-location "@custom-location.json" --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"

    # Create target at line level (Line02)
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$line02" --display-name "$line02" --hierarchy-level "$level2" --capabilities "$capChildList" --description "$childDesc" --solution-scope "$solutionScope" --target-specification "@targetspecs.json" --extended-location "@custom-location.json" --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"

    # Create target at line level (Line03)
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$line03" --display-name "$line03" --hierarchy-level "$level2" --capabilities "$capChildList" --description "$childDesc" --solution-scope "$solutionScope" --target-specification "@targetspecs.json" --extended-location "@custom-location.json" --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

### [PowerShell](#tab/powershell)

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```powershell
    $customLocationName = az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group $rg --name "$clusterName-Location" --query "[].id" --output tsv
    ```

1. Create a target at parent level *Redmond*. Ensure *custom-location.json* is updated with the created custom location's ID.

    ```powershell
    az workload-orchestration target create --resource-group $rg --location $l --name $factory --display-name $factory --hierarchy-level $level1 --capabilities $capParentList --description $parentDesc --solution-scope $solutionScope --target-specification @targetspecs.json --extended-location @custom-location.json --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

1. Create targets at line level *Line01*, *Line02*, and *Line03*. Ensure *custom-location.json* is updated with the created custom location's ID.

    ```powershell
    # Create target at line level (Line01)
    az workload-orchestration target create --resource-group $rg --location $l --name $line01 --display-name $line01 --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope $solutionScope --target-specification @targetspecs.json --extended-location @custom-location.json --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName

    # Create target at line level (Line02)
    az workload-orchestration target create --resource-group $rg --location $l --name $line02 --display-name $line02 --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope $solutionScope --target-specification @targetspecs.json --extended-location @custom-location.json --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName

    # Create target at line level (Line03)
    az workload-orchestration target create --resource-group $rg --location $l --name $line03 --display-name $line03 --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope $solutionScope --target-specification @targetspecs.json --extended-location @custom-location.json --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

***

## Create schema for Shared Sync Adapter (SSA)

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
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the above step.
1. Create the SSA solution template using the following command:

    ```bash
    # Create Helm Solution / Application
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName1" --description "$desc" --capabilities "$appCapList1" --config-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

> [!NOTE]
> Update the parameter `solutionTemplateId` under dependencies section of FSAD configuration template and schema with the SSA solution template ID.
> The ``solution-template create` command will display the ID along with solution template version.
>
> ```bash
> solutionTemplateId="/subscriptions/$subId/resourceGroups/$rg/ providers/Microsoft.Edge/solutiontemplates/$appName1"
> ```

### [PowerShell](#tab/powershell)

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the above step.
1. Create the SSA solution template using the following command:

    ```powershell
    # Create Helm Solution / Application
    az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName1 --description $desc --capabilities $appCapList1 --config-template-file $appConfig --specification @specs.json --version $appVersion
    ```

> [!NOTE]
> Update the parameter `solutionTemplateId` under dependencies section of FSAD configuration template and schema with the SSA solution template ID.
> The ``solution-template create` command will display the ID along with solution template version.
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
    # Any modifications to solution files will necessitate a version update.
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName2" --description "$desc2" --capabilities "$appCapList2" --config-template-file "$appConfig2" --specification "@fsad-specs.json" --version "$appVersion"
    ```

### [PowerShell](#tab/powershell)

1. Create a *fsad-specs.json* file by referring to *fsad-specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *fsad-specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
1. Create the FSAD solution template using the following command:

    ```powershell
    # Any modifications to solution files will necessitate a version update.
    az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName2 --description $desc2 --capabilities $appCapList2 --config-template-file $appConfig2 --specification @fsad-specs.json --version $appVersion
    ```

***


## Set the configuration values for the solution

### [Bash](#tab/bash)

1. Set the configuration values for SSA.

    ```bash
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factory" --template-name "$appName" --version $appVersion --solution
    ```

1. Set the configuration values for FSAD at line level. Configuration `DeploymentName` should be unique for each line.

    ```bash
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01" --template-name "$appName2" --version $appVersion --solution
    
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02" --template-name "$appName2" --version $appVersion --solution
    
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03" --template-name "$appName2" --version $appVersion --solution
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration values for SSA at factory and line levels.

    ```powershell
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factory" --template-name "$appName" --version $appVersion --solution
    ```

1. Set the configuration values for FSAD at line level. Configuration `DeploymentName` should be unique for each line.

    ```powershell
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01" --template-name "$appName2" --version $appVersion --solution
    
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02" --template-name "$appName2" --version $appVersion --solution
    
    az workload-orchestration configuration set --template-resource-group "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03" --template-name "$appName2" --version $appVersion --solution
    ```

***

> [!TIP]
> You can also set the configuration values for the solution using the [Configure tab in Workload orchestration portal](configure.md)

## Deploy the solution FSAD at *Line01*

### Review the configurations of FSAD at *Line01*

#### [Bash](#tab/bash)

1. Verify that FSAD *Line01* and FSAD *Line02* depend on *ssa-instance-a* at factory level.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --target-name "$factory" --solution-name "$appName" --solution-version "$appVersion" --solution-instance-name "ssa-instance-a"
    ```

1. Verify that FSAD is targeted to *Line01*. Replace the `solutionVersionId` in *dependencies.json* file with ID from the previous command response.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --target-name "$line01" --solution-name "$appName2" --solution-version "$app2Version" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```bash
    reviewId="<reviewId>"
    version="<name>"
    ```

#### [PowerShell](#tab/powershell)

1. Verify that FSAD *Line01* and FSAD *Line02* depend on *ssa-instance-a* at factory level.

    ```powershell
    az workload-orchestration target review --resource-group $rg --target-name $factory --solution-name $appName --solution-template-version $appVersion --solution-instance-name "ssa-instance-a"
    ```

1. Verify that FSAD is targeted to *Line01*. Replace the `solutionVersionId` in *dependencies.json* file with ID from the previous command response.

    ```powershell
    az workload-orchestration target review --resource-group $rg --target-name $line01 --solution-name $appName2 --solution-version $app2Version --solution-dependencies @dependencies.json
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```powershell
    $reviewId = "<reviewId>"
    $version = "<name>"
    ```

***

> [!TIP]
> You can also deploy the solution using the [Deploy tab in Workload orchestration portal](deploy.md)

### Publish and install the solution FSAD at *Line01*

#### [Bash](#tab/bash)

1. Run `target publish` to publish FSAD solution at *Line01*.

    ```bash
    az workload-orchestration target publish --resource-group "$rg"  --target-name "$line01" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01/solutions/$appName2/versions/$app2Version
    ```


1. Run `target install` to install FSAD solution at *Line01*.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$line01" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01/solutions/$appName2/versions/$app2Version
    ```


1. Make sure all configurations are updated at edge. 

    ```bash
    kubectl config set-context --current --namespace "$solutionScope"
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config
    
    kubectl describe configmap FSAD-config-"$DeploymentName" # DeploymentName set for FSAD config for Line01
    ```

#### [PowerShell](#tab/powershell)

1. Run `target publish` to publish FSAD solution at *Line01*.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $line01 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01/solutions/$appName2/versions/$app2Version
    ```

1. Run `target install` to install FSAD solution at *Line01*.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $line01 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line01/solutions/$appName2/versions/$app2Version
    ```

1. Make sure all configurations are updated at edge. 

    ```powershell
    kubectl config set-context --current --namespace $solutionScope
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config
    
    kubectl describe configmap FSAD-config-"$DeploymentName" # DeploymentName set for FSAD config for Line01
    ```

***

## Deploy the solution FSAD at *Line02*

### Review the configurations of FSAD at *Line02*

#### [Bash](#tab/bash)

1. Verify that FSAD is targeted to *Line02*. Replace the `solutionVersionId` in *dependencies.json* file with ID from the command response in the [Review the configurations of FSAD at *Line01*](#review-the-configurations-of-fsad-at-line01) section.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --target-name "$line02" --solution-name "$appName2" --solution-version "$app2Version" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```bash
    reviewId="<reviewId>"
    version="<name>"
    ```

#### [PowerShell](#tab/powershell)

1. Verify that FSAD is targeted to *Line02*. Replace the `solutionVersionId` in *dependencies.json* file with ID from the command response in the [Review the configurations of FSAD at *Line01*](#review-the-configurations-of-fsad-at-line01) section.

    ```powershell
    az workload-orchestration target review --resource-group $rg --target-name $line02 --solution-name $appName2 --solution-version $app2Version --solution-dependencies @dependencies.json
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```powershell
    $reviewId = "<reviewId>"
    $version = "<name>"
    ```

***

### Publish and install the solution FSAD at *Line02*

#### [Bash](#tab/bash)

1. Run `target publish` to publish FSAD solution at *Line02*.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$line02" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02/solutions/$appName2/versions/$app2Version 
    ```

1. Run `target install` to install FSAD solution at *Line02*.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02/solutions/$appName2/versions/$app2Version --target-name "$line02"
    ```

1. Make sure all configurations are updated at edge. 

    ```bash
    kubectl config set-context --current --namespace "$solutionScope"
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config
    
    kubectl describe configmap FSAD-config-"$DeploymentName" # DeploymentName set for FSAD config for Line02
    ```

#### [PowerShell](#tab/powershell)

1. Run `target publish` to publish FSAD solution at *Line02*.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $line02 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02/solutions/$appName2/versions/$app2Version 
    ```

1. Run `target install` to install FSAD solution at *Line02*.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $line02 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line02/solutions/$appName2/versions/$app2Version
    ```

1. Make sure all configurations are updated at edge. 

    ```powershell
    kubectl config set-context --current --namespace $solutionScope
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config
    
    kubectl describe configmap FSAD-config-"$DeploymentName" # DeploymentName set for FSAD config for Line02
    ```

***

## Deploy the solution FSAD at *Line03*

### Review the configurations of FSAD at *Line03*

#### [Bash](#tab/bash)

1. Verify that FSAD at line level *Line03* depends on *ssa-instance-b*. The *ssa-instance-b* instance should have unique `DeploymentName`.

    ```bash    
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName" --solution-version "$appVersion" --target-name "$factory" --solution-instance-name "ssa-instance-b"
    ```

1. For FSAD in *Line03*, replace the `solutionVersionId` in *dependencies.json* with ID from the previous command response.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --target-name "$line03" --solution-name "$appName2" --solution-version "$app2Version" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```bash
    reviewId="<reviewId>"
    version="<name>"
    ```

#### [PowerShell](#tab/powershell)

1. Verify that FSAD at line level *Line03* depends on *ssa-instance-b*. The *ssa-instance-b* instance should have unique `DeploymentName`.

    ```powershell    
    az workload-orchestration target review --resource-group $rg --solution-name $appName --solution-version $appVersion --target-name $factory --solution-instance-name "ssa-instance-b"
    ```

1. For FSAD in *Line03*, replace the `solutionVersionId` in *dependencies.json* with ID from the previous command response.

    ```powershell
    az workload-orchestration target review --resource-group $rg --target-name $line03 --solution-name $appName2 --solution-version $app2Version --solution-dependencies @dependencies.json
    ```

1. Copy the `reviewId` and `name` from the previous command and set the following variables:

    ```powershell
    $reviewId = "<reviewId>"
    $version = "<name>"
    ```

***

### Publish and install the solution FSAD at *Line03*

#### [Bash](#tab/bash)

1. Run `target publish` to publish FSAD solution at *Line03*.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$line03" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03/solutions/$appName2/versions/$app2Version 
    ```

1. Run `target install` to install FSAD solution at *Line03*.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$line03" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03/solutions/$appName2/versions/$app2Version
    ```

1. Make sure all configurations are updated at edge. 

    ```bash
    kubectl config set-context --current --namespace "$solutionScope"
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config for instance-B
    
    kubectl describe configmap FSAD-config-"$DeploymentName" # DeploymentName set for FSAD config for Line03
    ```

#### [PowerShell](#tab/powershell)

1. Run `target publish` to publish FSAD solution at *Line03*.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $line03 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03/solutions/$appName2/versions/$app2Version
    ```

1. Run `target install` to install FSAD solution at *Line03*.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $line03 --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$line03/solutions/$appName2/versions/$app2Version
    ```

1. Make sure all configurations are updated at edge. 

    ```powershell
    kubectl config set-context --current --namespace $solutionScope
    kubectl describe configmap SSA-config-$DeploymentName  # DeploymentName set for SSA config for instance-B
    
    kubectl describe configmap FSAD-config-$DeploymentName # DeploymentName set for FSAD config for Line03
    ```

***



## Uninstall FSAD from Line01 and Line02

If any dependent app, for example, FSAD is uninstalled from any target line, then the corresponding configurations are removed from SSA instances. If there are zero deployed instances of FSAD, then SSA is automatically uninstalled. 

### [Bash](#tab/bash)

1. Uninstall FSAD from *Line01*.

    ```bash
    az workload-orchestration target uninstall --resource-group "$rg" --target-name "$line01" --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName2
    ```

    Verify whether FSAD *Line01* configurations are removed from *ssa-instance-a* instance using `kubectl` command.
    
    ```bash    
    kubectl config set-context --current --namespace "$solutionScope"
    kubectl describe configmap SSA-config-"$DeploymentName"  # DeploymentName set for SSA config for instance-A
    ```

1. Uninstall FSAD from *Line02*.

    ```bash
    az workload-orchestration target uninstall --resource-group "$rg" --target-name "$line02" --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName2
    ```

    Verify whether *ssa-instance-a* instance has been uninstalled.

### [PowerShell](#tab/powershell)

1. Uninstall FSAD from *Line01*.

    ```powershell
    az workload-orchestration target uninstall --resource-group $rg --target-name $line01 --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName2
    ```

    Verify whether FSAD *Line01* configurations are removed from *ssa-instance-a* instance using `kubectl` command.
    
    ```powershell
    kubectl config set-context --current --namespace $solutionScope
    kubectl describe configmap SSA-config-$DeploymentName  # DeploymentName set for SSA config for instance-A
    ```

1. Uninstall FSAD from *Line02*.

    ```powershell
    az workload-orchestration target uninstall --resource-group $rg --target-name $line02 --solution-template-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName2
    ```

    Verify whether *ssa-instance-a* instance has been uninstalled.

***