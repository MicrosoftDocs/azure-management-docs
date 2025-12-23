---
title: Solution with a Leaf Target 
description: Learn how to create and configure a solution with a leaf target in a four-level service group hierarchy using workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/02/2025
---

# Tutorial: Create a solution with a leaf target using service groups

In this tutorial, you create and configure a target at the line level, which is the leaf level in a four-level service group hierarchy. You use service groups to orchestrate workloads across different levels of the hierarchy.

For more information, see [Service groups at different hierarchy levels in workload orchestration](service-group.md#service-groups-at-different-hierarchy-levels).

[!INCLUDE [service-groups-note](includes/service-groups-note.md)]

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 
- Create the service groups and hierarchy levels. If you haven't, follow the steps in [Service groups at different hierarchy levels](service-group.md#service-groups-at-different-hierarchy-levels).

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).

## Define the scenario

The organization has a four-level hierarchy, which is represented in the following diagram. The hierarchy consists of region, city, factory, and line levels. These levels represent a top-down structure where each level narrows the scope of orchestration. 

:::image type="content" source="./media/scenario-leaf-target.png" alt-text="Diagram of the four-level hierarchy and target at line level." lightbox="./media/scenario-leaf-target.png":::

The sites references are created at each level of the hierarchy, being SGRegion at the region level, SGCity at the city level, SGFactory at the factory level. The target is created at the line level, which is the lowest level in the hierarchy, so it's referred to as a leaf target. 

The solution is named EdgeLink (EL) and is deployed at the target, which means that the solution is specific to the line level. 

## Create a leaf target

### [Bash](#tab/bash)

1. Create a target and set `--hierarchy-level` to `line` level. The target is named Line01, which represents a specific line in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    Linename="Line01"

    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$Linename" \
      --display-name "$Linename" \
      --hierarchy-level "line" \
      --capabilities "soap" \
      --description "Use for soap production" \
      --solution-scope "new" \
      --target-specification '@targetspecs.json' \
      --extended-location '@custom-location.json' \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

1. Get the target ID of the created target.

    ```bash
    targetId=$(az workload-orchestration target show --resource-group "$rg" --name "$Linename" --query id --output tsv)
    ```

1. Link the target ID to the factory service group. Make sure to replace `$level3Name` with the name of your factory service group.

    ```bash
    az rest \
      --method put \
      --uri "$targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{ \"properties\": { \"targetId\": \"/providers/Microsoft.Management/serviceGroups/$level3Name\" } }"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated.

    ```bash
    az workload-orchestration target update --resource-group "$rg" --name "$Linename"
    ```

### [PowerShell](#tab/powershell)

1. Create a target and set `--hierarchy-level` to `line` level. The target is named Line01, which represents a specific line in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```powershell
    $Linename = "Line01"

    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $Linename `
      --display-name $Linename `
      --hierarchy-level "line" `
      --capabilities "soap" `
      --description "Use for soap production" `
      --solution-scope "new" `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

1. Get the target ID of the created target.

    ```powershell
    $targetId = $(az workload-orchestration target show --resource-group $rg --name "$Linename" --query id --output tsv)
    ```

1. Link the target ID to the factory service group. Make sure to replace `$level3Name` with the name of your factory service group.

    ```powershell
    az rest `
      --method put `
      --uri $targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level3Name'}}"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated.

    ```powershell
    az workload-orchestration target update --resource-group $rg --name $Linename
    ```
***

## Prepare the solution template

To create the solution schema, configuration templates and solution template, you can use the sample files provided in **Service Groups/Scenario1_LeafTarget** within [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) as reference. 


### [Bash](#tab/bash)

1. Create the solution schema file. 

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-file ./edgeLink-schema.yaml -l "$l"
    ```

1. Create the region config template file and link it to **region** hierarchy level

    ```bash
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-region-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $regionSiteId --context-id $contextId
    ```

1. Create the city config template file and link it to **city** hierarchy level

    ```bash
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-city-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $citySiteId --context-id $contextId

1. Create the factory config template file and link it to **factory** hierarchy level

    ```bash
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-factory-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $factorySiteId --context-id $contextId

