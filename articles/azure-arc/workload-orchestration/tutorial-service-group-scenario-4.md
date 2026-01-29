---
title: Multiple Solutions with a Single Shared Dependency at Different Levels
description: Learn how to create multiple solutions that share a single dependency at different levels of the hierarchy in Azure Arc-enabled Kubernetes.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: tutorial
ms.date: 06/01/2025
---

# Tutorial: Create multiple solutions with a single shared dependency at different levels using service groups

In this tutorial, you create a scenario with multiple solutions that share a single dependency at different levels of the hierarchy. You use service groups to orchestrate workloads across different levels of the hierarchy.

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

:::image type="content" source="./media/scenario-multiple-solutions-dependency.png" alt-text="Diagram of the four-level hierarchy and target at each level, where line level solution is dependent on the other solution." lightbox="./media/scenario-multiple-solutions-dependency.png":::

The sites references are created at each level of the hierarchy, being the Service Group Region (SGRegion) at the region level, Service Group City (SGCity) at the city level, Service Group Factory (SGFactory) at the factory level. A target is created at each level, being RegionTarget at the region level, CityTarget at the city level, FactoryTarget at the factory level, and LineTarget at the line level.

A solution is deployed at each target as follows:

- Line App is a solution at line level.
- Factory App is a solution at factory level.
- City App is a solution at city level.
- Global Adapter is a solution at region level. Line App, Factory App, and City App are dependent on Global Adapter, which means that during deployment, the City App, Factory App, and Line App configurations are inherited from the Global Adapter configuration.

All the instances of Line App, Factory App, City App, and Global Adapter are deployed in the same Azure Arc-enabled Kubernetes cluster. 

## Create targets

### [Bash](#tab/bash)

1. Create a target for each hierarchy level. Update *custom-location.json* file with the custom location of your cluster.

    ```bash
    solution_scope="n-to-one-app"  # if you want to change the name, make sure it follows the K8s object naming convention
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

1. Link the target IDs to their respective service groups.

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
    $solution_scope = "n-to-one-app"  # if you want to change the name, make sure it follows the K8s object naming convention
    $regionTarget = "Italy"
    $cityTarget = "Naples"
    $factoryTarget = "Contoso"
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

1. Link the target IDs to their respective service groups. 

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

To create the solution schema, configuration templates and solution template, you can use the sample files provided in **Service Groups/Scenario4_NApp1Dependency** within [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) as reference.

### Solution template for Global Adapter

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "ga-schema" --schema-file ./ga-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    gaversion="1.0.0"
    ganame="${resourcePrefix}-ga"
    az workload-orchestration solution-template create \
        --solution-template-name "$ganame" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is Global Adapter Solution" \
        --configuration-template-file ./ga-config-template.yaml \
        --specification "@ga-specs.json" \
        --version "$gaversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "ga-schema" --schema-file .\ga-schema.yaml -l $l
    ```

1. Create the solution template file

    ```powershell
    $gaversion = "1.0.0"
    $ganame = "$resourcePrefix-ga" 
    az workload-orchestration solution-template create `
        --solution-template-name "$ganame" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is Global Adapter Solution" `
        --configuration-template-file .\ga-config-template.yaml `
        --specification "@ga-specs.json" `
        --version $gaversion
    ```
***

### Solution template for City App

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "capp-schema" --schema-file ./capp-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    cappversion="1.0.0"
    cappname="${resourcePrefix}-capp"
    az workload-orchestration solution-template create \
        --solution-template-name "$cappname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is City App Solution" \
        --configuration-template-file ./capp-config-template.yaml \
        --specification "@capp-specs.json" \
        --version "$cappversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "capp-schema" --schema-file .\capp-schema.yaml -l $l
    ```

1. Create the solution template file

    ```powershell
    $cappversion = "1.0.0"
    $cappname = "$resourcePrefix-capp" 
    az workload-orchestration solution-template create `
        --solution-template-name "$cappname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is City App Solution" `
        --configuration-template-file .\rapp-config-template.yaml `
        --specification "@capp-specs.json" `
        --version $cappversion
    ```
***

