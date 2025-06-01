---
title: Solution with a Leaf Target 
description: Learn how to create and configure a solution with a leaf target in a four-level service group hierarchy using workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/01/2025
---

# Tutorial: Create a solution with a leaf target

In this tutorial, you will create and configure a target at the line level, which is the leaf level in a four-level service group hierarchy. You will use service groups to orchestrate workloads across different levels of the hierarchy.

For more information, see [Service groups for workload orchestration](service-group.md).

## Prerequisites


## Define the scenario

The organization has a four-level hierarchy, which is represented in the following diagram. The hierarchy consists of country, region, factory, and line levels. These levels represent a top-down structure where each level narrows the scope of orchestration. 

The sites references are created at each level of the hierarchy, being SGCountry at the country level, SGRegion at the region level, SGFactory at the factory level. The target is created at the line level, which is the lowest level in the hierarchy, so it's referred to as a leaf target. 

The solution is named EdgeLink (EL) and is deployed at the target, which means that the solution is specific to the line level. 

:::image type="content" source="./media/scenario-leaf-target.png" alt-text="Diagram of the four-level hierarchy and target at line level." lightbox="./media/scenario-leaf-target.png":::


## Create a leaf target

### [Bash](#tab/bash)

1. Create a target and set `--hierarchy-level` to `line` level. The target is named EL-71, which represents a specific line in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$resourcePrefix-el71" \
      --display-name "$resourcePrefix-el71" \
      --hierarchy-level line \
      --capabilities "$resourcePrefix-soap" \
      --description "This is EL-71 Target" \
      --solution-scope "elapp" \
      --target-specification @targetspecs.json \
      --extended-location @custom-location.json
    ```

1. Get the target ID of the created target.

    ```bash
    targetId=$(az workload-orchestration target show --resource-group "$rg" --name "$resourcePrefix-el71" --query id --output tsv)
    ```

1. Link the target ID to the factory service group.

    ```bash
    az rest \
      --method put \
      --uri "$targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{ \"properties\": { \"targetId\": \"/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory\" } }"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated. This step is optional but recommended.

    ```bash
    az workload-orchestration target update \
      --resource-group "$resourcePrefix" \
      --name "$resourcePrefix-el71"
    ```

### [PowerShell](#tab/powershell)

1. Create a target and set `--hierarchy-level` to `line` level. The target is named EL-71, which represents a specific line in the factory. Update *custom-location.json* file with the custom location of your cluster.

    ```powershell
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name "$resourcePrefix-el71" `
      --display-name "$resourcePrefix-el71" `
      --hierarchy-level line `
      --capabilities "$resourcePrefix-soap" `
      --description "This is EL-71 Target" `
      --solution-scope "elapp" `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json'
    ```

1. Get the target ID of the created target.

    ```powershell
    $targetId = $(az workload-orchestration target show --resource-group $rg --name "$resourcePrefix-el71" --query id --output tsv)
    ```

1. Link the target ID to the factory service group.

    ```powershell
    az rest `
      --method put `
      --uri $targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory'}}"
    ```

1. Update the target after connecting it to the service group to make sure the hierarchy configuration is updated. This step is optional but recommended.

    ```powershell
    az workload-orchestration target update `
      --resource-group $resourcePrefix `
      --name $resourcePrefix-el71
    ```
***

## Prepare the solution template

### [Bash](#tab/bash)

1. Create the solution schema file named edgeLink-schema.yaml.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "$resourcePrefix-ELS" --schema-file ./edgeLink-schema.yaml -l "$l"
    ```

1. Create the solution template file named edgeLink-config-template.yaml.

    ```bash
    az workload-orchestration solution-template create \
        --solution-template-name "$resourcePrefix-el" \
        -g "$rg" \
        -l "$l" \
        --capabilities "$resourcePrefix-soap" \
        --description "This is EdgeLink Solution" \
        --config-template-file ./edgeLink-config-template.yaml \
        --specification "@edgeLink-specs.json" \
        --version "1.0.0"
    ```

### [PowerShell](#tab/powershell)

1. Create the solution schema file named edgeLink-schema.yaml.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "$resourcePrefix-ELS" --schema-file .\edgeLink-schema.yaml -l $l
    ```

