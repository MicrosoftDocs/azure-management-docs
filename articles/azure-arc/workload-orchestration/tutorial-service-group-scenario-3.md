---
title: Solution with Multiple Shared Dependencies at Different Hierarchy Levels
description: Learn how to create a solution with multiple shared dependencies at different hierarchy levels in Azure Arc-enabled Kubernetes.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/01/2025
---

# Tutorial: Create a solution with multiple shared dependencies at different levels using service groups

In this tutorial, you create a solution with multiple shared dependencies at different levels in a four-level hierarchy organization. You use service groups to orchestrate workloads across different levels of the hierarchy.

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

:::image type="content" source="./media/scenario-single-solution-dependencies.png" alt-text="Diagram of the four-level hierarchy and target at each level, where line level solution is dependent on the other solution." lightbox="./media/scenario-single-solution-dependencies.png":::

The sites references are created at each level of the hierarchy, being SGRegion at the region level, SGCity at the city level, SGFactory at the factory level. A target is created at each level, being RegionTarget at the region level, CityTarget at the city level, FactoryTarget at the factory level, and LineTarget at the line level.

A solution is deployed at each target as follows:

- Region Adapter (RA) is a solution at region level.
- City Adapter (CA) is a solution at city level.
- Factory Adapter (FA) is a solution at factory level.
- Line Event Tracker (LET) is a solution at line level. LET is dependent on FA, RA, and CA, which means that during the deployment, LET configuration will be updated with the information from the other solutions.

All the instances of RA, CA, FA, and LET are deployed in the same Azure Arc-enabled Kubernetes cluster.

## Create targets

### [Bash](#tab/bash)