### Solution template for Factory App

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "fapp-schema" --schema-file ./fapp-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    fappversion="1.0.0"
    fappname="${resourcePrefix}-fapp"
    az workload-orchestration solution-template create \
        --solution-template-name "$fappname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is Factory App Solution" \
        --configuration-template-file ./fapp-config-template.yaml \
        --specification "@fapp-specs.json" \
        --version "$fappversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "fapp-schema" --schema-file .\fapp-schema.yaml -l $l
    ```

1. Create the solution template file

    ```powershell
    $fappversion = "1.0.0"
    $fappname = "$resourcePrefix-fapp" 
    az workload-orchestration solution-template create `
        --solution-template-name "$fappname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is Factory App Solution" `
        --configuration-template-file .\fapp-config-template.yaml `
        --specification "@fapp-specs.json" `
        --version $fappversion
    ```
***

### Solution template for Line App

#### [Bash](#tab/bash)

1. Create the solution schema file.

    ```bash
    az workload-orchestration schema create --resource-group "$rg" --version "1.0.0" --schema-name "lapp-schema" --schema-file ./lapp-schema.yaml -l "$l"
    ```

1. Create the solution template file

    ```bash
    lappversion="1.0.0"
    lappname="${resourcePrefix}-lapp"
    az workload-orchestration solution-template create \
        --solution-template-name "$lappname" \
        -g "$rg" \
        -l "$l" \
        --capabilities "Use for soap production" \
        --description "This is Line App Solution" \
        --configuration-template-file ./lapp-config-template.yaml \
        --specification "@lapp-specs.json" \
        --version "$lappversion"
    ```

#### [PowerShell](#tab/powershell)

1. Create the solution schema file.

    ```powershell
    az workload-orchestration schema create --resource-group $rg --version "1.0.0" --schema-name "lapp-schema" --schema-file .\lapp-schema.yaml -l $l
    ```

1. Create the solution template file

    ```powershell
    $lappversion = "1.0.0"
    $lappname = "$resourcePrefix-lapp" 
    az workload-orchestration solution-template create `
        --solution-template-name "$lappname" `
        -g $rg `
        -l $l `
        --capabilities "Use for soap production" `
        --description "This is Line App Solution" `
        --configuration-template-file .\lapp-config-template.yaml `
        --specification "@lapp-specs.json" `
        --version $lappversion
    ```
***

## Set the configuration for the solution templates

### [Bash](#tab/bash)

1. Set the configuration for Global Adapter solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$ganame" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$regionTarget" --solution
    ```

1. Set the configuration for City App solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$cappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget" --solution
    ```

1. Set the configuration for Factory App solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$fappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget" --solution
    ```

1. Set the configuration for Line App solution.

    ```bash
    az workload-orchestration configuration set -g "$rg" --template-name "$lappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget" --solution
    ```

### [PowerShell](#tab/powershell)

1. Set the configuration for Global Adapter solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$ganame" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$regionTarget" --solution
    ```

1. Set the configuration for City App solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$cappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget" --solution
    ```

1. Set the configuration for Factory App solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$fappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget" --solution
    ```

1. Set the configuration for Line App solution.

    ```powershell
    az workload-orchestration configuration set -g "$rg" --template-name "$lappname" --hierarchy-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget" --solution
    ```

***

## Review the Global Adapter configuration

Review the configuration for Global Adapter solution with "ga-instance-a" instance.

### [Bash](#tab/bash)

```bash
az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$ganame/versions/$gaversion  --resource-group "$rg" --target-name "$regionTarget" --solution-instance-name "ga-instance-a"
```

### [PowerShell](#tab/powershell)


```powershell
az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$ganame/versions/$gaversion --resource-group $rg --target-name $regionTarget --solution-instance-name "ga-instance-a"
```

***

In the *dependencies.json* file, replace `solutionVersionId` with the ID from the output of the previous commands.

## Review, publish, and deploy City App solution

### [Bash](#tab/bash)

1. Review the configuration for City App solution with dependency on Global Adapter solution.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$cappname/versions/$cappversion  --resource-group "$rg" --target-name "$cityTarget" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```bash
    cappReviewId="<reviewId>"
    cappSolutionVersion="<name>"
    ```

1. Publish the City App solution.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$cityTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget/solutions/$cappname/versions/$cappSolutionVersion
    ```

