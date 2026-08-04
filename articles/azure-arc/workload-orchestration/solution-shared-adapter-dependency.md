---
title: Create a Solution with Shared Adapter Dependency with Workload Orchestration
description: Learn how to create a solution with shared adapter dependency using workload orchestration.
author: sethmanheim
ms.author: sethm
ms.topic: tutorial
ms.date: 05/07/2025
ms.custom:
  - build-2025
# Customer intent: "As a cloud architect, I want to create a solution that utilizes shared adapter dependencies through CLI, so that I can effectively manage data synchronization across different application targets."
---

# Tutorial: Create a solution with shared adapter dependency

In this tutorial, you use workload orchestration to create a Factory Sensor Anomaly Detector (FSAD) solution which is dependent on a Shared Sync Adapter (SSA) solution. A shared sync adapter is a component used in various solutions to manage data synchronization between devices and servers.

The FSAD solution is deployed on a child target, while the SSA solution is deployed on a parent target. The FSAD solution uses the SSA solution to synchronize data between devices and servers.

## Prerequisites

- Set up the required resources for workload orchestration by referring to [Set up workload orchestration](set-up-workload-orchestration.md).
- Download the artifacts from the [workload-orchestration GitHub repository](https://github.com/Azure/workload-orchestration). 

    [![Download](https://img.shields.io/badge/Download%20zip%20file-0078D4?style=flat&labelColor=0078D4)](https://github.com/Azure/workload-orchestration/archive/refs/heads/main.zip) 

> [!NOTE]
> You can reuse the global variables defined in [Set up workload orchestration](set-up-workload-orchestration.md).


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

The configuration templates for shared applications follow the same rules as mentioned in [configuration model](configuration-model.md), with slight additions.

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
  DependentAppConfigs: []   # It should be an array or null

```

`AppConfig` of the FSAD configuration template changes at every target, and the target-specific `AppConfig` gets injected into `dependentAppConfigs` of the SSA configuration template.


## Define the variables for solution templating

Create the template and schema YAML files by referring to *shared-schema.yaml* and *app-config-template.yaml* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/).

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

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/).
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

1. Create *targetspecs.json* file by referring to the *targetspecs.json* file in the [GitHub repository](https://github.com/Azure/workload-orchestration/).
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

## Create a Shared Sync Adapter (SSA) solution template

1. Create a *specs.json* file by referring to *specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/).
1. In your *specs.json* file, update the Helm URL, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
Update the *app-config-template.yaml* file with proper reference to your schema which you created in the previous step.
1. Create the SSA solution template using the following command. The following command takes version input from CLI argument:

    ```azurecli
    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName1" --description "$desc" --capabilities "$appCapList1" --config-template-file "$appConfig" --specification "@specs.json" --version "$appVersion"
    ```

## Create a target at child level for FSAD

Create a target at child level. Ensure *custom-location.json* is updated with the created custom location's ID.

```azurecli	
az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description "$childDesc" --solution-scope "$childName-scope" --target-specification '@targetspecs.json' --extended-location '@custom-location.json' --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextChildName"
```

> [!NOTE]
> Update the parameter `solutionTemplateId` under dependencies section of FSAD configuration template and schema with the SSA solution template ID.
> The `solution-template create` command displays the ID along with solution template version.
>
> ```bash
> solutionTemplateId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutiontemplates/$appName1"
> ```


## Create a FSAD solution template

1. Create a *fsad-specs.json* file by referring to *fsad-specs.json* in the compressed folder from the [GitHub repository](https://github.com/Azure/workload-orchestration/).
1. In your *fsad-specs.json* file, update the Helm URL, for example, *contosocm.azurecr.io/helm/app*, and chart version in x.x.x format, for example, *0.5.0*.
1. Create the FSAD solution template using the following command:

    ```azurecli
    # Any modifications to solution files will necessitate version update.

    az workload-orchestration solution-template create --resource-group "$rg" --location "$l" --solution-template-name "$appName2" --description "$desc2" --capabilities "$appCapList2" --config-template-file "$appConfig2" --specification "@fsad-specs.json" --version "$app2Version"
    ```

## Configuration solution parameters

### [CLI](#tab/cli)

1. Edit parameters of FSAD at child level.

    ```azurecli
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName" --template-name "$appName2" --version $appVersion --solution
    ```

1. Run `target review` command for SSA with dependencies.

    ```azurecli
    # Fetch the appVersion from SSA solution create command
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName1" --solution-version "$appVersion" --target-name "$parentName"
    ```

1. Update the solution version ID, `Id`, from the output of previous command in the *dependencies.json* file. 
1. Run `target review` command for FSAD with dependencies. Ensure that the FSAD solution version has dependencies listed in the CLI output. 

    ```azurecli
    # Fetch the appVersion from FSAD solution create command
    az workload-orchestration target review --resource-group "$rg" --solution-name "$appName2" --solution-version "$app2Version" --target-name "$childName" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `solutionTemplateVersionId` from `solutionDependencies` in the output. Execute the `az rest` command and ensure FSAD configuration is injected.

    ```azurecli
    az rest --uri "<solutionTemplateVersionId>?api-version=2025-01-01-preview" --method GET
    ```

1. In the CLI response, check `reviewId` and `name` values. The name displays the new solution template version.

### [Portal](#tab/portal)

The configuration process is similar to the one described in [Deploy a basic solution](solution-without-common-configuration.md#deploy-the-solution), but with some additional steps.

1. Select the **FSAD solution** you want to configure and click on **Configure and publish**.
1. The new details pane shows the configuration values for the selected solution.
1. In the **Select targets to publish solution** step, auto-publish option is enabled by default which means the values will be applied for all targeted lines. You can proceed with auto-publish or choose the targets where the FSAD solution needs to be deployed. Click on **Next**.

    :::image type="content" source="./media/configure-fsad-1.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure the targets for a solution FSAD." lightbox="./media/configure-fsad-1.png":::

1. In the **Configure target** step, enter the instance name for the solution and the values for FSAD configurations.

    :::image type="content" source="./media/configure-fsad-2.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to enter the values to configure the targets for a solution FSAD." lightbox="./media/configure-fsad-2.png":::

1. In the **Shared dependencies** step, you can see the details of the dependent SSA instance. Under the **Instance** field, either choose an existing one or create a new SSA instance. 

    :::image type="content" source="./media/configure-fsad-3.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to configure the shared dependencies for a solution FSAD." lightbox="./media/configure-fsad-3.png":::

    1. To create a new instance of SSA, **enter** the new instance name and the configuration values for SSA. Click on **Next**.
    
        :::image type="content" source="./media/configure-fsad-4.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to create a new SSA instance." lightbox="./media/configure-fsad-4.png":::
    
    1. Review the SSA details and click on **Configure + publish**.
    
        :::image type="content" source="./media/configure-fsad-5.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review a new SSA instance and publish it." lightbox="./media/configure-fsad-5.png":::

1. Once the dependent solution details are filled in, click **Next**.

    :::image type="content" source="./media/configure-fsad-6.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review the details of the shared dependencies for a solution FSAD." lightbox="./media/configure-fsad-6.png":::

1. In the **Review** step, review the FSAD configuration details, status and click on **Publish** to create a new revision of configuration values for the selected targets.

    :::image type="content" source="./media/configure-fsad-7.png" alt-text="Screenshot of the solution tab in workload orchestration portal showing how to review and publish a FSAD solution." lightbox="./media/configure-fsad-7.png":::

***

## Deploy the solution

### [CLI](#tab/cli)

1. Run `target publish` to publish FSAD solution with dependencies. The command publishes the FSAD solution with dependencies to SSA. Enter the `reviewId` from the previous command response for `--solution-version-id`.

    ```azurecli
    az workload-orchestration target publish --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version 
    ```

1. Run `target install` command to deploy the solution.

    ```azurecli
    az workload-orchestration target install --resource-group "$rg" --target-name "$childName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$childName/solutions/$appName2/versions/$app2Version
    ```

### [Portal](#tab/portal)

The portal deployment process is similar to the one described in [Deploy a basic solution](solution-without-common-configuration.md#deploy-the-solution).

***