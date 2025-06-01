---
title: Solution with Multiple Shared Dependencies at Different Hierarchy Levels
description: Learn how to create a solution with multiple shared dependencies at different hierarchy levels in Azure Arc-enabled Kubernetes.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/01/2025
---

# Tutorial: Create a solution with multiple shared dependencies at different levels

In this tutorial, you will create a solution with multiple shared dependencies at different levels in a four-level hierarchy organization. You will use service groups to orchestrate workloads across different levels of the hierarchy.

For more information, see [Service groups for workload orchestration](service-group.md).

## Prerequisites

## Define the scenario

The organization has a four-level hierarchy, which is represented in the following diagram. The hierarchy consists of country, region, factory, and line levels. These levels represent a top-down structure where each level narrows the scope of orchestration. 

The sites references are created at each level of the hierarchy, being SGCountry at the country level, SGRegion at the region level, SGFactory at the factory level. A target is created at each level, being CountryTarget at the country level, RegionTarget at the region level, FactoryTarget at the factory level, and LineTarget at the line level.

A solution is deployed at each target as follows:

- Country Adapter (CA) is a solution at country level.
- Region Adapter (RA) is a solution at region level.
- Shared Sync Adapter (SSA) is a solution at factory level.
- Factory Sensor Anomaly Detector (FSAD) is a solution at line level. FSAD is dependent on SSA, RA, and CA, which means that during the deployment, FSAD configuration will be updated with the information from the other solutions.

All the instances of CA, RA, SSA, and FSAD are deployed in the same Azure Arc-enabled Kubernetes cluster.

:::image type="content" source="./media/scenario-single-solution-dependencies.png" alt-text="Diagram of the four-level hierarchy and target at each level, where line level solution is dependent on the other solution." lightbox="./media/scenario-single-solution-dependencies.png":::

## Create targets

### [Bash](#tab/bash)

1. Create a target for each hierarchy level. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    solution_scope="one-to-n-app"  # if you want to change the name, make sure it follows the K8s object naming convention
    countryTarget="${resourcePrefix}-country"
    regionTarget="${resourcePrefix}-region"
    factoryTarget="${resourcePrefix}-factory"
    mk80Target="${resourcePrefix}-mk80"

    # Create target at country level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$countryTarget" \
      --display-name "$countryTarget" \
      --hierarchy-level country \
      --capabilities "${resourcePrefix}-soap" \
      --description "This is Country Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json"

    # Create target at region level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$regionTarget" \
      --display-name "$regionTarget" \
      --hierarchy-level region \
      --capabilities "${resourcePrefix}-soap" \
      --description "This is Region Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json"

    # Create target at factory level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$factoryTarget" \
      --display-name "$factoryTarget" \
      --hierarchy-level factory \
      --capabilities "${resourcePrefix}-soap" \
      --description "This is Factory Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json"

    # Create target at line level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$mk80Target" \
      --display-name "$mk80Target" \
      --hierarchy-level line \
      --capabilities "${resourcePrefix}-soap" \
      --description "This is MK-80 Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json"
    ```

1. Get the target IDs of the created targets.

    ```bash
    mk80TargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$mk80Target" --query id --output tsv)
    factoryTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$factoryTarget" --query id --output tsv)
    regionTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$regionTarget" --query id --output tsv)
    countryTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$countryTarget" --query id --output tsv)
    ```

1. Link the target IDs to their respective service groups.

    ```bash
    # Link to country service group
    az rest \
      --method put \
      --uri "${countryTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/${resourcePrefix}-SGCountry'}}"

    # Link to region service group
    az rest \
      --method put \
      --uri "${regionTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/${resourcePrefix}-SGRegion'}}"

    # Link to factory service group
    az rest \
      --method put \
      --uri "${factoryTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/${resourcePrefix}-SGFactory'}}"

    # Link to line service group
    az rest \
      --method put \
      --uri "${mk80TargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/${resourcePrefix}-SGFactory'}}"
    ```

1. Update the targets after connecting them to the service groups to make sure the hierarchy configurations are updated. This step is optional but recommended.

    ```bash
    # Update country target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$countryTarget"

    # Update region target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$regionTarget"

    # Update factory target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$factoryTarget"

    # Update line target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$mk80Target"
    ```

### [PowerShell](#tab/powershell)

1. Create a target for each hierarchy level. Update *custom-location.json* file with the custom location of your cluster.

    ```powershell
    $solution_scope = "one-to-n-app"  # if you want to change the name, make sure it follows the K8s object naming convention
    $countryTarget = "$resourcePrefix-country"
    $regionTarget = "$resourcePrefix-region"
    $factoryTarget = "$resourcePrefix-factory"
    $mk80Target = "$resourcePrefix-mk80"
    
    # Create target at country level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name "$countryTarget" `
      --display-name "$countryTarget" `
      --hierarchy-level country `
      --capabilities "$resourcePrefix-soap" `
      --description "This is Country Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json'
    
    # Create target at region level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name "$regionTarget" `
      --display-name "$regionTarget" `
      --hierarchy-level region `
      --capabilities "$resourcePrefix-soap" `
      --description "This is Region Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json'
    
    # Create target at factory level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name "$factoryTarget" `
      --display-name "$factoryTarget" `
      --hierarchy-level factory `
      --capabilities "$resourcePrefix-soap" `
      --description "This is Factory Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json'
    
    # Create target at line level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name "$mk80Target" `
      --display-name "$mk80Target" `
      --hierarchy-level line `
      --capabilities "$resourcePrefix-soap" `
      --description "This is MK-80 Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json'
    ```

