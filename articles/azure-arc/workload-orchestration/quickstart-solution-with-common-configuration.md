---
title: Create a Basic Solution with Common Configurations with Workload Orchestration
description: Learn how to create a basic solution with common configurations using the workload orchestration via CLI. 
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: quickstart
ms.date: 05/03/2025
ms.custom:
  - build-2025
---

# Quickstart: Create a basic solution with common configurations

In this quickstart, you create a basic solution with common configurations using the workload orchestration via CLI. The solution is a Helm chart that contains the application and its dependencies. The common configurations are used to define the configurable attributes at each hierarchical level that can be used for a particular solution.


## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).

## Define the variables for solution templating

Create the template and schema files by referring to *common-config.yaml*, *common-schema.yaml* and *app-config-template.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).

### [Bash](#tab/bash)

```bash
# Create variables for schema
# Enter schema name
schemaName="Common schema"
# Enter schema file name
schemaFile="common-schema.yaml"
# Enter schema version in x.x.x format
schemaVersion="1.0.0"
# Enter name for common config
configName="common-config"
# Enter config file name
configFile="common-config.yaml"
# Enter config version name
configVersion="1.0.0"

# Create variables for application/solution
# Change version number if you have tried multiple times

# Edit "app-config-template.yaml" & add these 2 new configs. Change names in $config lookup as per your resources.
#  SqlServerEndpoint: ${{$config(CommonConfig/version1,SqlServerEndpoint)}}
#  LineHealthEndpoint: ${{$config(CommonConfig/version1,LineHealthEndpoint)}}

# Enter name of application
appName1="PriceDetector"
# Enter value in x.x.x format
appVersion="1.0.2"
# Enter description for application
desc="To calculate total by detecting price of each item"
# Enter capabilities of application
appCapList1="[soap,conditioner]"
# Enter configuration template file name for the application 
appConfig="app-config-template.yaml"
```

### [PowerShell](#tab/powershell)

```powershell
# Create variables for schema
# Enter schema name
$schemaName = "Common schema"
# Enter schema file name
$schemaFile = "common-schema.yaml"
# Enter schema version in x.x.x format
$schemaVersion = "1.0.0"
# Enter name for common config
$configName = "common-config"
# Enter config file name
$configFile = "common-config.yaml"
# Enter config version name
$configVersion = "1.0.0"

# Create variables for application/solution
# Change version number if you have tried multiple times

# Edit "app-config-template.yaml" & add these 2 new configs. Change names in $config lookup as per your resources.
#  SqlServerEndpoint: $config(CommonConfig/version1,SqlServerEndpoint)
#  LineHealthEndpoint: $config(CommonConfig/version1,LineHealthEndpoint)

# Enter name of application
$appName1 = "PriceDetector"
# Enter value in x.x.x format
$appVersion = "1.0.2"
# Enter description for application
$desc = "To calculate total by detecting price of each item"
# Enter capabilities of application
$appCapList1 = "[soap,conditioner]"
# Enter configuration template file name for the application 
$appConfig = "app-config-template.yaml"
```

***

## Create the solution template

The solution template consists of a schema and a configuration template. Both are defined in YAML format. 

### Create a common schema

In this quickstart you use a common schema. Common schemas define configurable attributes at each hierarchical level that can be used for a particular solution.

#### [Bash](#tab/bash)

1. Create the common solution schema. The following command takes version input from CLI argument:

    ```bash
    # Create Shared Solution Schema
    az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --version "$schemaVersion" --schema-file "$schemaFile"
    
    # View Schema
    az workload-orchestration schema version show --resource-group "$rg" --schema-name "$schemaName" --version "$schemaVersion"
    ```

1. You can provide version on file instead of as a CLI argument. Add below section to the *shared-schema.yaml* file.

    ```yaml
    metadata:
        name: <name> [optional]
        version: <version> [optional]
    ```

1. Run the same CLI command without `--version argument`. The service takes version input from file.

    ```bash
    # Create Shared Solution Schema
    az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --schema-file "$schemaFile"
    ```

The name field is introduced for user to identify the resource name and its version the file refers to. If name is provided, then it should match `--schema-name` argument.

#### [PowerShell](#tab/powershell)

1. Create the common solution schema. The following command takes version input from CLI argument:

    ```powershell
    # Create Shared Solution Schema
    az workload-orchestration schema create --resource-group $rg --location $l --schema-name $schemaName --version $schemaVersion --schema-file $schemaFile
    
    # View Schema
    az workload-orchestration schema version show --resource-group $rg --schema-name $schemaName --version $schemaVersion
    ```

1. You can provide version on file instead of as a CLI argument. Add below section to the *shared-schema.yaml* file.

    ```yaml
    metadata:
        name: <name> [optional]
        version: <version> [optional]
    ```

1. Run the same CLI command without `--version` argument. The service takes version input from file.

    ```powershell
    # Create Shared Solution Schema
    az workload-orchestration schema create --resource-group $rg --location $l --schema-name $schemaName --schema-file $schemaFile
    ```

