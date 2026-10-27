---
title: Create a Basic Solution with Common Configurations with Workload Orchestration
description: Learn how to create a basic solution with common configurations using the workload orchestration. 
author: nathmanish
ms.author: sethm
ms.topic: quickstart
ms.date: 08/27/2026
ai-usage: ai-assisted
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

This guide uses the same environment, schema, and application variables defined in [Deploy a basic solution](solution-without-common-configuration.md#define-the-variables). Set those variables first, and then define the following additional variables for the hierarchical configuration template. If you plan to use the portal, you can skip this step.

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

Follow these steps to create a [hierarchy configuration template](configuration-model.md#hierarchy-configuration-template), link it to the desired sites or targets, and set the common configuration values.

### [CLI](#tab/cli)

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

    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "siteId" --template-name "$appName" --version $appVersion
    ```

### [Portal](#tab/portal)

1. On the environment **Overview** page, under **Associate solutions**, open **Link/Associate**, and then select **Create Config template**. Alternatively, under **Configuration Templates**, select **Hierarchy Configuration Templates**, and then select **Create**.

    :::image type="content" source="./media/it-portal-create-hierarchy-configuration-menu.png" alt-text="Screenshot of the environment Overview page showing Create Config template in the Link or Associate menu." lightbox="./media/it-portal-create-hierarchy-configuration-menu.png":::

    :::image type="content" source="./media/it-portal-hierarchy-configuration-templates-create.png" alt-text="Screenshot of the Hierarchy Configuration Templates page with Create highlighted." lightbox="./media/it-portal-hierarchy-configuration-templates-create.png":::

1. On the **Basics** tab, select the **Subscription**, **Resource group**, and **Region** for the hierarchy configuration template.
1. Enter the **Hierarchy configuration name**, semantic **Version**, and optional **Description**.

    :::image type="content" source="./media/it-portal-hierarchy-configuration-basics.png" alt-text="Screenshot of the Create Hierarchy Configuration Basics tab showing project details, instance details, and the hierarchy configuration code editor." lightbox="./media/it-portal-hierarchy-configuration-basics.png":::

1. Under **Hierarchy configuration code**, upload a YAML or JSON configuration file or enter the configuration code in the editor. To view a sample configuration, select **Download sample file**.

    :::image type="content" source="./media/it-portal-hierarchy-configuration-upload-file.png" alt-text="Screenshot of the Create Hierarchy Configuration Basics tab with Upload YAML or JSON file highlighted." lightbox="./media/it-portal-hierarchy-configuration-upload-file.png":::

1. After the portal loads the hierarchy configuration code, select **Next: Link to Hierarchy**.

    :::image type="content" source="./media/it-portal-hierarchy-configuration-code.png" alt-text="Screenshot of the Create Hierarchy Configuration Basics tab showing configuration code and the Next Link to Hierarchy button." lightbox="./media/it-portal-hierarchy-configuration-code.png":::

1. On the **Link to Hierarchy** tab, select the sites or targets where you want to apply the configuration template. This step is optional. You can link the template after creating it.
1. Select **Next: Review + Create**.

    :::image type="content" source="./media/it-portal-hierarchy-configuration-link-targets.png" alt-text="Screenshot of the Link to Hierarchy tab showing selected targets and the Next Review and Create button." lightbox="./media/it-portal-hierarchy-configuration-link-targets.png":::

1. After validation passes, review the project details, instance details, hierarchy configuration code, and linked hierarchy resources.
1. Select **Review+Create** to create the hierarchy configuration template.

    :::image type="content" source="./media/it-portal-hierarchy-configuration-review-create.png" alt-text="Screenshot of the hierarchy configuration Review and Create tab showing successful validation and the Review and Create button." lightbox="./media/it-portal-hierarchy-configuration-review-create.png":::

1. Sign in to the [workload orchestration portal](https://portal.digitaloperations.configmanager.azure.com/#/browse/overview), and select **Configure Hierarchy** on the left.

1. Select the site you want to configure whose configuration status isn't **Configuration up to date**, and then select **Configure**.

    :::image type="content" source="./media/configure-line-1.png" alt-text="Screenshot of the Configure Hierarchy page showing a selected Site and the Configure button." lightbox="./media/configure-line-1.png":::

1. If a single configuration template is linked to the hierarchy resource, enter its configuration values. If multiple templates are linked, select the templates to configure on the **Select Version** screen, and then select **Next**. You can filter templates by name, version, and status.

    :::image type="content" source="./media/configure-line-2.png" alt-text="Screenshot of the Select Version page showing configuration templates selected for a Site." lightbox="./media/configure-line-2.png":::

1. In the **Configure** step, enter the parameters for each template, and then select **Next**. If you previously configured another version of the same template, you can autofill values from that version.

    :::image type="content" source="./media/configure-line-3.png" alt-text="Screenshot of the Configure step showing parameter values for hierarchy configuration templates." lightbox="./media/configure-line-3.png":::

1. Review the configuration details, and then select **Confirm** to apply the changes.

    :::image type="content" source="./media/configure-line-4.png" alt-text="Screenshot of the Review step showing hierarchy configuration values and the Confirm button." lightbox="./media/configure-line-4.png":::

***

> [!NOTE]
> You can also link a configuration template to a specific target instead of a Site, by specifying the target ID as value for `--hierarchy-ids`. 


## Create the solution template 

Follow these steps to create a [solution template](configuration-model.md#solution-template) for your application.

### [CLI](#tab/cli)

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

### [Portal](#tab/portal)

1. Open your workload orchestration environment in the Azure portal.

    :::image type="content" source="./media/it-portal-environments-overview.png" alt-text="Screenshot of the environment grid page." lightbox="./media/it-portal-environments-overview.png":::

1. On **Overview**, under **Associate solutions**, open **Link/Associate**, and then select **Create Solution template**.

    :::image type="content" source="./media/it-portal-create-solution-template-menu.png" alt-text="Screenshot of the environment Overview page showing Create Solution template in the Link or Associate menu." lightbox="./media/it-portal-create-solution-template-menu.png":::

1. On the **Basics** tab, select the **Subscription**, **Resource group**, and **Region** for the solution template.
1. Enter the **Solution name**, semantic **Version**, and optional **Description**.

    :::image type="content" source="./media/it-portal-solution-template-basics.png" alt-text="Screenshot of the Create solution template Basics tab showing project and instance details." lightbox="./media/it-portal-solution-template-basics.png":::

1. Under **Solution template code**, upload the YAML template or enter the template code in the editor. To view the sample template code, select **Download sample file**.

    :::image type="content" source="./media/it-portal-solution-template-upload-file.png" alt-text="Screenshot of the Create solution template Basics tab with Upload YAML or JSON file highlighted." lightbox="./media/it-portal-solution-template-upload-file.png":::

1. After the portal loads the configuration template, select **Next: Add component**.

    :::image type="content" source="./media/it-portal-solution-template-configuration-code.png" alt-text="Screenshot of the Create solution template Basics tab showing configuration template code and the Next Add component button." lightbox="./media/it-portal-solution-template-configuration-code.png":::

1. On the **Add component** tab, upload a JSON component specification or enter the specification in the editor. Use the sample file as a reference.

    :::image type="content" source="./media/it-portal-solution-template-add-component.png" alt-text="Screenshot of the empty Add component tab in the Create solution template wizard." lightbox="./media/it-portal-solution-template-add-component.png":::

1. Select **Next: Capability tags**.

    :::image type="content" source="./media/it-portal-solution-template-helm-component.png" alt-text="Screenshot of the Add component tab showing a Helm component specification and the Next Capability tags button." lightbox="./media/it-portal-solution-template-helm-component.png":::

1. On the **Capability tags** tab, select the tags that map this solution template to compatible targets.
1. Select **Next: Review + Create**.

    :::image type="content" source="./media/it-portal-solution-template-capability-tags.png" alt-text="Screenshot of the Capability tags tab showing an added capability tag and the Next Review and Create button." lightbox="./media/it-portal-solution-template-capability-tags.png":::

1. After validation passes, select **Review+Create** to create the solution template.

    :::image type="content" source="./media/it-portal-solution-template-review-create.png" alt-text="Screenshot of the solution template Review and Create tab showing successful validation and the Review and Create button." lightbox="./media/it-portal-solution-template-review-create.png":::

***

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