1. Get the target IDs of the created targets.

    ```powershell
    $mk80TargetId = $(az workload-orchestration target show --resource-group $rg --name "$mk80Target" --query id --output tsv)
    $factoryTargetId = $(az workload-orchestration target show --resource-group $rg --name "$factoryTarget" --query id --output tsv)
    $regionTargetId = $(az workload-orchestration target show --resource-group $rg --name "$regionTarget" --query id --output tsv)
    $countryTargetId = $(az workload-orchestration target show --resource-group $rg --name "$countryTarget" --query id --output tsv)
    ```

1. Link the target IDs to their respective service groups. 

    ```powershell
    #Link to country service group
    az rest `
      --method put `
      --uri $countryTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCountry'}}"
    
    #Link to region service group
    az rest `
      --method put `
      --uri $regionTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion'}}"
    
    #Link to factory service group
    az rest `
      --method put `
      --uri $factoryTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory'}}"
    
    #Link to line service group
    az rest `
      --method put `
      --uri $mk80TargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory'}}"
    ```

1. Update the targets after connecting them to the service groups to make sure the hierarchy configurations are updated. This step is optional but recommended.

    ```powershell
    #Update country target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $countryTarget 
    
    #Update region target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $regionTarget 
    
    #Update factory target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $factoryTarget 
    
    #Update line target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $mk80Target
    ```
***

## Prepare the solution templates

### Solution template for CA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ca-schema" --schema-file ./ca-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    caversion="1.0.0"
    caname="${resourcePrefix}-ca"
    az workload-orchestration solution-template create \
        --solution-template-name "$caname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "${resourcePrefix}-soap" \
        --description "This is CA Solution" \
        --configuration-template-file ./ca-config-template.yaml \
        --specification "@ca-specs.json" \
        --version "$caversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "ca-schema" --schema-file .\ca-schema.yaml -l $l
    ```

1. Create the solution template file

    ````powershell
    $caversion = "1.0.0"
    $caname = "$resourcePrefix-ca" 
    az workload-orchestration solution-template create `
        --solution-template-name "$caname" `
        -g $rg `
        -l $l `
        --capabilities "$resourcePrefix-soap" `
        --description "This is CA Solution" `
        --configuration-template-file .\ca-config-template.yaml `
        --specification "@ca-specs.json" `
        --version $caversion
    ```
***

