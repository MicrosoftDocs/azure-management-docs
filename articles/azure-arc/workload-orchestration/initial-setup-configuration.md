---
title: Set Up Workload Orchestration
description: Learn how to configure resources, author solutions, and manage deployments for Azure Arc workload orchestration.
ms.custom:
  - references_regions
  - build-2025
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: install-set-up-deploy
ms.date: 09/05/2025
# Customer intent: As an IT admin, I want to configure and manage workload orchestration resources so that I can effectively deploy applications across my Kubernetes clusters while ensuring streamlined processes and compliance with organizational requirements.
---

# Set up workload orchestration

IT admins and developers are responsible for setting up and configuring workload orchestration. This includes configuring the resources, authoring solutions, and managing deployments. 

This article describes the steps to configure application specific resources such as workload orchestration resources, author solutions, and manage deployments. It also provides information about application and solution versioning, different ways to author configurations, and different solution authoring scenarios.

> [!TIP]
> You can follow the instructions in this article and run through each command, or if you prefer, you can run the [onboarding scripts](onboarding-scripts.md) for a one-click setup.

## Prerequisites

Before you begin, you need to have the prerequisites and ensure you followed the steps listed in [Prepare the environment for workload orchestration](initial-setup-environment.md).

> [!IMPORTANT]
> Standard Azure resources, such as Arc-enabled Kubernetes clusters and custom location, and workload orchestration resources, such as context, targets, and solutions, should be created in the same Azure region. 

## Set up the resources of workload orchestration

Workload orchestration is implemented as a resource provider in Azure and exposes multiple resource types that stitch the whole experience together. The IT admin is responsible for setting up the resources that are needed to run workload orchestration. The resources are created in the Azure portal and are used to manage the deployment of applications across the Arc-enabled Kubernetes cluster.

Some of the resources that need to be set up by the IT admin are:

- **Hierarchies:** Name-description pairs that define the levels of hierarchical structure resonating with the customer’s resource topology. For example, a manufacturing customer can have two levels, Factory and Line.
- **Capabilities:** Name-description pairs that describe what a resource is capable of doing. 
- **Targets:** The actual resources. They're typically a Kubernetes cluster or custom location within Kubernetes cluster where you can deploy applications. These resources need to be tagged with 'Capability Lists' for application deployment purposes.

> [!IMPORTANT]
> Workload orchestration resources should be created in EastUS and EastUS2 Azure regions.

The following steps show how to configure the resources of workload orchestration.

#### [Bash](#tab/bash)

1. Define workload orchestration variables. The following variables are used in the example. You can change the values as per your requirements.

    ```bash
    # No space is allowed between comma-separated values for lists

    # Enter resource group 
    rg="<resource-group-name>"
    # Enter name of Configuration Manager instance
    instanceName="redmondInstance"
    # Enter name of hierarchy list
    hierarchyName="hierarchyList1"
    # Enter name of capability list
    capListName="tagList1"
    # Enter parent level name
    level1="factory"
    # Enter child level name
    level2="line"
    # The parent will be the site here
    parentName="$siteName"
    # Enter child name
    childName="Line01"
    # Enter description of Line01
    childDesc="This line is used for soap and conditioner production"
    # Enter name of site
    siteName="Site01"
    # Enter id of the site
    siteId="/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$sitename"
    # Enter capabilities for child target
    capChildList="[soap,shampoo,conditioner]"
    ```

    > [!NOTE]
    > All resource names—including targets, solution templates, configuration schemas, and instance names—must follow the naming pattern given by the regular expression `^(?!v-)(?!.*-v-)a-zA-Z0-9?(\.a-zA-Z0-9?)*$`. This means:
    > - Must start and end with an alphanumeric character.
    > - Can contain hyphens, but not at the start or end of any segment.
    > - Can contain dots to separate segments, but not consecutive dots or empty segments.
    > - Can't have any special characters other than hyphen and dot.
    > - Can't start with v-.
    > - Can't contain -v- anywhere.

1. Create workload orchestration context with hierarchies. Edit the `--hierarchies` and `--capabilities` parameters as per your requirements. The following example creates a context with two hierarchies: `factory` and `line`, and three capabilities: `soap`, `shampoo`, and `conditioner`.

    ```bash
    az workload-orchestration context create --subscription "$subId" --resource-group "$rg" --location "$l" --name "$instanceName" \
      --hierarchies '[{"name":"factory","description":"belongs to Factory and hence lines within the factory"},{"name":"line","description":"belongs to specific line"}]' \
      --capabilities '[{"name":"soap","description":"For soap production"},{"name":"shampoo","description":"For shampoo production"},{"name":"conditioner","description":"For conditioner production"}]'
    ```

    You can also use an already existing context by running the `context create` command with the `--context-id` parameter while passing the desired list of capabilities and hierarchies into it. You can add more capabilities, but removing and deleting isn't supported.
    
    ```bash
    az workload-orchestration context create --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName" --hierarchies "<hierarchies-list>" --capabilities "<capabilities-list>"
    ```
    
    You can set the current context to be used or view details about the same.

    ```bash
    # Set current context by name and resource group
    az workload-orchestration context use -n "$contextName" -g "$rg"

    # Set current context using ARM resource ID 
    az workload-orchestration context set --id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName"
    
    # Display current context information
    az workload-orchestration context current 
    ```

    > [!NOTE]
    > The context name must be between 3 and 61 characters in length and follow the naming pattern defined by the regular expression `^a-zA-Z0-9?(\.a-zA-Z0-9?)*$`. This means:
    > - Must start and end with an alphanumeric character.
    > - Can contain hyphens, but not at the start or end of any segment.
    > - Can contain dots to separate segments, but not consecutive dots or empty segments.
    > - Can't have any special characters other than hyphen and dot.

