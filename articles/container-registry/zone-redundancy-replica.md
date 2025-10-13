---
title: Create a zone-redundant registry and replica in Azure Container Registry
description: Learn how to create a zone-redundant registry and replica in Azure Container Registry
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 07/24/2025
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry

# Customer intent: As a cloud architect, I want to configure zone redundancy for my Azure Container Registry geo-replica, so that I can ensure high availability and resiliency for container images within a specific region.
---

# Create a zone-redundant replica in Azure Container Registry

This article describes how to set up a zone-redundant replica in an Azure region separate to your registry's home region.

[Geo-replication](/azure/reliability/reliability-container-registry#multi-region-support) in the [Premium tier of Azure Container Registry](container-registry-skus.md) replicates your container registry's contents to multiple Azure regions.

When your registry uses geo-replication, your replicas will also be zone redundant when the replica is provisioned in an Availability Zone enabled region. Zone redundancy allows for the distributing of registry data and operations across multiple availability zones within the region.

Zone redundancy is enabled by default for all Azure Container Registry replicas in regions that support availability zones, making your resources more resilient automatically and at no additional cost. This enhancement to both new and existing replicas in supported regions.

>[!IMPORTANT]
>The Azure portal and CLI may not yet reflect the zone redundancy update accurately. The `zoneRedundancy` property in your replica’s configuration might still show as false even though zone redundancy is active for all replicas in supported regions. We’re actively updating the portal and API surfaces to reflect this default behavior more transparently. All previously enabled features will continue to function as expected.

For more information about availability zone support requirements and features, as well as multi-region deployment options, see [Reliability in Azure Container Registry](/azure/reliability/reliability-container-registry).

## Prerequisites

- A container registry that uses the Premium service tier. If you don't have one already, follow the steps in [Create a zone-redundant registry in Azure Container Registry](./zone-redundancy.md).

- Select a region for your replica that [supports availability zones](/azure/reliability/regions-list), such as *eastus*.

## Create a zone-redundant replica

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

1. Make sure that you have Azure CLI version 2.17.0 or later, or Azure Cloud Shell. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

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