1. Create the solution template file. 

    ```bash
    az workload-orchestration solution-template create \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is EdgeLink Solution" \
        --configuration-template-file ./edgeLink-config-template.yaml \
        --specification "@edgeLink-specs.json" \
        --version "1.0.0"
    ```

### [PowerShell](#tab/powershell)

1. Create the solution schema file. 

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "$Linename-schema" --schema-file .\edgeLink-schema.yaml -l $l
    ```

1. Create the region config template file and link it to **region** hierarchy level

    ```powershell
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-region-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $regionSiteId --context-id $contextId
    ```

1. Create the city config template file and link it to **city** hierarchy level

    ```powershell
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-city-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $citySiteId --context-id $contextId

1. Create the factory config template file and link it to **factory** hierarchy level

    ```powershell
    az workload-orchestration config-template create -g "$rg" --location "$l" --configuration-template-file ./edgeLink-factory-config-template.yaml --description "<description>"
    az workload-orchestration config-template link -g "$rg" -n "$configName" --hierarchy-ids $factorySiteId --context-id $contextId

1. Create the solution template file. 

    ```powershell
    az workload-orchestration solution-template create `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is EdgeLink Solution" `
        --configuration-template-file .\edgeLink-config-template.yaml `
        --specification "@edgeLink-specs.json" `
        --version "1.0.0"
    ```
***

## Set the configuration for each template

### [Bash](#tab/bash)

1. Set the configuration for region site.

    ```bash
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "regionSiteId" --template-name "EdgeLinkRegion" --version "1.0.0"
    ```

1. Set the configuration for city site.

    ```bash
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "citySiteId" --template-name "EdgeLinkCity" --version "1.0.0"
    ```

1. Set the configuration for factory site.

    ```bash
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "citySiteId" --template-name "EdgeLinkCity" --version "1.0.0"
    ```

1. Set the configuration for target at line (leaf) level.

    ```bash
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$Linename" --template-name "EdgeLinkApp" --version 1.0.0 --solution
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for region site.

    ```powershell
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "regionSiteId" --template-name "EdgeLinkRegion" --version "1.0.0"
    ```

1. Set the configuration for city site.

    ```powershell
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "citySiteId" --template-name "EdgeLinkCity" --version "1.0.0"
    ```

1. Set the configuration for factory site.

    ```powershell
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "citySiteId" --template-name "EdgeLinkCity" --version "1.0.0"
    ```

1. Set the configuration for target at line (leaf) level.

    ```powershell
    az workload-orchestration configuration set --template-rg "$rg" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$Linename" --template-name "EdgeLinkApp" --version 1.0.0 --solution
    ```
***

## Review, publish, and deploy the solution 

### [Bash](#tab/bash)

1. Review the configuration. Replace the `<solution-version>` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```bash
    az workload-orchestration target review --resource-group "$rg" --target-name "$Linename" --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName/versions/$solutionVersion
    ```

1. Publish the configuration.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$Linename" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$LineName/solutions/$solutionName/versions/$solutionVersion
    ```

1. Deploy the solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$Linename" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineName/solutions/$solutionName/versions/$solutionVersion
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration. Replace the `<solution-version>` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```powershell
    $solutionVersion = "<solution-version>"
    $subId = "<subscription-id>"

    az workload-orchestration target review --resource-group $rg --target-name $Linename --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$solutionName/versions/$solutionVersion 
    ```

1. Publish the configuration. 

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $Linename --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$Linename/solutions/$solutionName/versions/$solutionVersion
    ```

1. Deploy the solution. 

    ```powershell
   az workload-orchestration target install --resource-group $rg --target-name $Linename --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$Linename/solutions/$solutionName/versions/$solutionVersion
    ```
***

## Validate the deployment

### [Bash](#tab/bash)

1. Get the cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

    ```bash
    az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName> --overwrite-existing
    ```

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a leaf target](#create-a-leaf-target) step, which is `new` in this case.

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

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a leaf target](#create-a-leaf-target) step, which is `new` in this case.

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
