---
title: Solution with a Non-Leaf Target
description: Learn how to create and configure a solution with a non-leaf target in a four-level service group hierarchy.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/01/2025
---

# Tutorial: Create a solution with a non-leaf target using service groups

In this tutorial, you create and configure a target at the region level, which is the third level in a four-level service group hierarchy. You use service groups to orchestrate workloads across different levels of the hierarchy.

For more information, see [Service groups at different hierarchy levels in workload orchestration](service-group.md#service-groups-at-different-hierarchy-levels).

[!INCLUDE [service-groups-note](includes/service-groups-note.md)]

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/).
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 
- Create the service groups and hierarchy levels. If you haven't, follow the steps in [Service groups at different hierarchy levels](service-group.md#service-groups-at-different-hierarchy-levels).


> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).

## Define the scenario

The organization has a four-level hierarchy, which is represented in the following diagram. The hierarchy consists of country, region, factory, and line levels. These levels represent a top-down structure where each level narrows the scope of orchestration. 

:::image type="content" source="./media/scenario-non-leaf-target.png" alt-text="Diagram of the four-level hierarchy and target at region level." lightbox="./media/scenario-non-leaf-target.png":::

The sites references are created at only at country and region level, being SGCountry at the country level, SGRegion at the region level. The target is created at the region level, which isn't the lowest level in the hierarchy, so it's referred to as a non-leaf target.

The solution is named RegionHub (RH) and is deployed at the target, which means that the solution is specific to the regional level. 

## Create a non-leaf target

### [Bash](#tab/bash)

1. Create a target and set `--hierarchy-level` to `region` level. The target is named Region01, which represents a specific region in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    RegionName="Region01"

    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$RegionName" \
      --display-name "$RegionName" \
      --hierarchy-level "region" \
      --capabilities "Use for soap soap" \
      --description "This is Region01 Target" \
      --solution-scope "new" \
      --target-specification @targetspecs.json \
      --extended-location @custom-location.json \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

1. Get the target ID of the created target.

    ```bash
    targetId=$(az workload-orchestration target show --resource-group $rg --name "$RegionName" --query id --output tsv)
    ```

1. Link the target ID to the region service group. Make sure to replace `$level2Name` with the name of your region service group.

    ```bash
    az rest \
      --method put \
      --uri "$targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{ \"properties\": { \"targetId\": \"/providers/Microsoft.Management/serviceGroups/$level2Name\" } }"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated. This step is optional but recommended.

    ```bash
    az workload-orchestration target update --resource-group "$rg" --name "$RegionName"
    ```

### [PowerShell](#tab/powershell)

1. Create a target and set `--hierarchy-level` to `region` level. The target is named Region01, which represents a specific line in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```powershell
    $RegionName = "Region01"

    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $RegionName`
      --display-name $RegionName `
      --hierarchy-level "region" `
      --capabilities "Use for soap production" `
      --description "This is Region01 Target" `
      --solution-scope "new" `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

1. Get the target ID of the created target.

    ```powershell
    $targetId = $(az workload-orchestration target show --resource-group $rg --name "$RegionName" --query id --output tsv)
    ```

1. Link the target ID to the region service group. Make sure to replace `$level2Name` with the name of your region service group.

    ```powershell
    az rest `
      --method put `
      --uri $targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level2Name' }}"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated. This step is optional but recommended.

    ```powershell
    az workload-orchestration target update --resource-group $rg --name $RegionName
    ```
***

## Prepare the solution template

To create the solution schema and solution template files, you can use *common-schema.yaml* and *app-config-template.yaml* files, respectively, in [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) as reference. 

> [!IMPORTANT]
> If the solution is to be deployed at factory level, then the `editable_at` field in the schema only accepts the parent levels in addition to target level, that is country, region, and factory. If the solution is to be deployed at region level, then the `editable_at` field in the schema accepts only country and region levels.

### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "$RegionName-schema" --schema-file ./regionHub-schema.yaml -l "$l"
    ```

