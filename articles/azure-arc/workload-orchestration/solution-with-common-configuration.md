---
title: Create a Basic Solution with Common Configurations with Workload Orchestration
description: Learn how to create a basic solution with common configurations using the workload orchestration via CLI. 
author: nathmanish
ms.author: sethm
ms.topic: quickstart
ms.date: 05/03/2025
ms.custom:
  - build-2025
# Customer intent: As a developer working with workload orchestration, I want to create a basic solution template with common configurations using CLI commands, so that I can streamline my deployment process and manage application dependencies effectively.
---

# Deploy a basic solution with common configurations

In this guide, you create a basic solution with common configurations using the workload orchestration via CLI. The common configurations are enabled by defining the configurable attributes at each hierarchical level that can be used for a particular solution.


## Prerequisites

- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](set-up-workload-orchestration.md).
- Download the artifacts from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

    [![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip) 

## Define the variables

Create the template and schema files by referring to *common-config.yaml*, *common-schema.yaml, and *app-config-template.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).

### [Bash](#tab/bash)

```bash
# Set environment variables
subId="<SUBSCRIPTION_ID>"
rg="<RESOURCE_GROUP_NAME>"
l="<LOCATION>"
childName="<TARGET_NAME>"

# Create variables for schema
# Enter schema name
schemaName="Common schema"
# Enter schema file name
schemaFile="common-schema.yaml"
# Enter schema version in x.x.x format
schemaVersion="1.0.0"
# Enter name for hierarchical configuration template
configName="CommonConfig"
# Enter hierarchical config file name
configFile="common-config.yaml"
# Enter hierarchical config version name
configVersion="1.0.0"

# Create variables for application/solution
# Enter name of application
appName="PriceDetector"
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
# Set environment variables
$subId="<SUBSCRIPTION_ID>"
$rg="<RESOURCE_GROUP_NAME>"
$l="<LOCATION>"
$childName="<TARGET_NAME>"

# Create variables for schema
# Enter schema name
$schemaName = "Common schema"
# Enter schema file name
$schemaFile = "common-schema.yaml"
# Enter schema version in x.x.x format
$schemaVersion = "1.0.0"
# Enter name for hierarchical configuration template
$configName = "CommonConfig"
# Enter hierarchical config file name
$configFile = "common-config.yaml"
# Enter hierarchical config version name
$configVersion = "1.0.0"

# Create variables for application/solution
# Enter name of application
$appName = "PriceDetector"
# Enter value in x.x.x format
$appVersion = "1.0.2"
# Enter description for application
$desc = "To calculate total by detecting price of each item"
# Enter capabilities of application
$appCapList1 = "[soap, conditioner]"
# Enter configuration template file name for the application 
$appConfig = "app-config-template.yaml"
```

***

## Create a configuration schema

Create the schema file by referring to *common-schema.yaml* from [GitHub repository](https://github.com/Azure/workload-orchestration).
   
```azurecli
az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --version "$schemaVersion" --schema-file "$schemaFile"
```

You can provide schema name and version in schema file instead of as CLI arguments. To do that, add the following section to the *shared-schema.yaml* file and run the previous command without `--schema-name` and `--version` arguments.

```yaml
metadata:
    name: <name> [optional]
    version: <version> [optional]
```

> [!TIP]
> You can view the created schema using `az workload-orchestration schema version show --resource-group "$rg" --schema-name "$schemaName" --version "$schemaVersion"`

## Set the hierarchy configuration

1. Create the hierarchy configuration template.

    ```azurecli
    az workload-orchestration config-template create --resource-group "$rg" --location "$l" --config-template-name "$configName" --version "$configVersion" --configuration-template-file "$configFile" --description "<description>"

1. Link the template to the Site of the desired hierarchy level, that is factory in this case.
    ```azurecli
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $siteId --context-id $contextId
    ```

    You can also view the linked hierarchies using:
    ```azurecli
    az workload-orchestration config-template hierarchy show -g "$rg" -n "$configName"
    ```

1. Set the common configuration for the Site.
    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "siteId" --template-name "$appName" --version $appVersion
    ```

> [!NOTE]
> You can also link a configuration template to a specific target instead of a Site, by specifying the target ID as value for parameter --hierarchy-ids. 


## Create the solution template 

Follow these steps to create a solution template for your application.

1. Create the *specs.json* and *app-config-template.yaml* files by referring to sample files from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). In *specs.json*, update the helm url, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*. Update the *app-config-template.yaml* file with proper reference to your schema that you created in the previous step, and add the following new configs to it.

    ```yaml
    SqlServerEndpoint: ${{$config(CommonConfig/version1,SqlServerEndpoint)}}
    LineHealthEndpoint: ${{$config(CommonConfig/version1,LineHealthEndpoint)}}
    ```

1. Create the solution template resource.

    ```azurecli
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName" --description "$desc" --capabilities "$appCapList1" --configuration-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

    Values for `--solution-template-name` and `--version` can be provided in the solution template file instead of as CLI arguments. If name or version is specified in both file and CLI, the values should match. Add the following section to the *app-config-template.yaml* file:

    ```yaml
    metadata:
      name: <name> [optional]
      version: <version> [optional]
    ```

    > [!NOTE]
    > The list of capabilities for a solution template should be a subset of the capabilities of the targets where the solution is intended to be deployed. To update the list of capabilities for an existing solution template, run `az workload-orchestration solution-template update-capabilities -n "$appName" --capabilities "<capability 1>" "<capability 2>" --description "$desc" --location $l -g $rg`.


## Deploy the solution

Run the following command to configure the solution template and deploy the corresponding solution/application to your target. The configuration values need to be stored in `config.yaml`.

```azurecli
az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-template-version-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName/versions/$appVersion" --configuration “config.yaml”   
```

<details>
<summary> You can also configure and deploy the solution in separate individual steps. </summary>

1. Set the configuration values for the solution.

    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName" --template-name "$appName" --version $appVersion --solution
    ```

    > [!NOTE]
    > To view the configuration values you have set, use `az workload-orchestration configuration show` with the same set of arguments. You can use the `--template-subscription` argument to set or show configurations for a template residing in an Azure subscription other than the current subscription.

1. Review the configurations for a particular target. This step ensures the configured values obey all schema rules and generates a solution version based on the solution template.

    ```azurecli
    az workload-orchestration target review --resource-group "$rg" --target-name "$childName" --solution-template-version-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName/versions/$appVersion"
    ```

1. Publish the solution. Enter `reviewId` from the previous command response.

    ```azurecli
    reviewId="<reviewId>"
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

    Completion of this step generates the final configuration of the solution after it is validated and approved, created by combining the schema, configuration template, and solution Helm chart. It represents a fully rendered, a predeployment ready, targeted solution.

1. Deploy the solution.

    ```azurecli
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

</details>

> [!TIP]
> You can also deploy the solution using the [Deploy tab in Workload orchestration portal](deploy.md)