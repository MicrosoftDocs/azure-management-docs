---
title: Service Groups for Workload Orchestration
description: Learn about service groups and how to configure them in workload orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 05/07/2025
# Customer intent: As a cloud administrator, I want to configure service groups for workload orchestration, so that I can efficiently organize and manage resources across multiple subscriptions and apply governance effectively.
---

# Service groups for workload orchestration

Service groups are a new resource type in Azure Resource Manager (ARM) that help you organize related resources, like resource groups, subscriptions, and management groups, under one service, application, or workload. This article explains how to create a service group and configure it to use it with workload orchestration.

For more information, see [RBAC for service groups](rbac-guide.md#rbac-for-service-groups).

## What is a service group?

Service groups are tenant-level resource containers that represent a subset of collection of resources across Azure subscriptions or resource groups. They allow you to organize selected resources into a unified logical grouping, while maintaining your existing setup. 

Service groups also allow you to create layers (self-nesting) so you can organize services in a way that matches your real-world structure. This setup gives you a consistent view of your services, making it easier to manage visibility, tagging, policy enforcement, and governance, no matter how your resources are actually organized.

Every service group has a display name and one parent service group. Relationships are created between resources to establish a hierarchical structure such as parent and child associations.

The following diagram shows how service groups structure groups related components and deployment targets into logical units, making the architecture more modular and easier to manage.

:::image type="content" source="./media/service-group-diagram.png" alt-text="Diagram showing how a service group works in workload orchestration." lightbox="./media/service-group-diagram.png":::

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/).
- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.
- Download and extract the artifacts from the [GitHub repository](https://github.com/microsoft/AEP/blob/main/content/en/docs/Configuration%20Manager%20(Public%20Preview)/Scripts%20for%20Onboarding/Configuration%20manager%20files.zip) into a particular folder. 

> [!NOTE]
> You can reuse the global variables defined in [Prepare the basics to run workload orchestration](initial-setup-environment.md#prepare-the-basics-to-run-workload-orchestration) and the resource variables defined in [Configure the resources of workload orchestration](initial-setup-configuration.md#configure-the-resources-of-workload-orchestration).

## Create a service group

The following command creates a service group with the specified name and tenant ID. Make sure to replace `<service-group-name>` and `<tenant-id>` with your actual values.

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

Sites and Site addresses are used to identify the physical hierarchy such as plant, factory, and store. Sites can be created on top of subscriptions and resource groups.

To ensure that Sites appear appropriately in the Azure portal, make sure to tag the Sites with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup.

For example, if the hierarchy is *[Factory, Line]*, when a Site is created at the Factory level, it should be tagged as 
\{`level`: `Factory`\}, where `level` is the label key and `Factory` is the label value.

#### [Bash](#tab/bash)

To tag the Site correctly, you can use the following commands, ensuring that the site is tagged according to its respective hierarchy level:

```bash
# Tag a site
az rest \
  --method put \
  --url "https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName?api-version=2025-03-01-preview" \
  --body "{'properties':{'displayName':'$sg','description': '$sg','labels': {'level': 'Factory'}}}" \
  --resource https://management.azure.com
```

If you have a Site previously created, to view the same on the workload orchestration portal, you need to patch the site with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup.

```bash
# Patch a site with correct labels
az rest \
  --method patch \
  --url "https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName?api-version=2025-03-01-preview" \
  --body "{'properties':{'labels': {'<label-key>': '<label-value>'}}}" \
  --resource https://management.azure.com
```

#### [PowerShell](#tab/powershell)

To tag the Site correctly, you can use the following commands, ensuring that the Site is tagged according to its respective hierarchy level:

```bash
# Tag a site
az rest `
  --method put `
  --url https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName`?api-version=2025-03-01-preview `
  --body "{'properties':{'displayName':'$sg','description': '$sg','labels': {'level': 'Factory'}}}" `
  --resource https://management.azure.com
```

If you have a Site previously created, to view the same on the workload orchestration portal, you need to patch the Site with the correct labels. The labels should be set according to the Site’s hierarchy level, as defined in your workload orchestration setup.

```bash
# Patch a site with correct labels
az rest `
  --method patch `
  --url https://management.azure.com/providers/Microsoft.Management/serviceGroups/$sg/providers/Microsoft.Edge/sites/$siteName`?api-version=2025-03-01-preview `
  --body "{'properties':{'labels': {'level': 'factory'}}}" `
  --resource https://management.azure.com
```

***

## Set up a service group hierarchy for workload orchestration

#### [Bash](#tab/bash)

1. Once the service group is created and the sites are appropriately tagged, you need to grant the workload orchestration access to the service group hierarchy. This is done by assigning the `Service Group Contributor` role. 

    ```bash
    # Assign the Service Group Contributor role
    providerAppId="cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    providerOid=$(az ad sp show --id "$providerAppId" --query "id" --output "tsv")

    az role assignment create --assignee "$providerOid" \
      --role "Service Group Contributor" \
      --scope "/providers/Microsoft.Management/serviceGroups/$sg"
    ```

1. To connect a service group site to a context, you need to create a site reference. This is done by using the `az workload-orchestration context site-reference create` command. Make sure to replace the placeholders with your actual values.

    ```bash
    # Create a site reference
    az workload-orchestration context site-reference create \
      --subscription "$subscriptionId" \
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
      --subscription "$subscriptionId" \
      --resource-group "$rg" \
      --location "$l" \
      --name "Contoso-Context" \
      --capabilities "@context-capabilities.json" \
      --hierarchies "[0].name=factory" "[0].description=Factory" "[1].name=line" "[1].description=Line"
    ```

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
      --extended-location "@custom-location.json"
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

1. Once the service group is created and the sites are appropriately tagged, you need to grant the workload orchestration access to the service group hierarchy. This is done by assigning the `Service Group Contributor` role. 

    ```powershell
    # Assign the Service Group Contributor role
    $providerAppId = "cba491bc-48c0-44a6-a6c7-23362a7f54a9" # Workload orchestration Provider App ID
    $providerOid = $(az ad sp show --id $providerAppId --query id -o tsv)
    
    az role assignment create --assignee "$providerOid" `
        --role "Service Group Contributor" `
        --scope "/providers/Microsoft.Management/serviceGroups/$sg"
    ```

1. To connect a service group site to a context, you need to create a site reference. This is done by using the `az workload-orchestration context site-reference create` command. Make sure to replace the placeholders with your actual values.

    ```powershell
    # Create a site reference
    az workload-orchestration context site-reference create `
       --subscription $subscriptionId `
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
      --subscription $subscriptionId `
      --resource-group $rg `
      --location $l `
      --name Contoso-Context `
      --capabilities "@context-capabilities.json" `
      --hierarchies [0].name=factory [0].description=Factory [1].name=line [1].description=Line
    ```

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
      --extended-location '@custom-location.json'
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

Once the setup is completed, you can proceed with the solution authoring steps in [Solution authoring and deployment](initial-setup-configuration.md#solution-authoring-and-deployment).

## Related content

- [Prepare your environment for workload orchestration](initial-setup-environment.md)
- [Configuration template](configuring-template.md)
- [Configuration schema](configuring-schema.md)
- [RBAC guide](rbac-guide.md)
