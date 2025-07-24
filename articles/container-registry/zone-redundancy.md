---
title: Create a zone-redundant registry in Azure Container Registry
description: Learn how to create a zone-redundant registry in Azure Container Registry
ms.topic: how-to
author: chasedmicrosoft
ms.author: doveychase
ms.date: 07/24/2025
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry

# Customer intent: As a cloud architect, I want to configure zone redundancy for Azure Container Registry, so that I can ensure high availability and resiliency for container images within a specific region.
---

# Create a zone-redundant registry in Azure Container Registry

This article describes how to set up a zone-redundant container registry.

Zone redundancy in the [Premium tier of Azure Container Registry](container-registry-skus.md) provides protection against single zone failures. Zone redundancy allows for the distributing of registry data and operations across multiple availability zones within the region.

For more information about availability zone support requirements and features, as well as multi-region deployment options, see [Reliability in Azure Container Registry](/azure/reliability/reliability-container-registry).

## Prerequisites

- An [Azure subscription](https://azure.microsoft.com/free/).

- Select a region that [supports availability zones](/azure/reliability/regions-list), such as *eastus*.

- You must use the [Premium service tier](container-registry-skus.md) to enable zone redundancy in Azure Container Registry.

## Create a zone-redundant registry

To create a zone-redundant registry in the Premium service tier, use Azure portal, Azure CLI, or a Bicep file.

### [Azure portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Select **Create a resource** > **Containers** > **Container Registry**.

1. In the **Basics** tab, select or create a resource group, and enter a unique registry name.

1. In **Location**, select a region that [supports availability zones](/azure/reliability/regions-list), such as *East US*.

1. In **SKU**, select **Premium**.

1. In **Availability zones**, select **Enabled**.

1. Optionally, configure more registry settings, and then select **Review + create**.

1. Select **Create** to deploy the registry instance.

    :::image type="content" source="media/zone-redundancy/enable-availability-zones-portal.png" alt-text="Enable zone redundancy in Azure portal.":::


### [Azure CLI](#tab/cli)

1. Make sure that you have Azure CLI version 2.17.0 or later, or Azure Cloud Shell. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

1. If you don't have a resource group in a region that supports availability zones, run [az group create](/cli/azure/group#az-group-create) to create a resource group (replace `<resource-group-name>` and `<location>` with your values):

    ```azurecli
    az group create --name <resource-group-name> --location <location>
    ```

1. Select a region that [supports availability zones](/azure/reliability/regions-list), such as *eastus*.

1. Create a zone-enabled registry in the Premium service tier by running the [az acr create](/cli/azure/acr#az-acr-create) command (replace `<resource-group-name>`, `<container-registry-name>`, and `<region-name>` with your values):

    ```azurecli
    az acr create \
      --resource-group <resource-group-name> \
      --name <container-registry-name> \
      --location <region-name> \
      --zone-redundancy enabled \
      --sku Premium
    ```

1. In the command output, note the `zoneRedundancy` property for the registry. When `zoneRedundancy` is set to `"Enabled"`, the registry is zone redundant:
    
    ```JSON
    {
      [...]
      "zoneRedundancy": "Enabled"
    }
    ```

### [Bicep](#tab/biep)

1. If you don't have a resource group in a region that supports availability zones, run [az group create](/cli/azure/group#az-group-create) to create a resource group (replace `<resource-group-name>` and `<location>` with your values):

    ```azurecli
    az group create --name <resource-group-name> --location <location>
    ```

1. To create a zone-redundant registry, copy the following Bicep file to a new file and save it using a filename such as `registryZone.bicep`. 

    By default, the Bicep file enables zone redundancy in the registry.

    ```bicep
    @description('Globally unique name of your Azure Container Registry')
    @minLength(5)
    @maxLength(50)
    param containerRegistryName string = 'acr${uniqueString(resourceGroup().id)}'

    @description('Location for registry home replica.')
    param location string = resourceGroup().location

    @description('Enable admin user for registry. This is not recommended for production use.')
    param adminUserEnabled bool = false

    @description('Enable zone redundancy of registry\'s home replica. Requires the registry\'s region supports availability zones.')
    @allowed([
      'Enabled'
      'Disabled'
    ])
    param containerRegistryZoneRedundancy string = 'Enabled'

    // Tier of your Azure Container Registry. Geo-replication and zone redundancy require Premium SKU.
    var acrSku = 'Premium'

    resource containerRegistry 'Microsoft.ContainerRegistry/registries@2025-04-01' = {
      name: containerRegistryName
      location: location
      sku: {
        name: acrSku
      }
      properties: {
        adminUserEnabled: adminUserEnabled
        zoneRedundancy: containerRegistryZoneRedundancy
      }
    }

    output containerRegistryLoginServer string = containerRegistry.properties.loginServer
    ```
    
1. Run the following [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create) command to create the registry using the preceding template file (replace `<resource-group-name>` and `<registry-name>` with your values). 

     >[!NOTE]
     > If you deploy the template without parameters, it creates a unique name for you.

    
    ```azurecli
    az deployment group create \
      --resource-group <resource-group-name> \
      --template-file registryZone.json \
      --parameters containerRegistryName=<registry-name> 
    ```

---
