---
title: Service Groups for Workload Orchestration
description: Learn about service groups and how to configure them in workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 11/05/2025
ms.custom:
  - build-2025
# Customer intent: As a cloud administrator, I want to configure service groups for workload orchestration, so that I can efficiently organize and manage resources across multiple subscriptions and apply governance effectively.
---

# Service groups for workload orchestration

Service groups are a new resource type in Azure Resource Manager (ARM) that help you organize related resources, like resource groups, subscriptions, and management groups, under one service, application, or workload. This article explains how to create a service group and configure it to use it with workload orchestration.

For more information, see [RBAC for service groups](rbac-guide.md#rbac-for-service-groups).

[!INCLUDE [service-groups-note](includes/service-groups-note.md)]

## What is a service group?

Service groups are tenant-level resource containers that represent a subset of collection of resources across Azure subscriptions or resource groups. They allow you to organize selected resources into a unified logical grouping, while maintaining your existing setup. 

Service groups also allow you to create layers (self-nesting) so you can organize services in a way that matches your real-world structure. This setup gives you a consistent view of your services, making it easier to manage visibility, tagging, policy enforcement, and governance, no matter how your resources are actually organized.

Every service group has a display name and one parent service group. Relationships are created between resources to establish a hierarchical structure such as parent and child associations.

The following diagram shows how service groups structure groups related components and deployment targets into logical units, making the architecture more modular and easier to manage.

:::image type="content" source="./media/service-group-diagram.png" alt-text="Diagram showing how a service group works in workload orchestration." lightbox="./media/service-group-diagram.png":::

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Set up the resources of workload orchestration](initial-setup-configuration.md#set-up-the-resources-of-workload-orchestration).

## Create a service group

The following command creates a service group with the specified name and tenant ID. Make sure to replace `<service-group-name>` and `<tenant-id>` with your actual values. Service group names must be unique within the tenant and can only contain alphanumeric characters, underscores, and hyphens. 

If your organization has multiple hierarchy levels, you need to create a service group at each level except for the target level. For more information, see [Service groups at different hierarchy levels](#service-groups-at-different-hierarchy-levels).

#### [Bash](#tab/bash)

```bash
sg="<service-group-name>"
tenantId="<tenant-id>"

az rest \
  --method put \
  --headers "Content-Type=application/json" \
  --url "https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$sg?api-version=2024-02-01-preview" \
  --body "{'properties':{'displayName':'$sg','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$tenantId'}}}" \
  --resource https://management.azure.com
```

#### [PowerShell](#tab/powershell)

```powershell
$sg = "<service-group-name>"
$tenantId = "<tenant-id>"

az rest `
    --method put `
    --header Content-Type=application/json `
    --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$sg`?api-version=2024-02-01-preview `
    --body "{'properties':{'displayName':'$sg','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$tenantId'}}}" `
    --resource https://management.azure.com
```

***

## Create and tag Sites 

Sites are used to identify the physical hierarchy such as plant, factory, and store. Sites can be created on top of subscriptions and resource groups. Site references are defined only for the **highest hierarchy level**. For example, if your hierarchy is *[Factory, Line]*, then you create a Site at the factory level. If your hierarchy is *[City, Factory, Line]*, then you create a Site at the city level.

To ensure that Sites appear appropriately in the Azure portal, make sure to tag the Sites with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup. 

For example, if the hierarchy is *[Factory, Line]*, then Site is created at the factory level and it should be tagged as \{`level`: `Factory`\}, where `level` is the label key and `Factory` is the label value.

#### [Bash](#tab/bash)

1. To tag the Site correctly, you can use the following commands, ensuring that the site is tagged according to its respective hierarchy level:

    ```bash
    # Tag a site
    az rest \
      --method put \
      --url "https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName?api-version=2025-03-01-preview" \
      --body "{'properties':{'displayName':'$sg','description': '$sg','labels': {'level': 'Factory'}}}" \
      --resource https://management.azure.com
    ```

1. If you have a Site previously created, to view the same on the workload orchestration portal, you need to patch the Site with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup.

    ```bash
    # Patch a site with correct labels
    az rest \
      --method patch \
      --url "https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName?api-version=2025-03-01-preview" \
      --body "{'properties':{'labels': {'<label-key>': '<label-value>'}}}" \
      --resource https://management.azure.com
    ```

1. Create configuration.

    ```bash
    configName="<configuration name>"
    configId="/subscriptions/$subscriptionId/resourceGroups/$resourcegroup/providers/microsoft.edge/configurations/$configName"
    az rest --method put --url "$configId?api-version=2025-08-01" --body "{'location':'$location'}"
    ```

1. Create the configuration reference.

    ```bash
    # For service group-based sites
    az rest --method put --url "$servicegroupId/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'$configId'}}"
    
    # For resource group-based sites
    az rest --method put --url "$siteId/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'$configId'}}"
    ```

1. Create the schema.

    ```bash
    schemaName="<schema name>"
    schemaId="/subscriptions/$subscriptionId/resourceGroups/$resourcegroup/providers/microsoft.edge/schemas/$schemaName"
    az rest --method put --url "$schemaId?api-version=2025-08-01" --body "{'location':'$location'}"
    ```

1. Create the schema reference.

    ```bash
    # For service group-based sites
    az rest --method put --url "$servicegroupId/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'$schemaId'}}"
    
    # For resource group-based sites
    az rest --method put --url "$siteId/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'$schemaId'}}"
    ```

#### [PowerShell](#tab/powershell)

1. To tag the Site correctly, you can use the following commands, ensuring that the Site is tagged according to its respective hierarchy level:

    ```powershell
    # Tag a site
    az rest `
      --method put `
      --url https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName`?api-version=2025-03-01-preview `
      --body "{'properties':{'displayName':'$sg','description': '$sg','labels': {'level': 'Factory'}}}" `
      --resource https://management.azure.com
    ```

1. If you have a Site previously created, to view the same on the workload orchestration portal, you need to patch the Site with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup.
    
    ```powershell
    # Patch a site with correct labels
    az rest `
      --method patch `
      --url https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName`?api-version=2025-03-01-preview `
      --body "{'properties':{'labels': {'level': 'factory'}}}" `
      --resource https://management.azure.com
    ```

1. Create configuration.

    ```powershell
    $configName="<configuration name>"
    $configId="/subscriptions/$subscriptionId/resourceGroups/$resourcegroup/providers/microsoft.edge/configurations/$configName"
    az rest --method put --url "$configId`?api-version=2025-08-01" --body "{'location':'$location'}"
    ```

1. Create the configuration reference.

    ```powershell
    # For service group-based sites
    az rest --method put --url "$servicegroupId/providers/microsoft.edge/configurationreferences/default`?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'$configId}}"
    
    # For resource group-based sites
    az rest --method put --url "$siteId/providers/microsoft.edge/configurationreferences/default`?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'$configId}}"
    ```

1. Create the schema.

    ```powershell
    $schemaName="<schema name>"
    $schemaId="/subscriptions/$subscriptionId/resourceGroups/$resourcegroup/providers/$microsoft.edge/schemas/$schemaName"
    az rest --method put --url "$schemaId`?api-version=2025-08-01" --body "{'location':'$location'}"
    ```

1. Create the schema reference.

    ```powershell
    # For service group-based sites
    az rest --method put --url "$servicegroupId/providers/microsoft.edge/schemareferences/default`?api-version=2025-08-01" --body "{'properties':{'schemaId':'$schemaId'}}"
    
    # For resource group-based sites
    az rest --method put --url "$siteId/providers/microsoft.edge/schemareferences/default`?api-version=2025-08-01" --body "{'properties':{'schemaId':'$schemaId'}}"
    ```

***

## Set up a service group hierarchy for workload orchestration

#### [Bash](#tab/bash)

1. Once the service group is created and the sites are appropriately tagged, you need to grant the workload orchestration access to the service group hierarchy. This is done by assigning the `Service Group Reader` role. 

    ```bash
    # Assign the Service Group Reader role
    providerAppId="cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    providerOid=$(az ad sp show --id "$providerAppId" --query "id" --output "tsv")

    az role assignment create --assignee "$providerOid" \
      --role "Service Group Reader" \
      --scope "/providers/Microsoft.Management/serviceGroups/$sg"
    ```

1. To connect a service group site to a context, you need to create a site reference. This is done by using the `az workload-orchestration context site-reference create` command. Make sure to replace the placeholders with your actual values.

    ```bash
    # Create a site reference
    az workload-orchestration context site-reference create \
      --subscription "$subId" \
      --resource-group "$rg" \
      --context-name "$instanceName" \
      --name "$siteReference" \
      --site-id "/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName"
    ```

1. Update *context-capabilities.json* file with the target capabilities you want to add to the context. It can only exist one context per tenant. For example: 

    ```json
    {
      "capabilities": [
        {
          "name": "soap",
          "description": "For soap production"
        },
        {
          "name": "shampoo",
          "description": "For shampoo production"
        }
      ]
    }
    ```

1. Once the context capabilities JSON file is updated, you can create a new context. Make sure to replace the placeholders with your actual values.

    ```bash
    # Create a new context
    az workload-orchestration context create \
      --subscription "$subId" \
      --resource-group "$rg" \
      --location "$l" \
      --name "Contoso-Context" \
      --capabilities "@context-capabilities.json" \
      --hierarchies "[0].name=factory" "[0].description=Factory" "[1].name=line" "[1].description=Line"
    ```

    > [!NOTE]
    > If you have a two-level hierarchy organization and you want to update the hierarchy levels to three or four levels, or vice versa, you can also use the `az workload-orchestration context create` to update the context with the new hierarchy levels. For more information, see [Service groups at different hierarchy levels](#service-groups-at-different-hierarchy-levels)

1. Update *custom-location.json* file with your custom location details.
1. Create a target. Make sure to update `solution-scope` value and `--capabilities` with the necessary values as per your scenario.

    ```bash
    # Create a target
    az workload-orchestration target create \
      --resource-group "$rg" \
      --location "$l" \
      --name "$childName" \
      --display-name "$childName" \
      --hierarchy-level "$level2" \
      --capabilities "[0].name=soap" "[0].description=For soap production" "[1].name=shampoo" "[1].description=For shampoo production" \
      --description "$childDesc" \
      --solution-scope "new" \
      --target-specification "@targetspecs.json" \
      --extended-location "@custom-location.json" \
      --context-id "/subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName"
    ```

1. Get the ID for Target created in the previous step.

    ```bash
    # Link the target ID to the factory service group
    targetId=$(az workload-orchestration target show --resource-group "$rg" --name "$childName" --query id --output tsv)
    ```

1. Link the target ID to the factory service group.

    ```bash
    az rest \
      --method put \
      --url "$targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview" \
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$sg'}}"
    ```

1. Update or refresh a target after connecting it to a service group to make sure hierarchy configuration is in sync with the service group. This step is optional.

    ```bash
    # Update or refresh a target
    az workload-orchestration target update --resource-group "$rg" --name "$childName"
    ```

#### [PowerShell](#tab/powershell)

1. Once the service group is created and the sites are appropriately tagged, you need to grant the workload orchestration access to the service group hierarchy. This is done by assigning the `Service Group Reader` role. 

    ```powershell
    # Assign the Service Group Reader role
    $providerAppId = "cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    $providerOid = $(az ad sp show --id $providerAppId --query id -o tsv)
    
    az role assignment create --assignee "$providerOid" `
        --role "Service Group Reader" `
        --scope "/providers/Microsoft.Management/serviceGroups/$sg"
    ```

1. To connect a service group site to a context, you need to create a site reference. This is done by using the `az workload-orchestration context site-reference create` command. Make sure to replace the placeholders with your actual values.

    ```powershell
    # Create a site reference
    az workload-orchestration context site-reference create `
       --subscription $subId `
       --resource-group $rg `
       --context-name $instanceName `
       --name $siteReference `
       --site-id "/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName"
    ```

1. Once the context capabilities JSON file is updated, you can create a new context.  Make sure to replace the placeholders with your actual values.

    ```powershell
    # Create a new context
    $context = $(az workload-orchestration context show --subscription $contextSubscriptionId --resource-group Contoso --name Contoso-Context) | ConvertFrom-JSON
    
    $context.properties.capabilities = $context.properties.capabilities + @(
       [PSCustomObject]@{description="shampoo"; name="shampoo"},
       [PSCustomObject]@{description="soap"; name="soap"}
    )
    $context.properties.capabilities = $context.properties.capabilities | Select-Object -Property name, description -Unique
    $context.properties.capabilities | ConvertTo-JSON -Compress | Set-Content context-capabilities.json
    
    az workload-orchestration context create `
      --subscription $subId `
      --resource-group $rg `
      --location $l `
      --name Contoso-Context `
      --capabilities "@context-capabilities.json" `
      --hierarchies [0].name=factory [0].description=Factory [1].name=line [1].description=Line
    ```

    > [!NOTE]
    > If you have a two-level hierarchy organization and you want to update the hierarchy levels to three or four levels, or vice versa, you can also use the `az workload-orchestration context create` to update the context with the new hierarchy levels. For more information, see [Service groups at different hierarchy levels](#service-groups-at-different-hierarchy-levels)

1. Update *custom-location.json* file with your custom location details.
1. Create a target. Make sure to update `solution-scope` value and `--capabilities` with the necessary values as per your scenario.

    ```powershell
    # Create a target
    az workload-orchestration target create `
      --resource-group $rg `
      --location $l `
      --name $childName `
      --display-name $childName `
      --hierarchy-level $level2 `
      --capabilities "[0].name=soap" "[0].description=For soap production" "[1].name=shampoo" "[1].description=For shampoo production" `
      --description $childDesc `
      --solution-scope "new" `
      --target-specification '@targetspecs.json' `
      --extended-location '@custom-location.json' `
      --context-id /subscriptions/$subId/resourceGroups/$rg/providers/Microsoft.Edge/contexts/$contextName
    ```

1. Get the ID for target created in the previous step.

    ```powershell
    $targetId = $(az workload-orchestration target show --resource-group $rg --name $childName --query id --output tsv)
    ```

1. Link the target ID to the factory service group.

    ```powershell
    az rest `
      --method put `
      --uri $targetId/providers/Microsoft.Relationships/serviceGroupMember/SGRelation?api-version=2023-09-01-preview `
      --body "{'properties':{ 'targetId': '/providers/Microsoft.Management/serviceGroups/$sg'}}"
    ```

1. Update or refresh a target after connecting it to a service group to make sure hierarchy configuration is in sync with the service group. This step is optional.

    ```powershell
    # Update or refresh a target
    az workload-orchestration target update --resource-group $sg --name $childName
    ```

***

Once the setup is completed, you can proceed with the solution authoring steps in [Solution authoring and deployment](workflow-features.md#solution-authoring-and-deployment).

> [!NOTE]
> If you run into any issues while creating service groups or configuring them, see the [Troubleshooting guide](troubleshooting.md#troubleshoot-service-groups).

## Service groups at different hierarchy levels

Service groups can be created for a **two-level** hierarchy organization, such as a factory and line, a **three-level** hierarchy organization, such as a city, factory, and line, and a maximum of **four-level** hierarchy organization, such as a region, city, factory, and line. The hierarchy names can be customized to match your organizational structure.

The previous sections show how to create a service group for a two-level hierarchy organization, which you can use as a reference to create service groups for a three-level or four-level hierarchy organization.

To ease the process, the following steps show how to create a four-level service group hierarchy organization with levels: region, city, factory, and line. You need to consider the following points:

- **The top three levels** in the hierarchy must each have its own **service group** created. For example, for the four-level hierarchy organization, you need to create a service group for levels region, city, and factory.
- **Site reference** is defined at the **highest level**. Although the context has 4 levels, if the site reference is defined at city level, then the particular site will have only 3 levels: city, factory, and line.  If the site reference is at factory level, then the particular site will have only 2 levels: factory and line. 
- The **`editable_at`** field in the [configuration schema](configuring-schema.md) only **accepts the parent levels** in addition to target level. For example, if the solution is to be deployed at factory level, then the `editable_at` field in the schema only accepts the region, city, and factory levels. If the solution is to be deployed at city level, then the `editable_at` field in the schema accepts only region and city levels.

### [Bash](#tab/bash)

1. Define the global variables.

    ```bash
    rg="<resource-group-name>"
    tenantId="<tenant-id>"
    l="<Azure region>"
    subscriptionId="<subscription ID>"
    resourcePrefix="<prefix for resource name>"
    ```

1. Define the service group names and hierarchy levels. 

    ```bash
    ## Level 1 / Region
    # Create Top / Level 1 Service Group $resourcePrefix-SGRegion to link resources to:
    az rest \
        --method put \
        --header Content-Type=application/json \
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion?api-version=2024-02-01-preview \
        --body "{'properties':{'displayName':'$resourcePrefix-SGRegion','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$tenantId'}}}" \
        --resource https://management.azure.com

    # Create Top / Level 1 Site $resourcePrefix-SGRegion to visualize & store configuration onto:
    az rest \
      --method put \
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/Microsoft.Edge/sites/$resourcePrefix-SGRegion?api-version=2025-03-01-preview \
      --body "{'properties':{'displayName':'$resourcePrefix-SGRegion','description': '$resourcePrefix-SGRegion','labels': {'level': 'Region'}}}" \
      --resource https://management.azure.com
    
    siteName="$resourcePrefix-SGRegion"
    siteName="${siteName:0:18}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema'}}"
    
    ## Level 2 / City
    # Create Level 2 Service Group $resourcePrefix-SGCity to link resources to:
    az rest \
        --method put \
        --header Content-Type=application/json \
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity?api-version=2024-02-01-preview \
        --body "{'properties':{'displayName':'$resourcePrefix-SGCity','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion'}}}" \
        --resource https://management.azure.com
    
    # Create Level 2 Site $resourcePrefix-SGCity to visualize & store configuration onto:
    az rest \
      --method put \
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/Microsoft.Edge/sites/$resourcePrefix-SGCity?api-version=2025-03-01-preview \
      --body "{'properties':{'displayName':'$resourcePrefix-SGCity','description': '$resourcePrefix-SGCity','labels': {'level': 'City'}}}" \
      --resource https://management.azure.com
    
    siteName="$resourcePrefix-SGCity"
    siteName="${siteName:0:18}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema'}}"
    
    ## Level 3 / Factory
    # Create Level 3 Service Group $resourcePrefix-SGFactory to link resources to:
    az rest \
        --method put \
        --header Content-Type=application/json \
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory?api-version=2024-02-01-preview \
        --body "{'properties':{'displayName':'$resourcePrefix-SGFactory','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity'}}}" \
        --resource https://management.azure.com
    
    # Create Level 3 / Factory Site $resourcePrefix-SGFactory to visualize & store configuration onto:
    az rest \
      --method put \
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/Microsoft.Edge/sites/$resourcePrefix-SGFactory?api-version=2025-03-01-preview \
      --body "{'properties':{'displayName':'$resourcePrefix-SGFactory','description': '$resourcePrefix-SGFactory','labels': {'level': 'Factory'}}}" \
      --resource https://management.azure.com        
    
    siteName="$resourcePrefix-SGFactory"
    siteName="${siteName:0:18}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/${siteName}Config'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/${siteName}Schema'}}"
    ```

1. Once the service groups are created, you need to grant access to the workload orchestration service. This is done by assigning the `Service Group Reader` role to the workload orchestration provider app ID.

    ```bash
    providerAppId="cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    providerOid=$(az ad sp show --id "$providerAppId" --query "id" --output "tsv")

    az role assignment create --assignee "$providerOid" \
      --role "Service Group Reader" \
      --scope "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion"
    ```

1. To connect a service group site to a context, you need to create a site reference.

    ```bash
    contextSubscriptionId="<subscription-id used to create the context for the first time>"
    contextRG="<resource group used to create the context for the first time>"
    # Enter the name of the context used during creation
    contextName="Contoso-Context"
    siteReference="Region-SG"
    # Enter your specific region
    l="eastus"

    az workload-orchestration context site-reference create \
      --subscription "$contextSubscriptionId" \
      --resource-group "$contextRG" \
      --context-name "$contextName" \
      --name "$siteReference" \
      --site-id "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/Microsoft.Edge/sites/$level1Name"
    ```

1. Update *context-capabilities.json* file with the target capabilities you want to add to the context.
1. Create a new context or use an already existing one. The example uses region, city, factory, and line as hierarchies. Make sure to replace the placeholders and the hierarchy values with your actual values.

    ```bash
    # Only one context can exist per tenant.
    # Enter your capabilities. These are tags which help relate applications and targets

    capability1="soap"
    capability2="shampoo"

    # Create the capabilities JSON file
    cat <<EOF > context-capabilities.json
    {
      "capabilities": [
        {
          "name": "$capability1",
          "description": "For $capability1 production"
        },
        {
          "name": "$capability2",
          "description": "For $capability2 production"
        }
      ]
    }
    EOF

    az workload-orchestration context create \
      --subscription "$contextSubscriptionId" \
      --resource-group "$contextRG" \
      --location "$l" \
      --name "$contextName" \
      --capabilities "@context-capabilities.json" \
      --hierarchies "[0].name=region" "[0].description=Region" "[1].name=city" "[1].description=City" "[2].name=factory" "[2].description=Factory" "[3].name=line" "[3].description=Line"
    ```

### [PowerShell](#tab/powershell)

1. Define the global variables.

    ```powershell
    $rg="<resource-group-name>"
    $tenantId="<tenant-id>"
    $l="<Azure region>"
    $subscriptionId="<subscription ID>"
    $resourcePrefix="<prefix for resource name>"
    ```

1. Define the service group names and hierarchy levels. 

    ```powershell
    ## Level 1 / Region
    # Create Top / Level 1 Service Group $resourcePrefix-SGRegion to link resources to:
    az rest `
        --method put `
        --header Content-Type=application/json `
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion?api-version=2024-02-01-preview `
        --body "{'properties':{'displayName':'$resourcePrefix-SGRegion','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$tenantId'}}}" `
        --resource https://management.azure.com

    # Create Top / Level 1 Site $resourcePrefix-SGRegion to visualize & store configuration onto:
    az rest `
      --method put `
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/Microsoft.Edge/sites/$resourcePrefix-SGRegion?api-version=2025-03-01-preview `
      --body "{'properties':{'displayName':'$resourcePrefix-SGRegion','description': '$resourcePrefix-SGRegion','labels': {'level': 'Region'}}}" `
      --resource https://management.azure.com

    $siteName = "$resourcePrefix-SGRegion"
    $siteName = $siteName.Substring(0, [math]::Min(18, $siteName.Length))
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")'}}"

    ## Level 2 / City
    # Create Level 2 Service Group $resourcePrefix-SGCity to link resources to:
    az rest `
        --method put `
        --header Content-Type=application/json `
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity?api-version=2024-02-01-preview `
        --body "{'properties':{'displayName':'$resourcePrefix-SGCity','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGRegion'}}}" `
        --resource https://management.azure.com

    # Create Level 2 Site $resourcePrefix-SGCity to visualize & store configuration onto:
    az rest `
      --method put `
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/Microsoft.Edge/sites/$resourcePrefix-SGCity?api-version=2025-03-01-preview `
      --body "{'properties':{'displayName':'$resourcePrefix-SGCity','description': '$resourcePrefix-SGCity','labels': {'level': 'City'}}}" `
      --resource https://management.azure.com

    $siteName = "$resourcePrefix-SGCity"
    $siteName = $siteName.Substring(0, [math]::Min(18, $siteName.Length))
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")'}}"

    ## Level 3 / Factory
    # Create Level 3 Service Group $resourcePrefix-SGFactory to link resources to:
    az rest `
        --method put `
        --header Content-Type=application/json `
        --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory?api-version=2024-02-01-preview `
        --body "{'properties':{'displayName':'$resourcePrefix-SGFactory','parent': { 'resourceId': '/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGCity'}}}" `
        --resource https://management.azure.com
    
    # Create Level 3 / Factory Site $resourcePrefix-SGFactory to visualize & store configuration onto:
    az rest `
      --method put `
      --url https://eastus2euap.management.azure.com/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/Microsoft.Edge/sites/$resourcePrefix-SGFactory?api-version=2025-03-01-preview `
      --body "{'properties':{'displayName':'$resourcePrefix-SGFactory','description': '$resourcePrefix-SGFactory','labels': {'level': 'Factory'}}}" `
      --resource https://management.azure.com        
    
    $siteName = "$resourcePrefix-SGFactory"
    $siteName = $siteName.Substring(0, [math]::Min(18, $siteName.Length))
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")?api-version=2025-08-01" --body "{'location':'$l'}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/microsoft.edge/configurationreferences/default?api-version=2025-08-01" --body "{'properties':{'configurationResourceId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/configurations/$($siteName + "Config")'}}"
    az rest --method put --url "/providers/Microsoft.Management/serviceGroups/$resourcePrefix-SGFactory/providers/microsoft.edge/schemareferences/default?api-version=2025-08-01" --body "{'properties':{'schemaId':'/subscriptions/$subscriptionId/resourceGroups/$rg/providers/microsoft.edge/schemas/$($siteName + "Schema")'}}"
    ```

1. Once the service groups are created, you need to grant access to the workload orchestration service. This is done by assigning the `Service Group Reader` role to the workload orchestration provider app ID.

    ```powershell
    $providerAppId = "cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    $providerOid = $(az ad sp show --id $providerAppId --query id -o tsv)
    
    az role assignment create --assignee "$providerOid" `
        --role "Service Group Reader" `
        --scope "/providers/Microsoft.Management/serviceGroups/$level1Name"
    ```

1. To connect a service group site to a context, you need to create a site reference.

    ```powershell
    $contextSubscriptionId = "<subscription-id used to create the context for the first time>"
    $contextRG = "<resource group used to create the context for the first time>"
    #Enter the name of the context used during creation
    $contextName = "Contoso-Context"
    $siteReference = "Region-SG"
    #Enter your specific region
    $l = "eastus"
    
    
    az workload-orchestration context site-reference create --subscription $contextSubscriptionId --resource-group $contextRG --context-name $contextName --name $siteReference --site-id "/providers/Microsoft.Management/serviceGroups/$level1Name/providers/Microsoft.Edge/sites/$level1Name"
    ```

1. Update *context-capabilities.json* file with the target capabilities you want to add to the context.
1. Create a new context or use an already existing one. The example uses region, city, factory, and line as hierarchies. Make sure to replace the placeholders and the hierarchy values with your actual values.

    ```powershell
    #Context can only exist one per tenant.
    #Enter your capabilities. These are tags which helps relate applications and targets
    
    $capability1 = "soap"
    $capability2 = "shampoo"
    
    $context = $(az workload-orchestration context show --subscription $contextSubscriptionId --resource-group $contextRG --name $contextName) | ConvertFrom-JSON
    
    $context.properties.capabilities = $context.properties.capabilities + @(
       [PSCustomObject]@{description="$capability1"; name="$capability2"},
       [PSCustomObject]@{description="$capability1"; name="$capability2"}
    )
    $context.properties.capabilities = $context.properties.capabilities | Select-Object -Property name, description -Unique
    $context.properties.capabilities | ConvertTo-JSON -Compress | Set-Content context-capabilities.json
    
    az workload-orchestration context create --subscription $contextSubscriptionId --resource-group $contextRG --location $l --name $contextName --capabilities "@context-capabilities.json" --hierarchies [0].name=region [0].description=Region [1].name=city [1].description=City [2].name=factory [2].description=Factory [3].name=line [3].description=Line
    ```
***

> [!NOTE]
> Once the setup is completed, the hierarchy level is displayed in workload orchestration in Azure portal. For more information, see [Monitor your solutions with Azure portal](azure-portal-monitoring.md).

For more details, see the following tutorials on how to create solutions with different targets in a four-level hierarchy organization:

- [Create a solution with a leaf target](tutorial-service-group-scenario-1.md): This tutorial shows how to create a solution with a target at line level in a four-level hierarchy organization.
- [Create a solution with a non-leaf target](tutorial-service-group-scenario-2.md): This tutorial shows how to create a solution with a target at factory level in a four-level hierarchy organization.
- [Create a solution with multiple dependencies at different levels](tutorial-service-group-scenario-3.md): This tutorial shows how to create a solution with multiple shared adapter dependencies at different levels in a four-level hierarchy organization.
- [Create multiple solutions with a single dependency at different levels](tutorial-service-group-scenario-4.md): This tutorial shows how to create multiple solutions with a single shared adapter dependency at different levels in a four-level hierarchy organization.

## Related content

- [Prepare your environment for workload orchestration](initial-setup-environment.md)
- [Configuration template](configuring-template.md)
- [Configuration schema](configuring-schema.md)
- [RBAC guide](rbac-guide.md)
