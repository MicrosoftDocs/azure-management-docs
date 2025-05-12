---
title: Setup Workload Orchestration
description: Learn how to configure resources, author solutions, and manage deployments for Azure Arc workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: install-set-up-deploy
ms.date: 05/08/2025
---

# Setup workload orchestration

IT admins and developers are responsible for setting up and configuring workload orchestration. This includes configuring the resources, authoring solutions, and managing deployments. 

This article describes the steps to configure application specific resources such as workload orchestration resources, author solutions, and manage deployments. It also provides information about application and solution versioning, different ways to author configurations, and different solution authoring scenarios.

For information about prerequisites and initial setup, see [Prepare the environment for workload orchestration](initial-setup-environment.md).

## Configure the resources of workload orchestration

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

    # Enter name of Configuration Manager instance
    instanceName="redmondInstance"
    # Enter name of hierarchy list
    hierarchyName="hierarchyList1"
    # Enter comma-separated values with no space
    levelList="[factory,line]"
    # If you would prefer adding description to each of the hierarchy levels mentioned in the level list, then use the below format
    levelList='[{"name":"factory","description":"belongs to Factory and hence lines within the factory"},{"name":"line","description":"belongs to specific line"}]'
    # Enter name of capability list
    capListName="tagList1"
    # Enter comma-separated values with no space
    capFullList="[soap,shampoo,conditioner]"
    # If you would prefer adding description to each of the capabilities mentioned in the capability list, then use the below format
    capFullList='[{"name":"soap","description":"For soap production"},{"name":"shampoo","description":"For shampoo production"},{"name":"conditioner","description":"For conditioner production"}]'
    # Enter parent level name
    level1="factory"
    # Enter child level name
    level2="line"
    # The parent will be the site here
    parentName="$siteName"
    # Enter child name
    childName="Line01"
    # Enter capabilities of Mehoopany
    capParentList="[soap,conditioner,shampoo]"
    # Enter capabilities of Line01
    capChildList="[soap,conditioner]"
    # Enter description of Line01
    childDesc="This line is used for soap and conditioner production"
    ```

1. Create workload orchestration context with hierarchies. 

    ```bash
    az workload-orchestration context create --subscription "$subscriptionId" --resource-group "$rg" --location "$l" --name "$instanceName" --hierarchies "$levelList" --capabilities "$capFullList"
    ```

1. Create a site reference to link the workload orchestration instance to `Site` for parent hierarchical level operations.

    ```bash
    az workload-orchestration context site-reference create --subscription "$subscriptionId" --resource-group "$rg" --context-name "$instanceName" --name "$siteReference" --site-id "$siteId"
    ```

1. Create a JSON file named `targetspecs.json` by referring to the sample file [here](https://github.com/microsoft/AEP/blob/main/content/en/docs/Configuration%20Manager%20(Public%20Preview)/Scripts%20for%20Onboarding/Configuration%20manager%20files.zip).
1. Look up the custom location details.

    ```bash
    CustomLocationName=$(az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group "$rg" --name "$clusterName-Location" --query [].id --output tsv)
    ```

1. Create a target reference. The attribute `--solution-scope` is set to `new` to create a new target. The `--target-specification` attribute specifies that the Helm charts are being used for the K8s deployment. The `--extended-location` attribute is used to specify the custom location of the AKS cluster.

    ```bash
    az workload-orchestration target create --resource-group "$rg" --location "$l" --name "$childName" --display-name "$childName" --hierarchy-level "$level2" --capabilities "$capChildList" --description "$childDesc" --solution-scope "new" --target-specification '@targetspecs.json' --extended-location '@custom-location.json'
    ```

#### [Powershell](#tab/powershell)

1. Define workload orchestration variables. The following variables are used in the example. You can change the values as per your requirements.

    ```powershell
    # No space is allowed between comma-separated values for lists

    # Enter name of Configuration Manager instance
    $instanceName = "redmondInstance"
    # Enter name of hierarchy list
    $hierarchyName = "hierarchyList1"
    # Enter comma-separated values with no space
    $levelList = "[factory,line]"
    # If you would prefer adding description to each of the hierarchy levels mentioned in the level list, then use the below format
    $levelList = '[{"name":"factory","description":"belongs to Factory and hence lines within the factory"},{"name":"line","description":"belongs to specific line"}]'
    # Enter name of capability list
    $capListName = "tagList1"
    # Enter comma-separated values with no space
    $capFullList = "[soap,shampoo,conditioner]"
    # If you would prefer adding description to each of the capabilities mentioned in the capability list, then use the below format
    $capFullList = '[{"name":"soap","description":"For soap production"},{"name":"shampoo","description":"For shampoo production"},{"name":"conditioner","description":"For conditioner production"}]'
    # Enter parent level name
    $level1 = "factory"
    # Enter child level name
    $level2 = "line"
    # The parent will be the site here
    $parentName = $siteName
    # Enter child name
    $childName = "Line01"
    # Enter capabilities of Mehoopany
    $capParentList = "[soap,conditioner,shampoo]"
    # Enter capabilities of Line01
    $capChildList = "[soap,conditioner]"
    # Enter description of Line01
    $childDesc = "This line is used for soap and conditioner production"
    ```

1. Create workload orchestration context with hierarchies. 

    ```powershell
    az workload-orchestration context create --subscription $subscriptionId --resource-group $rg --location $l --name $instanceName --hierarchies $levelList --capabilities $capFullList
    ```

1. Create a site reference to link the workload orchestration instance to `Site` for parent hierarchical level operations.

    ```powershell
    az workload-orchestration context site-reference create --subscription $subscriptionId --resource-group $rg --context-name $instanceName --name $siteReference --site-id $siteId
    ```

1. Create a JSON file named `targetspecs.json` by referring to the sample file [here](https://github.com/microsoft/AEP/blob/main/content/en/docs/Configuration%20Manager%20(Public%20Preview)/Scripts%20for%20Onboarding/Configuration%20manager%20files.zip).
1. Look up the custom location details.

    ```powershell
    $CustomLocationName = (az resource list --resource-type Microsoft.ExtendedLocation/customLocations --resource-group $rg --name "$clusterName-Location" --query [].id --output tsv)
    ```

1. Create a target reference. The attribute `--solution-scope` is set to `new` to create a new target. The `--target-specification` attribute specifies that the Helm charts are being used for the K8s deployment. The `--extended-location` attribute is used to specify the custom location of the AKS cluster.

    ```powershell
    az workload-orchestration target create --resource-group $rg --location $l --name $childName --display-name $childName --hierarchy-level $level2 --capabilities $capChildList --description $childDesc --solution-scope "new" --target-specification "@targetspecs.json" --extended-location "@custom-location.json"
    ```

***

## Solution authoring and deployment 

The process of authoring and deploying solutions using workload orchestration involves creating schemas, configuration templates, and deploying solutions to the target environments. This process is typically performed by IT DevOps.

### Hierarchical configuration authoring

Solution authoring requires seamless integration of workload orchestration-managed ARM assets with user-provided artifacts, such as Helm charts and container images. Workload orchestration supports both CLI and portal-based methods for authoring solution configurations across various use cases.

The diagram below represents an example hierarchical configuration of objects:

:::image type="content" source="./media/hierarchy-configuration-objects.png" alt-text="Diagram illustrating the hierarchical configuration of objects in the Solution Authoring and Deployments process.":::

A typical solution consist of:

1. **Schema:** A schema is a JSON file that represents the declaration of configurable attributes of the solution and the associated permissions as it applies to hierarchies and personas. The schema is used to define the structure and format of the configuration data that is used in the solution. The schema is used to validate the configuration data before it's deployed to the target environment. For more information, see [Configuration Schema](configuring-schema.md).
1. **Configuration template:** A configuration template is a JSON file that  represents associated configurations of the previously declared schema. These values can be modified as necessary. See [Configuration template](configuring-template.md) for the list of rules used to define the template and schema for configurations. This page also details the steps to write conditional or nested expressions.
1. **Solution Helm chart:** A solution Helm chart is a package that contains all the necessary files and resources to deploy the solution to the target environment. The solution Helm chart integrates the configurable workload orchestration assets with the user provided application artifacts. Applications must be packaged as containers before uploading them to workload orchestration.
1. **Published solution configuration:** A published solution configuration is a JSON file that represents the final configuration of the solution after it's validated and approved. The published solution configuration is created by combining the schema, configuration template, and solution Helm chart. The published solution represents a fully rendered, a pre-deployment ready, targeted solution. At this point, the solution is ready to be deployed.


## Application and solution versioning

Application and solution versions must be manually updated. It's recommended to follow Semantic Versioning, a widely adopted versioning scheme that uses a three-part version number format: `major.minor.patch`. Each part of the version number reflects the scope of changes:

- *Major:* Incremented for changes that are not backward-compatible.
- *Minor:* Incremented for new features that are backward-compatible.
- *Patch:* Incremented for backward-compatible bug fixes.

Version information can also be specified in a file instead of being passed as a CLI argument when creating solution templates, schemas, or configuration templates. To include version details, add the following section to the YAML file:

```yaml
metadata:
    name: <name> [optional]
    version: <version> [optional]
