---
title: Create a Basic Solution with Workload Orchestration
description: Learn how to create a basic solution without common configurations using the workload orchestration. 
author: nathmanish
ms.author: nathmanish
ms.topic: quickstart
ms.date: 09/05/2025
ms.custom:
  - build-2025
# Customer intent: As a developer, I want to create a basic solution using workload orchestration without common configurations, so that I can deploy applications efficiently with minimal setup.
---

# Deploy a basic solution 

Follow this guide to deploy a basic solution using workload orchestration.

## Prerequisites

- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](set-up-workload-orchestration.md).
- Download the artifacts from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

  [![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip)


## Define the variables

### [Bash](#tab/bash)

```bash
# Set environment variables
subId="<SUBSCRIPTION_ID>"
rg="<RESOURCE_GROUP_NAME>"
l="<LOCATION>"
childName="<TARGET_NAME>"

# Create variables for schema
schemaName="Sharedschema"
schemaFile="shared-schema.yaml"
# Enter value in x.x.x format
schemaVersion="1.0.0"

# Create variables for application/solution
# Enter name of application
appName="priceDetector"
# Enter value in x.x.x format
appVersion="1.0.0"
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
$schemaName = "Sharedschema"
$schemaFile = "shared-schema.yaml"
# Enter value in x.x.x format
$schemaVersion = "1.0.0"

# Create variables for application/solution
# Enter name of application
$appName = "priceDetector"
# Enter value in x.x.x format
$appVersion = "1.0.0"
# Enter description for application
$desc = "To calculate total by detecting price of each item"
# Enter capabilities of application
$appCapList1 = "[soap,conditioner]"
# Enter configuration template file name for the application 
$appConfig = "app-config-template.yaml"
```

***


## Create the solution template

Follow these steps to create a [solution template](configuration-model.md#solution-template) for your application.

1. Create the *specs.json* and *app-config-template.yaml* files by referring to sample files from the [GitHub repository](https://github.com/Azure/workload-orchestration). In *specs.json*, you can update the Helm URL and chart version in x.x.x format. The *app-config-template.yaml* file defines the configurable template parameters and the [schema](configuration-model.md#configuration-schema) validation rules governing them.

1. Create the solution template.

    ```azurecli
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName" --description "$desc" --capabilities "$appCapList1" --configuration-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

    Values for `--solution-template-name` and `--version` can be provided in the solution template file instead of as CLI arguments. If specified in both file and CLI, the values should match. You can add the following section to the *app-config-template.yaml* file and rerun the previous command without the two arguments:

    ```yaml
    metadata:
      name: <name> [optional]
      version: <version> [optional]
    ```

    > [!NOTE]
    > The list of capabilities for a solution template should be a subset of that of the targets the solution is intended to be deployed to. To update the list of capabilities for an existing solution template, run `az workload-orchestration solution-template update-capabilities -n "$appName" --capabilities "<capability 1>" "<capability 2>" --description "$desc" --location $l -g $rg`.

<details>
<summary> You can also create a schema object and refer to it for multiple templates. </summary>

1. Create the schema resource by referring to *shared-schema.yaml* from [GitHub repository](https://github.com/Azure/workload-orchestration).
   
    ```azurecli
    az workload-orchestration schema create --resource-group "$rg" --location "$l" --schema-name "$schemaName" --version "$schemaVersion" --schema-file "$schemaFile"
    ```

1. In the *app-config-template.yaml* file, replace the `schema` section with a reference to the schema resource you created.

    ```yaml
    schema:
        name: <schema name>
        version: <schema version>
    ```

1. Create the solution template.

</details>


## Deploy the solution

### [CLI](#tab/cli)

Run the following command to configure the solution template and deploy the corresponding solution or application to your target. Store the configuration values in *config.yaml*.

```azurecli
az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-template-name "$appName" --solution-template-version $appVersion --configuration “config.yaml”   
```

> [!NOTE]
> If your template resides in a different resource group, you can use the `--solution-template-rg` argument to specify your template resource group. 

<details>
<summary> You can also configure and deploy the solution in separate individual steps. </summary>

1. Set the configuration values for the solution.

    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName" --template-name "$appName" --version $appVersion --solution
    ```

    > [!NOTE]
    > To view the configuration values you set, use `az workload-orchestration configuration show` with the same set of arguments. You can use the `--template-subscription` argument to set or show configurations for a template residing in an Azure subscription other than the current subscription.

1. Review the configurations for a particular target. This step ensures the configured values obey all schema rules and generates a solution version based on the solution template.

    ```azurecli
    az workload-orchestration target review --resource-group "$rg" --target-name "$childName" --solution-template-version-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName/versions/$appVersion"
    ```

1. Publish the solution. Enter `reviewId` from the previous command response.

    ```azurecli
    reviewId="<reviewId>"
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

    Completion of this step generates the final configuration of the solution after it's validated and approved, created by combining the schema, configuration template, and solution Helm chart. It represents a fully rendered, a predeployment ready, targeted solution.

1. Deploy the solution.

    ```azurecli
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName/versions/$appVersion
    ```

</details>

### [Portal](#tab/portal)

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview).
1. Click on **Configure Solutions** on the left. The **Solutions** tab shows the status of all solutions deployed or pending deployment in your environment.

    :::image type="content" source="./media/configure-solutions.png" alt-text="Screenshot of the Configure tab showing how to apply filters1." lightbox="./media/configure-solutions.png":::

1. Search and select the name of your solution with configuration status *Configuration pending* and click on **Configure and publish**. 

    :::image type="content" source="./media/configure-solution-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to select a solution to configure it." lightbox="./media/configure-solution-1.png":::

1. Select one or multiple targets you want to deploy the solution to. Targets can be filtered by name, parent site, hierarchy level and capabilities, and grouped by parent site and hierarchy level. Click on **Next**.

    :::image type="content" source="./media/configure-solution-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure a solution and disable autopublish." lightbox="./media/configure-solution-2.png":::

1. In the **Configure target** step, you can set common configurations for all targets or click on the **custom target value** icon to set custom configuration values for selective targets. You can also click on **Previous Versions** to view configuration for previously deployed versions of this solution. Once done, click on **Next**.

    :::image type="content" source="./media/configure-solution-3.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the parameters to configure the targets." lightbox="./media/configure-solution-3.png":::
    :::image type="content" source="./media/configure-solution-3-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the parameters to configure the targets1." lightbox="./media/configure-solution-3-1.png":::

1. Review the final configurations and click on **Publish** to create a new revision of configuration values for the selected targets. Once completed, the new solution version (or revision) is published for each target where the configurations were resolved successfully.

    :::image type="content" source="./media/configure-solution-7.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to publish the configuration of a solution target." lightbox="./media/configure-solution-7.png":::

1. Click on the **Deploy** tab and select the target you want to deploy the solution to.
    :::image type="content" source="./media/single-deploy-1.png" alt-text="Screenshot of the Deploy tab showing how to click on a target." lightbox="./media/single-deploy-1.png":::

1. Select your newly published solution from the list. Make sure it is in **Publish completed** state. Click on **Deploy Solution** and confirm.
    :::image type="content" source="./media/single-deploy-2.png" alt-text="Screenshot of the Deploy tab showing how to click on a target1." lightbox="./media/single-deploy-2.png":::

1. You can monitor the deployment progress by clicking on the status of the solution you deployed. This opens the **Status details** pane showing all the intermediate steps of the operation, along with date and time of completion and the user who initiated it.

    :::image type="content" source="./media/single-deploy-5.png" alt-text="Screenshot of the Deploy tab showing the deployment status details1." lightbox="./media/single-deploy-5.png":::

***

## Next steps

Once you know how to create a basic solution, you can explore more advanced scenarios. For example, check out how to [Create a solution with common configurations](solution-with-common-configuration.md), which is an extension of this guide.