1. Create a site reference to link the workload orchestration instance to `Site` for parent hierarchical level operations.

    ```bash
    az workload-orchestration context site-reference create --subscription "$subId" --resource-group "$rg" --context-name "$instanceName" --name "$siteReference" --site-id "$siteId"
    ```

1. Create JSON files named `targetspecs.json` and `custom-location.json` by referring to the files in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```bash
    CustomLocationName=$(az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group "$rg" --name "$clusterName-Location" --query [].id --output tsv)
    ```

1. Create a target reference. The attribute `--solution-scope` is set to `new` to create a new target. The `--target-specification` attribute specifies that the Helm charts are being used for the K8s deployment. The `--extended-location` attribute is used to specify the custom location of the AKS cluster.

    ```bash
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$childName" --display-name "$childName" --hierarchy-level "$level2" --capabilities "$capChildList" --description "$childDesc" --solution-scope "new" --target-specification '@targetspecs.json' --extended-location '@custom-location.json' --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

#### [PowerShell](#tab/powershell)

1. Define workload orchestration variables. The following variables are used in the example. You can change the values as per your requirements.

    ```powershell
    # No space is allowed between comma-separated values for lists

    # Enter resource group 
    $rg = "<resource-group-name>"
    # Enter name of Configuration Manager instance
    $instanceName = "redmondInstance"
    # Enter name of hierarchy list
    $hierarchyName = "hierarchyList1"
    # Enter name of capability list
    $capListName = "tagList1"
    # Enter parent level name
    $level1 = "factory"
    # Enter child level name
    $level2 = "line"
    # The parent will be the site here
    $parentName = $siteName
    # Enter child name
    $childName = "Line01"
    # Enter description of Line01
    $childDesc = "This line is used for soap and conditioner production"
    # Enter name of site
    $siteName = "Site01"
    # Enter id of the site
    $siteId = "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/sites/$sitename"
    # Enter capabilities for child target
    $capChildList = "[soap, shampoo, conditioner]" 
    ```

    > [!NOTE]
    > All resource names—including targets, solution templates, configuration schemas, and instance names—must follow the naming pattern given by the regular expression `^(?!v-)(?!.*-v-)a-zA-Z0-9?(\.a-zA-Z0-9?)*$`. This means:
    > - Must start and end with an alphanumeric character.
    > - Can contain hyphens, but not at the start or end of any segment.
    > - Can contain dots to separate segments, but not consecutive dots or empty segments.
    > - Can't have any special characters other than hyphen and dot.
    > - Can't start with v-.
    > - Can't contain -v- anywhere.

1. Create workload orchestration context with hierarchies. Edit the `--hierarchies` and `--capabilities` parameters as per your requirements. The following example creates a context with two hierarchies: `factory` and `line`, and three capabilities: `soap`, `shampoo`, and `conditioner`.

    ```powershell
    az workload-orchestration context create --subscription $subId --resource-group $rg --location $l --name $instanceName --hierarchies "[0].name=factory" "[0].description=belongs to Factory and hence lines within the factory" "[1].name=line" "[1].description=belongs to specific line" --capabilities "[0].name=soap" "[0].description=For soap production" "[1].name=shampoo" "[1].description=For shampoo production" "[2].name=conditioner" "[2].description=For conditioner production"
    ```

    You can also use an already existing context by running the `context create` command with the `--context-id` parameter while passing the desired list of capabilities and hierarchies into it. You can add more capabilities, but removing and deleting isn't supported.
    
    ```powershell
    az workload-orchestration context create --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName --hierarchies "<hierarchies-list" --capabilities "<capabilities-list>"
    ``` 

    You can set the current context to be used or view details about the same.

    ```powershell
    # Set current context by name and resource group
    az workload-orchestration context use -n $contextName -g $rg

    # Set current context using ARM resource ID 
    az workload-orchestration context set --id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/context/$contextName

    # Display current context information
    az workload-orchestration context current 
    ```

    > [!NOTE]
    > The context name must be between 3 and 61 characters in length and follow the naming pattern defined by the regular expression `^a-zA-Z0-9?(\.a-zA-Z0-9?)*$`. This means:
    > - Must start and end with an alphanumeric character.
    > - Can contain hyphens, but not at the start or end of any segment.
    > - Can contain dots to separate segments, but not consecutive dots or empty segments.
    > - Can't have any special characters other than hyphen and dot.

1. Create a site reference to link the workload orchestration instance to `Site` for parent hierarchical level operations.

    ```powershell
    az workload-orchestration context site-reference create --subscription $subId --resource-group $rg --context-name $instanceName --name $siteReference --site-id $siteId
    ```

1. Create JSON files named `targetspecs.json` and `custom-location.json` by referring to the files in the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip).
1. Look up the custom location details.

    ```powershell
    $CustomLocationName = (az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group $rg --name "$clusterName-Location" --query [].id --output tsv)
    ```

1. Create a target reference. The attribute `--solution-scope` is set to `new` to create a new target. The `--target-specification` attribute specifies that the Helm charts are being used for the K8s deployment. The `--extended-location` attribute is used to specify the custom location of the AKS cluster.

    ```powershell
    az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope "new" --target-specification "@targetspecs.json" --extended-location "@custom-location.json" --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

***

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]

## Next steps

Once you have set up the infrastructure and the workload orchestration resources, you can start authoring solutions and managing deployments. To get started, see the [Quickstart: Create a basic solution](quickstart-solution-without-common-configuration.md) to learn how to create a basic solution, configure it, and deploy it to a target.