The name field is introduced for user to identify the resource name and its version the file refers to. If name is provided, then it should match `--schema-name` argument.

***

### Create the configuration

Create the configuration template. The following command takes version input from CLI argument:

#### [Bash](#tab/bash)

```bash
az workload-orchestration config-template create --resource-group "$rg" --location "$l" --config-template-name "$configName" --version "$configVersion" --configuration-template-file "$configFile" --description "<description>"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration config-template create --resource-group $rg --location $l --config-template-name $configName --version $configVersion --configuration-template-file $configFile --description "<description>"
```

***

### Create the solution

#### [Bash](#tab/bash)

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the above step.
1. Create the Helm solution. The following command takes version input from CLI argument:

    ```bash
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName1" --description "$desc" --capabilities "$appCapList1" --config-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

    Version can be provided on file instead of as a CLI argument. Add the following section to the *app-config-template.yaml* file:

    ```yaml
    metadata:
        name: <name> [optional]
        version: <version> [optional]
    ```

    Run the same CLI command without the `--version` argument. The service will take the version input from the file.

    ```bash
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName1" --description "$desc" --capabilities "$appCapList1" --config-template-file "$appConfig" --specification "@specs.json"
    ```

#### [PowerShell](#tab/powershell)

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. In your *specs.json* file, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the above step.
1. Create the Helm solution. The following command takes version input from CLI argument:

    ```powershell
        az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName1 --description $desc --capabilities $appCapList1 --config-template-file $appConfig --specification "@specs.json" --version $appVersion
        ```

        Version can be provided on file instead of as a CLI argument. Add the following section to the *app-config-template.yaml* file:

        ```yaml
        metadata:
            name: <name> [optional]
            version: <version> [optional]
        ```

        Run the same CLI command without the `--version` argument. The service will take the version input from the file.

        ```powershell
        az workload-orchestration solution-template create --resource-group $rg --location $l --solution-template-name $appName1 --description $desc --capabilities $appCapList1 --config-template-file $appConfig --specification "@specs.json"
        ```

***


### Set the configuration values for the solution

#### [Bash](#tab/bash)

1. View parameters at the parent level, for example, Contoso factory:

    ```bash
    az workload-orchestration configuration show --resource-group "$rg" --target-name "$parentName" --solution-template-name "$appName1"
    ```

1. Edit parameters at the parent level:

    ```bash
    az workload-orchestration configuration set --resource-group "$rg" --target-name "$parentName" --solution-template-name "$appName1"
    ```

1. View parameters at the child level:

    ```bash
    az workload-orchestration configuration show --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName1"
    ```

1. Edit parameters at the child level:

    ```bash
    az workload-orchestration configuration set --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName1"
    ```

#### [PowerShell](#tab/powershell)

1. View parameters at the parent level, for example, Contoso factory:

    ```powershell
    az workload-orchestration configuration show --resource-group $rg --target-name $parentName --solution-template-name $appName1
    ```

1. Edit parameters at the parent level:

    ```powershell
    az workload-orchestration configuration set --resource-group $rg --target-name $parentName --solution-template-name $appName1
    ```

1. View parameters at the child level:

    ```powershell
    az workload-orchestration configuration show --resource-group $rg --target-name $childName --solution-template-name $appName1
    ```

1. Edit parameters at the child level:

    ```powershell
    az workload-orchestration configuration set --resource-group $rg --target-name $childName --solution-template-name $appName1
    ```

***

> [!TIP]
> You can also set the configuration values for the solution using the [Configure tab in Workload orchestration portal](configure.md)

## Deploy the solution

### [Bash](#tab/bash)

1. Review the configurations for a particular target. In the CLI output, check `reviewId` and name. The name displays the new solution template version.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName1" --solution-version "$appVersion" --target-name "$childName"
    ```

1. Publish the generated configuration for deployment. Enter `reviewId` from the previous command response.

    ```bash
    reviewId="<reviewId>"
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$childName/solutions/$appName1/versions/$appVersion
    ```

1. Deploy the solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$childName/solutions/$appName1/versions/$appVersion
    ```

### [PowerShell](#tab/powershell)

1. Review the configurations for a particular target. In the CLI output, check `reviewId` and name. The name displays the new solution template version.

    ```powershell
    az workload-orchestration target review --resource-group $rg --solution-name $appName1 --solution-version $appVersion --target-name $childName
    ```

1. Publish the generated configuration for deployment. Enter `reviewId` from the previous command response.

    ```powershell
    $reviewId = "<reviewId>"
    az workload-orchestration target publish --resource-group $rg --target-name $childName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$childName/solutions/$appName1/versions/$appVersion
    ```

1. Deploy the solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $childName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/private.edge/targets/$childName/solutions/$appName1/versions/$appVersion
    ```

***

> [!TIP]
> You can also deploy the solution using the [Deploy tab in Workload orchestration portal](deploy.md)