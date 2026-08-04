---
title: Create a Basic Solution with Common Configurations with Workload Orchestration
description: Learn how to create a basic solution with common configurations using the workload orchestration. 
author: nathmanish
ms.author: sethm
ms.topic: quickstart
ms.date: 05/03/2025
ms.custom:
  - build-2025
# Customer intent: As a developer working with workload orchestration, I want to create a basic solution template with common configurations using CLI commands, so that I can streamline my deployment process and manage application dependencies effectively.
---

# Deploy a solution with common configurations

In this guide, you create a solution with common configurations using workload orchestration. The common configurations are enabled by defining the configurable attributes at each hierarchical level or target that are applied to all deployed solutions.


## Prerequisites

- Set up the required resources for workload orchestration. If you haven't, refer to [Set up workload orchestration](set-up-workload-orchestration.md).
- Download the artifacts from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

    [![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip) 

## Define the variables

This guide uses the same environment, schema, and application variables defined in [Deploy a basic solution](solution-without-common-configuration.md#define-the-variables). Set those variables first, then define the following additional variables for the hierarchical configuration template.

### [Bash](#tab/bash)

```bash
# Create variables for hierarchical configuration template
configName="CommonConfig"
configFile="common-config.yaml"
configVersion="1.0.0"
```

### [PowerShell](#tab/powershell)

```powershell
# Create variables for hierarchical configuration template
$configName = "CommonConfig"
$configFile = "common-config.yaml"
$configVersion = "1.0.0"
```

***


## Set the hierarchy configuration

1. Create the [hierarchy configuration template](configuration-model.md#hierarchy-configuration-template) by referring to the sample *common-config.yaml* in the [GitHub repository](https://github.com/Azure/workload-orchestration).

    ```azurecli
    az workload-orchestration config-template create --resource-group "$rg" --location "$l" --config-template-name "$configName" --version "$configVersion" --configuration-template-file "$configFile" --description "<description>"
    ```

1. Link the template to the Site of the desired hierarchy level, that is factory in this case.
    ```azurecli
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $siteId --context-id $contextId
    ```

    You can also view the linked hierarchies using:
    ```azurecli
    az workload-orchestration config-template hierarchy show -g "$rg" -n "$configName"
    ```

1. Set the common configuration for the Site.

    ### [CLI](#tab/cli)
    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "siteId" --template-name "$appName" --version $appVersion
    ```

    ### [Portal](#tab/portal)
    1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview) and click on **Configure Solutions** on the left.

    1. Select the name of the factory you want to configure, whose configuration status is not *Configuration up to date*, and click on **Configure**. 

        :::image type="content" source="./media/configure-line-1.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to select a line." lightbox="./media/configure-line-1.png":::

    1. In case there is a single configuration template linked to the hierarchy level, you can directly configure the values. In case of multiple templates, select the templates to be configured from the **Select Version** screen and click on **Next**. You can also filter templates by Name, Version and Status.

        :::image type="content" source="./media/configure-line-2.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to select a line1." lightbox="./media/configure-line-2.png":::

    1. In the **Configure** step, enter the parameters for each template and click on **Next**. You can choose to autofill values from previous version only if configuration template was previously configured.

        :::image type="content" source="./media/configure-line-3.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to enter the parameters to configure a target." lightbox="./media/configure-line-3.png":::

    1. Review the details and click on **Confirm** to apply the changes.

        :::image type="content" source="./media/configure-line-4.png" alt-text="Screenshot of the line tab in workload orchestration portal showing how to review and apply the changes of the configuration." lightbox="./media/configure-line-4.png":::

***

> [!NOTE]
> You can also link a configuration template to a specific target instead of a Site, by specifying the target ID as value for `--hierarchy-ids`. 


## Create the solution template 

Follow these steps to create a [solution template](configuration-model.md#solution-template) for your application.

1. Create the *specs.json* and *app-config-template.yaml* files by referring to sample files from the [GitHub repository](https://github.com/Azure/workload-orchestration). In *specs.json*, you can update the Helm URL and chart version in x.x.x format. The *app-config-template.yaml* file defines the configurable template parameters and the [schema](configuration-model.md#configuration-schema) validation rules governing them.

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

### [CLI](#tab/cli)

Run the following command to configure the solution template and deploy the corresponding solution or application to your target. Store the configuration values in *config.yaml*.

```azurecli
az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-template-version-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$appName/versions/$appVersion" --configuration “config.yaml”   
```

> [!NOTE]
> If your template resides in a different resource group, you can use the `--solution-template-rg` argument to specify your template resource group. 

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