1. Create the solution template file. 

    ```bash
    solutionName="RegionHub Solution"    

    az workload-orchestration solution-template create \
        --solution-template-name "$solutionName" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is RegionHub Solution" \
        --config-template-file ./regionHub-config-template.yaml \
        --specification "@specs.json" \
        --version "1.0.0"
    ```

### [PowerShell](#tab/powershell)

1. Create the solution schema file. 

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "$RegionName-schema" --schema-file ./regionHub-schema.yaml -l $l
    ```

1. Create the solution template file. 

    ```powershell
    $solutionName = "RegionHub Solution"    

    az workload-orchestration solution-template create `
        --solution-template-name $solutionName `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is RegionHub Solution" `
        --config-template-file ./regionHub-config-template.yaml `
        --specification '@specs.json' `
        --version "1.0.0"
    ```
***

## Set the configuration for the solution template

### [Bash](#tab/bash)

1. Set the configuration for country service group.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$solutionName" --target-name "$level1Name"
    ```

1. Set the configuration for region service group.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$solutionName" --target-name "$level2Name"
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for country service group.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $solutionName --target-name $level1Name
    ```

1. Set the configuration for region service group.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $solutionName --target-name $level2Name
    ```
***

## Review, publish, and deploy the solution 

### [Bash](#tab/bash)

1. Review the configuration. Replace the `<solution-version>` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```bash
    solutionVersion="<solution-version>"
    subId="<subscription-id>"

    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName/versions/1.0.0 --resource-group "$rg" --target-name "$RegionName"
    ```

1. Publish the configuration. Replace `<SolutionVersion>` with the value of `properties.name`, and `<ReviewID>` with the value of `properties.properties.reviewId` returned from the previous command.

    ```bash
    solutionVersion="<SolutionVersion>" 

    az workload-orchestration target publish --resource-group "$rg" --target-name "$RegionName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$RegionName/solutions/$solutionName/versions/$solutionVersion
    ```

1. Deploy the solution. Replace `<SolutionVersion>` with the same value you used in the previous command.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$RegionName" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$RegionName/solutions/$solutionName/versions/$solutionVersion
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration. Replace the `<solution-version>` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```powershell
    $solutionVersion = "<solution-version>"
    $subId = "<subscription-id>"
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName/versions/1.0.0--resource-group $rg --target-name $RegionName
    ```

1. Publish the configuration. 

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $RegionName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$RegionName/solutions/$solutionName/versions/$solutionVersion
    ```

1. Deploy the solution. 

    ```powershell
   az workload-orchestration target install --resource-group $rg --target-name $RegionName --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$RegionName/solutions/$solutionName/versions/$solutionVersion
    ```
***

## Validate the deployment

### [Bash](#tab/bash)

1. Get the cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

    ```bash
    az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName> --overwrite-existing
    ```

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a non-leaf target](#create-a-non-leaf-target) step, which is `new` in this case.

    ```bash
    # There may be more releases in the same namespace if you have used the same custom location and namespace name before.
    # Look for your respective solution/instance.
    namespace="<Namespace>"
    for release in $(helm list -n "$namespace" -q); do
        echo "=================="
        echo "For release/instance: $release"
        helm get values "$release" -n "$namespace"
        echo "=================="
    done
    ```

### [PowerShell](#tab/powershell)

1. Get the cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

    ```powershell
    az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName> --overwrite-existing
    ```

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a non-leaf target](#create-a-non-leaf-target) step, which is `new` in this case.

    ```powershell
    # There may be more releases in same namespace if you have used same custom location and namespace name before
    # look for your respective solution/instance
    $namespace = "<Namespace>"
    helm list -n $namespace -q | ForEach-Object {
        Write-Host "=================="
        Write-Host "For release/instance: $_ "
        helm get values $_ -n $namespace
        Write-Host "=================="
    }
    ```
***

You can also go to the [Azure portal](https://portal.azure.com) and check pods in the namespace of your AKS cluster to validate the deployment. 