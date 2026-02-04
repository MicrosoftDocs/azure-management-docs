---
title: Zone redundancy in Azure Container Registry
description: Learn how to create a zone-redundant registry in Azure Container Registry
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 02/04/2026
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry

# Customer intent: As a cloud architect, I want to configure zone redundancy for Azure Container Registry, so that I can ensure high availability and resiliency for container images within a specific region.
---

# Zone redundancy in Azure Container Registry

Zone redundancy is enabled by default for all Azure Container Registries in regions that support availability zones, making your resources more resilient automatically and at no additional cost. This enhancement applies to all SKUs, including Basic and Standard, and has been rolled out to both new and existing registries in supported regions. For Premium registries that use geo-replication, all replicas in supported regions are also zone redundant by default.

For more information about availability zone support requirements and features, as well as multi-region deployment options, see [Reliability in Azure Container Registry](/azure/reliability/reliability-container-registry).

> [!IMPORTANT]
> The Azure portal and other tooling might not yet reflect the zone redundancy update accurately. The `zoneRedundancy` property in your registry’s configuration might still show as false, but zone redundancy is active for all registries in supported regions. We're actively updating the portal and API surfaces to reflect this default behavior more transparently. All previously enabled features continue to function as expected.

This article describes how to create zone-redundant registries and geo-replicas in Azure Container Registry.

## Create a zone-redundant registry in Azure Container Registry

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

1. Make sure that you have the latest version of Azure CLI. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

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

## Create a zone-redundant geo-replica

You can set up a zone-redundant replica in an Azure region separate to your registry's home region.

[Geo-replication](/azure/reliability/reliability-container-registry#multi-region-support) in the [Premium tier of Azure Container Registry](container-registry-skus.md) replicates your container registry's contents to multiple Azure regions. If your Premium registry uses geo-replication, your replicas will also be zone redundant when the replica is provisioned in a region that supports availability zones.

Zone redundancy is enabled by default for all Azure Container Registry replicas in regions that support availability zones, making your resources more resilient automatically and at no additional cost. This enhancement to both new and existing replicas in supported regions.

 Follow the steps below to create a zone-redundant replica for a container registry that uses the Premium service tier. If you don't have one already, follow the steps in [Create a zone-redundant registry in Azure Container Registry](./zone-redundancy.md).

To create a zone-redundant replica, use Azure portal, Azure CLI, or a Bicep file.

### [Azure portal](#tab/portal)

1. Sign in to the [Azure portal](https://portal.azure.com).

1. Go to your Premium tier container registry, and select **Geo-replications**.

1. On the map that appears, you do one of the following:

    - Select a green hexagon in a region that [supports availability zones](/azure/reliability/regions-list), such as **West US 2**.

    - Select **+ Add**.

1. In the **Create replication** window, confirm the **Location**.

    In **Availability zones**, select **Enabled**, and then select **Create**.

    :::image type="content" source="media/zone-redundancy-replica/enable-availability-zones-replication-portal.png" alt-text="Enable zone-redundant replication in Azure portal":::

### [Azure CLI](#tab/cli)

1. Make sure that you have the latest version of Azure CLI. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

1. Create zone-redundant replication by running [az acr replication create](/cli/azure/acr/replication#az-acr-replication-create) (replace `<resource-group-name>`, `<container-registry-name>`, and `<replica-region>` with your values):

    ```azurecli
    az acr replication create \
      --location <region-name> \
      --resource-group <resource-group-name> \
      --registry <container-registry-name> \
      --zone-redundancy enabled
    ```
 
1. In the command output, note the `zoneRedundancy` property for the replica. When `zoneRedundancy` is set to `"Enabled"`, the registry is zone redundant:

    ```JSON
    {
      [...]
      "zoneRedundancy": "Enabled"
    }
    ```

### [Bicep](#tab/bicep)

1. To create a geo-replica for your existing registry, copy the following Bicep template to a new file and save it using a filename such as `replicaZone.bicep`. 

    By default, the template enables zone redundancy the regional replica.

    ```bicep
    @description('Globally unique name of your Azure Container Registry')
    param containerRegistryName string

    @description('Short name for registry replica location, such as australiaeast or westus.')
    param replicaLocation string

    @description('Enable zone redundancy of registry replica. Requires replica location to support availability zones.')
    @allowed([
      'Enabled'
      'Disabled'
    ])
    param replicaZoneRedundancy string = 'Enabled'

    resource containerRegistry 'Microsoft.ContainerRegistry/registries@2025-04-01' existing = {
      name: containerRegistryName
    }

    resource containerRegistryReplica 'Microsoft.ContainerRegistry/registries/replications@2025-04-01' = {
      parent: containerRegistry
      name: replicaLocation
      location: replicaLocation
      properties: {
        zoneRedundancy: replicaZoneRedundancy
      }
    }
    ```
    
1. Run the following [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create) command to create the registry using the preceding template file (replace `<resource-group-name>`, `<registry-name>`, and `<replica-location>` with your values). 

    ```azurecli
    az deployment group create \
      --resource-group <resource-group-name> \
      --template-file registryZone.json \
      --parameters containerRegistryName=<registry-name> replicaLocation=<replica-location>
    ```

---