### Solution template for RA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ra-schema" --schema-file ./ra-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    raversion="1.0.0"
    raname="${resourcePrefix}-ra"
    az workload-orchestration solution-template create \
        --solution-template-name "$raname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "${resourcePrefix}-soap" \
        --description "This is RA Solution" \
        --configuration-template-file ./ra-config-template.yaml \
        --specification "@ra-specs.json" \
        --version "$raversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "ra-schema" --schema-file .\ra-schema.yaml -l $l
    ```

1. Create the solution template file

    ````powershell
    $raversion = "1.0.0"
    $raname = "$resourcePrefix-ra" 
    az workload-orchestration solution-template create `
        --solution-template-name "$raname" `
        -g $rg `
        -l $l `
        --capabilities "$resourcePrefix-soap" `
        --description "This is RA Solution" `
        --configuration-template-file .\ra-config-template.yaml `
        --specification "@ra-specs.json" `
        --version $raversion
    ```
***

### Solution template for SSA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ssa-schema" --schema-file ./ssa-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    ssaversion="1.0.0"
    ssaname="${resourcePrefix}-ssa"
    az workload-orchestration solution-template create \
        --solution-template-name "$ssaname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "${resourcePrefix}-soap" \
        --description "This is SSA Solution" \
        --configuration-template-file ./ssa-config-template.yaml \
        --specification "@ssa-specs.json" \
        --version "$ssaversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "ssa-schema" --schema-file .\ssa-schema.yaml -l $l
    ```

1. Create the solution template file

    ````powershell
    $ssaversion = "1.0.0"
    $ssaname = "$resourcePrefix-ssa" 
    az workload-orchestration solution-template create `
        --solution-template-name "$ssaname" `
        -g $rg `
        -l $l `
        --capabilities "$resourcePrefix-soap" `
        --description "This is SSA Solution" `
        --configuration-template-file .\ssa-config-template.yaml `
        --specification "@ssa-specs.json" `
        --version $ssaversion
    ```
***

### Solution template for FSAD

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "fsad-schema" --schema-file ./fsad-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    fsadversion="1.0.0"
    fsadname="${resourcePrefix}-fsad"
    az workload-orchestration solution-template create \
        --solution-template-name "$fsadname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "${resourcePrefix}-soap" \
        --description "This is FSAD Solution" \
        --configuration-template-file ./fsad-config-template.yaml \
        --specification "@fsad-specs.json" \
        --version "$fsadversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "fsad-schema" --schema-file .\fsad-schema.yaml -l $l
    ```

1. Create the solution template file

    ````powershell
    $fsadversion = "1.0.0"
    $fsadname = "$resourcePrefix-fsad" 
    az workload-orchestration solution-template create `
        --solution-template-name "$fsadname" `
        -g $rg `
        -l $l `
        --capabilities "$resourcePrefix-soap" `
        --description "This is FSAD Solution" `
        --configuration-template-file .\fsad-config-template.yaml `
        --specification "@fsad-specs.json" `
        --version $fsadversion
    ```
***

## Set the configuration for the solution templates

### [Bash](#tab/bash)

1. Set the configuration for CA solution.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$caname" --target-name "${resourcePrefix}-SGCountry"
    ```

1. Set the configuration for RA solution.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$raname" --target-name "${resourcePrefix}-SGCountry"

    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$raname" --target-name "${resourcePrefix}-SGRegion"
    ```

1. Set the configuration for SSA solution.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$ssaname" --target-name "${resourcePrefix}-SGCountry"

    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$ssaname" --target-name "${resourcePrefix}-SGRegion"

    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$ssaname" --target-name "${resourcePrefix}-SGFactory"
    ```

1. Set the configuration for FSAD solution.

    ```bash
    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$fsadname" --target-name "${resourcePrefix}-SGCountry"

    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$fsadname" --target-name "${resourcePrefix}-SGRegion"

    az workload-orchestration configuration set --subscription "$contextSubscriptionId" -g "$contextRG" --solution-template-name "$fsadname" --target-name "${resourcePrefix}-SGFactory"

    az workload-orchestration configuration set -g "$rg" --solution-template-name "$fsadname" --target-name "$mk80Target"
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for CA solution.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $caname --target-name $resourcePrefix-SGCountry
    ```
1. Set the configuration for RA solution.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $raname --target-name $resourcePrefix-SGCountry
    
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $raname --target-name $resourcePrefix-SGRegion
    ```    