1. Create a target for each hierarchy level. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    solution_scope="one-to-n-app"  # if you want to change the name, make sure it follows the K8s object naming convention
    regionTarget="Italy"
    cityTarget="Naples"
    factoryTarget="Contoso"
    lineTarget="Line01"

    # Create target at region level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$regionTarget" \
      --display-name "$regionTarget" \
      --hierarchy-level "region" \
      --capabilities "Use for soap production" \
      --description "This is Region Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json" \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextRegionName"

    # Create target at city level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$cityTarget" \
      --display-name "$cityTarget" \
      --hierarchy-level "city" \
      --capabilities "Use for soap production" \
      --description "This is City Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json" \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextCityName"

    # Create target at factory level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$factoryTarget" \
      --display-name "$factoryTarget" \
      --hierarchy-level "factory" \
      --capabilities "Use for soap production" \
      --description "This is Factory Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json" \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextFactoryName"

    # Create target at line level
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$lineTarget" \
      --display-name "$lineTarget" \
      --hierarchy-level "line" \
      --capabilities "Use for soap production" \
      --description "This is Line01 Target" \
      --solution-scope "$solution_scope" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json" \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextLineName"
    ```

1. Get the target IDs of the created targets.

    ```bash
    lineTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$lineTarget" --query id --output tsv)
    factoryTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$factoryTarget" --query id --output tsv)
    cityTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$cityTarget" --query id --output tsv)
    regionTargetId=$(az workload-orchestration target show --resource-group "$rg" --name "$regionTarget" --query id --output tsv)
    ```

1. Link the target IDs to their respective service groups. Make sure to replace the service group names with the ones you created.

    ```bash
    # Link to region service group
    az rest \
      --method put \
      --uri "${regionTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level1Name'}}"

    # Link to city service group
    az rest \
      --method put \
      --uri "${cityTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level2Name'}}"

    # Link to factory service group
    az rest \
      --method put \
      --uri "${factoryTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level3Name'}}"

    # Link to line service group
    az rest \
      --method put \
      --uri "${lineTargetId}/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level3Name'}}"
    ```

1. Update the targets after connecting them to the service groups to make sure the hierarchy configurations are updated. This step is optional but recommended.

    ```bash
    # Update region target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$regionTarget"

    # Update city target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$cityTarget"

    # Update factory target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$factoryTarget"

    # Update line target
    az workload-orchestration target update \
      --resource-group "$rg" \
      --name "$lineTarget"
    ```

### [PowerShell](#tab/powershell)

1. Create a target for each hierarchy level. Update *custom-location.json* file with the custom location of your cluster.

    ```powershell
    $solution_scope = "one-to-n-app"  # if you want to change the name, make sure it follows the K8s object naming convention
    $regionTarget = "Italy"
    $cityTarget = "Naples"
    $factoryTarget = "ContosoLtd"
    $lineTarget = "Line01"
    
    # Create target at region level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $regionTarget `
      --display-name $regionTarget `
      --hierarchy-level "region" `
      --capabilities "Use for soap production" `
      --description "This is Region Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextRegionName
    
    # Create target at city level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $cityTarget `
      --display-name $cityTarget `
      --hierarchy-level "city" `
      --capabilities "Use for soap production" `
      --description "This is City Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextCityName
    
    # Create target at factory level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $factoryTarget `
      --display-name $factoryTarget `
      --hierarchy-level "factory" `
      --capabilities "Use for soap production" `
      --description "This is Factory Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' ` 
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextFactoryName
    
    # Create target at line level
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $lineTarget `
      --display-name $lineTarget `
      --hierarchy-level "line" `
      --capabilities "Use for soap production" `
      --description "This is Line01 Target" `
      --solution-scope $solution_scope `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextLineName
    ```

1. Get the target IDs of the created targets.

    ```powershell
    $lineTargetId = $(az workload-orchestration target show --resource-group $rg --name "$lineTarget" --query id --output tsv)
    $factoryTargetId = $(az workload-orchestration target show --resource-group $rg --name "$factoryTarget" --query id --output tsv)
    $cityTargetId = $(az workload-orchestration target show --resource-group $rg --name "$cityTarget" --query id --output tsv)
    $regionTargetId = $(az workload-orchestration target show --resource-group $rg --name "$regionTarget" --query id --output tsv)
    ```

1. Link the target IDs to their respective service groups. Make sure to replace the service group names with the ones you created.

    ```powershell
    #Link to region service group
    az rest `
      --method put `
      --uri $regionTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level1Name'}}"
    
    #Link to city service group
    az rest `
      --method put `
      --uri $cityTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level2Name'}}"
    
    #Link to factory service group
    az rest `
      --method put `
      --uri $factoryTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level3Name'}}"
    
    #Link to line service group
    az rest `
      --method put `
      --uri $lineTargetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$level3Name'}}"
    ```

1. Update the targets after connecting them to the service groups to make sure the hierarchy configurations are updated. This step is optional but recommended.

    ```powershell
    #Update region target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $regionTarget 
    
    #Update city target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $cityTarget 
    
    #Update factory target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $factoryTarget 
    
    #Update line target
    az workload-orchestration target update `
      --resource-group $rg `
      --name $lineTarget
    ```
***

## Prepare the solution templates

To create the solution schema, configuration templates and solution template, you can use the sample files provided in **Service Groups/Scenario3_1AppNDependency** within [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) as reference.

### Solution template for RA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ra-schema" --schema-file ./ra-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    raversion="1.0.0"
    raname="ra-template"

    az workload-orchestration solution-template create \
        --solution-template-name "$raname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
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
    $raname = "ra-template" 
    az workload-orchestration solution-template create `
        --solution-template-name "$raname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is RA Solution" `
        --configuration-template-file .\ra-config-template.yaml `
        --specification "@ra-specs.json" `
        --version $raversion
    ```
***

### Solution template for CA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ca-schema" --schema-file ./ca-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    caversion="1.0.0"
    caname="ca-template"
    az workload-orchestration solution-template create \
        --solution-template-name "$caname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
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
    $caname = "ca-template" 
    az workload-orchestration solution-template create `
        --solution-template-name "$caname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is CA Solution" `
        --configuration-template-file .\ca-config-template.yaml `
        --specification "@ca-specs.json" `
        --version $caversion
    ```
***

### Solution template for FA

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "fa-schema" --schema-file ./fa-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    faversion="1.0.0"
    faname="fa-template"
    az workload-orchestration solution-template create \
        --solution-template-name "$faname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is fa Solution" \
        --configuration-template-file ./fa-config-template.yaml \
        --specification "@fa-specs.json" \
        --version "$faversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "fa-schema" --schema-file .\fa-schema.yaml -l $l
    ```