1. Deploy the City App solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$cityTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget/solutions/$cappname/versions/$cappSolutionVersion
    ```


### [PowerShell](#tab/powershell)

1. Review the configuration for City App solution with dependency on Global Adapter solution.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$cappname/versions/$cappversion  --resource-group $rg --target-name $cityTarget --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```powershell
    $cappReviewId = "<reviewId>"
    $cappSolutionVersion = "<name>"
    ```

1. Publish the City App solution.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $cityTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget/solutions/$cappname/versions/$cappSolutionVersion
    ```

1. Deploy the City App solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $cityTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$cityTarget/solutions/$cappname/versions/$cappSolutionVersion
    ```
***

## Review, publish, and deploy Factory App solution

### [Bash](#tab/bash)

1. Review the configuration for Factory App solution with dependency on Global Adapter solution.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$fappname/versions/$fappversion  --resource-group "$rg" --target-name "$factoryTarget" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```bash
    fappReviewId="<reviewId>"
    fappSolutionVersion="<name>"
    ```

1. Publish the Factory App solution.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$factoryTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget/solutions/$fappname/versions/$fappSolutionVersion
    ```

1. Deploy the Factory App solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$factoryTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget/solutions/$fappname/versions/$fappSolutionVersion
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration for Factory App solution with dependency on Global Adapter solution.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$fappname/versions/$fappversion  --resource-group $rg --target-name $factoryTarget --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```powershell
    $fappReviewId = "<reviewId>"
    $fappSolutionVersion = "<name>"
    ```

1. Publish the Factory App solution.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $factoryTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget/solutions/$fappname/versions/$fappSolutionVersion
    ```

1. Deploy the Factory App solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $factoryTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$factoryTarget/solutions/$fappname/versions/$fappSolutionVersion 
    ```
***

## Review, publish, and deploy Line App solution

### [Bash](#tab/bash)

1. Review the configuration for Line App solution with dependency on Global Adapter solution.

    ```bash
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$lappname/versions/$lappversion  --resource-group "$rg" --target-name "$lineTarget" --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```bash
    lappReviewId="<reviewId>"
    lappSolutionVersion="<name>"
    ```

1. Publish the Line App solution.

    ```bash
    az workload-orchestration target publish --resource-group "$rg" --target-name "$lineTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$lappname/versions/$lappSolutionVersion
    ```

1. Deploy the Line App solution.

    ```bash
    az workload-orchestration target install --resource-group "$rg" --target-name "$lineTarget" --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$lappname/versions/$lappSolutionVersion
    ```

### [PowerShell](#tab/powershell)

1. Review the configuration for Line App solution with dependency on Global Adapter solution.

    ```powershell
    az workload-orchestration target review --solution-template-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/solutionTemplates/$lappname/versions/$lappversion  --resource-group $rg --target-name $lineTarget --solution-dependencies "@dependencies.json"
    ```

1. Copy the `reviewId` from the output of the previous command.

    ```powershell
    $lappReviewId = "<reviewId>"
    $lappSolutionVersion = "<name>"
    ```

1. Publish the Line App solution.

    ```powershell
    az workload-orchestration target publish --resource-group $rg --target-name $lineTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$lappname/versions/$lappSolutionVersion
    ```

1. Deploy the Line App solution.

    ```powershell
    az workload-orchestration target install --resource-group $rg --target-name $lineTarget --solution-version-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/targets/$lineTarget/solutions/$lappname/versions/$lappSolutionVersion
    ```
***

## Validate the deployment

### [Bash](#tab/bash)

1. Get cluster credentials for the target. The cluster should be linked to the custom location which used in target creation.

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

1. If you want to check the configurations individually, run:

    ```bash
    # If you don't want to print all configs in the namespace, run
    helm list -A
    # Supply name and namespace in the command below to see configs for a specific release
    helm get values <name> -n <namespace>
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