1. Set the configuration for SSA solution.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $ssaname --target-name $resourcePrefix-SGCountry

    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $ssaname --target-name $resourcePrefix-SGRegion
    
    
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $ssaname --target-name $resourcePrefix-SGFactory
    ```

1. Set the configuration for FSAD solution.

    ```powershell
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $fsadname --target-name $resourcePrefix-SGCountry
    
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $fsadname --target-name $resourcePrefix-SGRegion
    
    az workload-orchestration configuration set --subscription $contextSubscriptionId -g $contextRG --solution-template-name $fsadname --target-name $resourcePrefix-SGFactory
    
    az workload-orchestration configuration set -g $rg --solution-template-name $fsadname --target-name $mk80Target
    ```

***

## Review the configuration

### [Bash](#tab/bash)

1. Review the configuration for CA solution with "ca-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-name "$caname" --solution-template-version "$caversion" --resource-group "$rg" --target-name "$countryTarget" --solution-instance-name "ca-instance-a"
    ```

1. Review the configuration for RA solution with "ra-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-name "$raname" --solution-template-version "$raversion" --resource-group "$rg" --target-name "$regionTarget" --solution-instance-name "ra-instance-a"
    ```

1. Review the configuration for SSA solution with "ssa-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-name "$ssaname" --solution-template-version "$ssaversion" --resource-group "$rg" --target-name "$factoryTarget" --solution-instance-name "ssa-instance-a"
    ```

1. Review the configuration for FSAD solution with dependencies on CA, RA, and SSA solutions. In the *dependencies.json* file, replace `solutionVersionId` with the ID from the output of the previous commands.

    ```bash
    az workload-orchestration target review --solution-template-name "$fsadname" --solution-template-version "$fsadversion" --resource-group "$rg" --target-name "$mk80Target" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command. You will need it to publish the solution.

    ```bash
    reviewId="<reviewId>"
    version="<name>"
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration for CA solution with "ca-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-name $caname --solution-template-version $caversion --resource-group $rg --target-name $countryTarget --solution-instance-name "ca-instance-a"
    ```

1. Review the configuration for RA solution with "ra-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-name $raname --solution-template-version $raversion --resource-group $rg --target-name $regionTarget --solution-instance-name "ra-instance-a"
    ```

1. Review the configuration for SSA solution with "ssa-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-name $ssaname --solution-template-version $ssaversion --resource-group $rg --target-name $factoryTarget --solution-instance-name "ssa-instance-a"
    ```

1. Review the configuration for FSAD solution with dependencies on CA, RA, and SSA solutions. In the *dependencies.json* file, replace `solutionVersionId` with the ID from the output of the previous commands.

    ```powershell
    az workload-orchestration target review --solution-template-name $fsadname --solution-template-version $fsadversion --resource-group $rg --target-name $mk80Target --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command. You will need it to publish the solution.

    ```powershell
    $reviewId = "<reviewId>"
    $version = "<name>"
    ```
***

## Publish and deploy the solution

### [Bash](#tab/bash)

1. Publish the solution with the review ID and version name.

    ```bash
    az workload-orchestration target publish --solution-name "$fsadname" --solution-version "$version" --review-id "$reviewId" --resource-group "$rg" --target-name "$mk80Target"
    ```

1. Deploy the solution to the target.

    ```bash
    az workload-orchestration target install --solution-name "$fsadname" --solution-version "$version" --resource-group "$rg" --target-name "$mk80Target"
    ```

### [PowerShell](#tab/powershell)

1. Publish the solution with the review ID and version name.

    ```bash
    az workload-orchestration target publish --solution-name $fsadname --solution-version $version --review-id $reviewId --resource-group $rg --target-name $mk80Target 
    ```

1. Deploy the solution to the target.

    ```bash
    az workload-orchestration target install --solution-name $fsadname --solution-version $version --resource-group $rg --target-name $mk80Target 
    ```
***

## Validate the deployment

### [Bash](#tab/bash)

1. Get the cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

    ```bash
    az aks get-credentials --resource-group <ResourceGroupName> --name <ClusterName> --overwrite-existing
    ```

1. Validate the configuration of the targets by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create targets](#create-targets) step.

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

1. Validate the configuration of the targets by checking the Helm release values. Replace `<Namespace>` with the `-solution-scope` you defined in the [Create targets](#create-targets) step.

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