1. Create the solution template file

    ````powershell
    $faversion = "1.0.0"
    $faname = "fa-template" 
    az workload-orchestration solution-template create `
        --solution-template-name "$faname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is FA Solution" `
        --configuration-template-file .\fa-config-template.yaml `
        --specification "@fa-specs.json" `
        --version $faversion
    ```
***

### Solution template for LET

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "let-schema" --schema-file ./let-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    letversion="1.0.0"
    letname="let-template"
    az workload-orchestration solution-template create \
        --solution-template-name "$letname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is LET Solution" \
        --configuration-template-file ./let-config-template.yaml \
        --specification "@let-specs.json" \
        --version "$letversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "let-schema" --schema-file .\let-schema.yaml -l $l
    ```

1. Create the solution template file

    ```powershell
    $letversion = "1.0.0"
    $letname = "let-template" 
    az workload-orchestration solution-template create `
        --solution-template-name "$letname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is LET Solution" `
        --configuration-template-file .\let-config-template.yaml `
        --specification "@let-specs.json" `
        --version $letversion
    ```
***

## Set configuration for the solution templates

### [Bash](#tab/bash)

1. Set the configuration for RA solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$raname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$regionTarget" --solution
    ```

1. Set the configuration for CA solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$caname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget" --solution
    ```

1. Set the configuration for FA solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$faname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget" --solution
    ```

1. Set the configuration for LET solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$letname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget" --solution
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for RA solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$raname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$regionTarget" --solution
    ```

1. Set the configuration for CA solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$caname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget" --solution
    ```

1. Set the configuration for FA solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$faname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget" --solution
    ```

1. Set the configuration for LET solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$letname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget" --solution
    ```

***

## Review the configuration

### [Bash](#tab/bash)

1. Review the configuration for RA solution with "ra-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$raname/versions/$raversion  --resource-group "$rg" --target-name "$regionTarget" --solution-instance-name "ra-instance-a"
    ```

1. Review the configuration for CA solution with "ca-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$caname/versions/$caversion  --resource-group "$rg" --target-name "$cityTarget" --solution-instance-name "ca-instance-a"
    ```

1. Review the configuration for FA solution with "fa-instance-a" instance.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$faname/versions/$faversion  --resource-group "$rg" --target-name "$factoryTarget" --solution-instance-name "fa-instance-a"
    ```

1. Review the configuration for LET solution with dependencies on RA, CA, and FA solutions. In the *dependencies.json* file, replace `solutionVersionId` with the ID from the output of the previous commands.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$letname/versions/$letversion  --resource-group "$rg" --target-name "$lineTarget" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command. You will need it to publish the solution.

    ```bash
    reviewId="<reviewId>"
    version="<name>"
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration for RA solution with "ra-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$raname/versions/$raversion  --resource-group $rg --target-name $regionTarget --solution-instance-name "ra-instance-a"
    ```

1. Review the configuration for CA solution with "ca-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$caname/versions/$caversion  --resource-group $rg --target-name $cityTarget --solution-instance-name "ca-instance-a"
    ```

1. Review the configuration for FA solution with "fa-instance-a" instance.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$faname/versions/$faversion  --resource-group $rg --target-name $factoryTarget --solution-instance-name "fa-instance-a"
    ```

1. Review the configuration for LET solution with dependencies on RA, CA, and FA solutions. In the *dependencies.json* file, replace `solutionVersionId` with the ID from the output of the previous commands.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$letname/versions/$letversion --resource-group $rg --target-name $lineTarget --solution-dependencies "@dependencies.json"
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
    az workload-orchestration target publish --resource-group "$rg" --target-name "$lineTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$letName/versions/$version
    ```

1. Deploy the solution to the target.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$lineTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$letname/versions/$version
    ```

### [PowerShell](#tab/powershell)

1. Publish the solution with the review ID and version name.

    ```bash
    az workload-orchestration target publish --resource-group $rg --target-name $lineTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$letName/versions/$version
    ```

1. Deploy the solution to the target.

    ```bash
    az workload-orchestration target install --resource-group $rg --target-name $lineTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$letname/versions/$version
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

1. If you want to check the configurations individually, run:

    ```powershell
    # If you dont want to print all configs in namespace then run
    helm list -A
    # Supply name and namespace in below command to see configs for respective case
    helm get values <name> -n <namespace>
    ```
***

You can also go to the [Azure portal](https://portal.azure.com) and check pods in the namespace of your AKS cluster to validate the deployment.