```

## Different ways to author configurations

Once solution is uploaded to workload orchestration, IT DevOps author the configuration template and schema for validation rules. IT DevOps can provide technical configurations and set the default values and ranges for all configurations in the template and schema.

- Azure CLI: IT DevOps can provide values for these configurations via CLI. The CLI will validate the configurations and publish them to the target. For more information, see [Different solution authoring scenarios](#different-solution-authoring-scenarios).
- Workload orchestration portal: The published solutions and configurations are reflected on the portal. Any no-code persona is able to view the configurations and provide values via workload orchestration portal based on RBAC. For more information, see [Configure your solutions](ot-configure.md).

## Revisions of configurations

When user provides values for solution configurations and publishes them for certain targets, revisions of configurations are created for each target. These revisions are incremented with each new change made by user for respective target.

## Different solution authoring scenarios

There are different variants of schemas that can be used to author solutions. See the following quickstarts for examples of different solution authoring scenarios:

1. **Shared schema:** This schema comprises of configurable attributes/properties that can be used across hierarchies and solutions. For more information, see [Create a basic solution without common configurations](quickstart-solution-without-common-config.md).
1. **Common schema:** This schema defines configurable attributes/properties at each hierarchical level that can be used for a particular solution. For more information, see [Create a basic solution with common configurations](quickstart-solution-with-common-config.md).
1. **Schema with dependencies:** This schema defines configurable attributes/properties of an applications dependent on another application. For more information, see [Create a solution with shared adapter dependencies](quickstart-solution-shared-adapter-dependency.md).
1. **Multiple shared adapter dependencies:** This schema defines configurable attributes/properties of an application dependent on more than one other application. For more information, see [Create a solution with multiple shared adapter dependencies](quickstart-solution-multiple-shared-adapter-dependency.md).
1. **Deploy multiple instances of the same application:** This schema defines configurable attributes/properties of an application that can be deployed multiple times in the same namespace. For more information, see [Create a solution with multiple instances](quickstart-solution-multiple-instances-k8s.md).
1. **Upgrade a shared solution:** This schema shows how to upgrade shared solution along with dependent solutions. For more information, see [Upgrade a shared solution](quickstart-upgrade-shared-application.md).

## Related content

- [Onboarding scripts](onboarding-scripts.md)
- [Prepare the environment for workload orchestration](initial-setup-environment.md)
- [Service groups for workload orchestration](service-group-wo.md)
- [Monitor your solutions in Azure portal](it-azure-portal-monitoring.md)