1. Create the solution template file named edgeLink-config-template.yaml.

    ````powershell
    az workload-orchestration solution-template create `
        --solution-template-name "$resourcePrefix-el" `
        -g $rg `
        -l $l `
        --capabilities "$resourcePrefix-soap" `
        --description "This is EdgeLink Solution" `
        --config-template-file .\edgeLink-config-template.yaml `
        --specification "@edgeLink-specs.json" `
        --version "1.0.0"
    ```
***

## Set the configuration for the solution template

### [Bash](#tab/bash)

1. Set the configuration for country service group.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$resourcePrefix-el" --target-name "$resourcePrefix-SGCountry"
    ```

1. Set the configuration for region service group.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$resourcePrefix-el" --target-name "$resourcePrefix-SGRegion"
    ```

1. Set the configuration for factory service group.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$resourcePrefix-el" --target-name "$resourcePrefix-SGFactory"
    ```

1. Set the configuration for target service group.

    ```bash
    az workload-orchestration configuration set -g "$rg" --solution-template-name "$resourcePrefix-el" --target-name "$resourcePrefix-el71"
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for country service group.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $resourcePrefix-el --target-name $resourcePrefix-SGCountry
    ```

1. Set the configuration for region service group.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $resourcePrefix-el --target-name $resourcePrefix-SGRegion
    ```

1. Set the configuration for factory service group.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $resourcePrefix-el --target-name $resourcePrefix-SGFactory
    ```

1. Set the configuration for target service group.

    ```powershell
    az workload-orchestration configuration set -g $rg --solution-template-name $resourcePrefix-el --target-name $resourcePrefix-el71
    ```
***

## Review, publish, and deploy the solution 

### [Bash](#tab/bash)

1. Review the configuration. Replace the `--solution-version` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```bash
    az workload-orchestration target review --solution-name "$resourcePrefix-el" --solution-version "1.0.0" --resource-group "$rg" --target-name "$resourcePrefix-el71"
    ```

1. Publish the configuration. Replace `<SolutionVersion>` with the value of `properties.name`, and `<ReviewID>` with the value of `properties.properties.reviewId` returned from the previous command.

    ```bash
    az workload-orchestration target publish --solution-name "$resourcePrefix-el" --solution-version <SolutionVersion> --review-id <ReviewID> --resource-group "$rg" --target-name "$resourcePrefix-el71"
    ```

1. Deploy the solution. Replace `<SolutionVersion>` with the same value you used in the previous command.

    ```bash
    az workload-orchestration target install --solution-name "$resourcePrefix-el" --solution-version <SolutionVersion> --resource-group "$rg" --target-name "$resourcePrefix-el71"
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration. Replace the `--solution-version` with the version of your solution template if you revised it, or use "1.0.0" if this is the first time you run this command.

    ```powershell
    az workload-orchestration target review --solution-name $resourcePrefix-el --solution-version "1.0.0" --resource-group $rg --target-name $resourcePrefix-el71
    ```

1. Publish the configuration. Replace `<SolutionVersion>` with the value of `properties.name`, and `<ReviewID>` the value of `properties.properties.reviewId` returned from the previous command.

    ```powershell
    az workload-orchestration target publish --solution-name $resourcePrefix-el --solution-version <SolutionVersion> --review-id <ReviewID> --resource-group $rg --target-name $resourcePrefix-el71
    ```

1. Deploy the solution. Replace `<SolutionVersion>` with the same value you used in the previous command.

    ```powershell
   az workload-orchestration target install --solution-name $resourcePrefix-el --solution-version <SolutionVersion> --resource-group $rg --target-name $resourcePrefix-el71
    ```
***

## Validate the deployment

### [Bash](#tab/bash)

1. Get the cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

    ```bash
    az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName> --overwrite-existing
    ```

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a leaf target](#create-a-leaf-target) step, which is `elapp` in this case.

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

1. Validate the configuration of the target by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create a leaf target](#create-a-leaf-target) step, which is `elapp` in this case.

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
