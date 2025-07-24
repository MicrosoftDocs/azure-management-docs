---
title: Create a zone-redundant registry and replica in Azure Container Registry
description: Learn how to create a zone-redundant registry and replica in Azure Container Registry
ms.topic: how-to
author: chasedmicrosoft
ms.author: doveychase
ms.date: 07/24/2025
ms.custom: references_regions, devx-track-azurecli
ms.service: azure-container-registry

# Customer intent: As a cloud architect, I want to configure zone redundancy for Azure Container Registry, so that I can ensure high availability and resiliency for container images within a specific region.
---

# Create a zone-redundant registry and replica in Azure Container Registry

This article shows how to set up a zone-redundant container registry and replica by using the Azure CLI, Azure portal, or Azure Resource Manager template. 

Zone redundancy in the [Premium tier of Azure Container Registry](container-registry-skus.md) provides protection against single zone failures. Zone redundancy allows for the distributing of registry data and operations across multiple availability zones within the region. In addition, if your registry uses [geo-replication](/azure/reliability/reliability-container-registry#multi-region-support) and zone redundancy together, you can  configure zone redundancy on each regional replica.  


## Prerequisites
- An [Azure subscription](https://azure.microsoft.com/free/).

- Select a region that [supports availability zones](/azure/reliability/region-list), such as *eastus*.

- You must use the [Premium service tier](container-registry-skus.md) to enable zone redundancy in Azure Container Registry.

For more information about availability zone support requirements and features -  as well as multi-region deployment options, see [Reliability in Azure Container Registry](/azure/reliability/reliability-container-registry).

## Create a zone-redundant registry and replica

To create a zone-redundant registry and replica in the Premium service tier, use Azure portal, Azure CLI, or an Azure Resource Manager template. 

### [Azure portal](#tab/portal)


1. Sign in to the [Azure portal](https://portal.azure.com).

1. Select **Create a resource** > **Containers** > **Container Registry**.

1. In the **Basics** tab, select or create a resource group, and enter a unique registry name.

1. In **Location**, select a region that [supports availability zones](/azure/reliability/region-list), such as *East US*.

1. In **SKU**, select **Premium**.

1. In **Availability zones**, select **Enabled**.

1. Optionally, configure more registry settings, and then select **Review + create**.

1. Select **Create** to deploy the registry instance.

    :::image type="content" source="media/zone-redundancy/enable-availability-zones-portal.png" alt-text="Enable zone redundancy in Azure portal":::

1. To create a zone-redundant replication:

    1. Go to your Premium tier container registry, and select **Replications**.

    1. On the map that appears, you do one of the following:
       - Select a green hexagon in a region that [supports availability zones](/azure/reliability/region-list), such as **West US 2**.

       - Select **+ Add**.

    1. In the **Create replication** window, confirm the **Location**. In **Availability zones**, select **Enabled**, and then select **Create**.

    :::image type="content" source="media/zone-redundancy/enable-availability-zones-replication-portal.png" alt-text="Enable zone-redundant replication in Azure portal":::


### [Azure CLI](#tab/cli)

1. Make sure that you have Azure CLI version 2.17.0 or later, or Azure Cloud Shell. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

1. If you don't have a resource group in a region that supports availability zones, run [az group create](/cli/azure/group#az-group-create) to create a resource group (replace `<resource-group-name>` and `<location>` with your values):

    ```azurecli
    az group create --name <resource-group-name> --location <location>
    ```
1. Select a region that [supports availability zones](/azure/reliability/region-list), such as *eastus*.

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
    "zoneRedundancy": "Enabled",
    }
    ```

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
    "zoneRedundancy": "Enabled",
    }
    ```

### [ARM template](#tab/arm-template)

1. If you don't have a resource group in a region that supports availability zones, run [az group create](/cli/azure/group#az-group-create) to create a resource group (replace `<resource-group-name>` and `<location>` with your values):

  ```azurecli
  az group create --name <resource-group-name> --location <location>
  ```

1. To create a zone-redundant, geo-replicated registry, copy the following ARM template to a new file and save it using a filename such as `registryZone.json`. 

    By default, the template enables zone redundancy in the registry and a regional replica.


    ```JSON
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
          "acrName": {
            "type": "string",
            "defaultValue": "[concat('acr', uniqueString(resourceGroup().id))]",
            "minLength": 5,
            "maxLength": 50,
            "metadata": {
              "description": "Globally unique name of your Azure Container Registry"
            }
          },
          "location": {
            "type": "string",
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for registry home replica."
            }
          },
          "acrSku": {
            "type": "string",
            "defaultValue": "Premium",
            "allowedValues": [
              "Premium"
            ],
            "metadata": {
              "description": "Tier of your Azure Container Registry. Geo-replication and zone redundancy require Premium SKU."
            }
          },
          "acrZoneRedundancy": {
            "type": "string",
            "defaultValue": "Enabled",
            "metadata": {
              "description": "Enable zone redundancy of registry's home replica. Requires registry location to support availability zones."
            }
          },
          "acrReplicaLocation": {
            "type": "string",
            "metadata": {
              "description": "Short name for registry replica location."
            }
          },
          "acrReplicaZoneRedundancy": {
            "type": "string",
            "defaultValue": "Enabled",
            "metadata": {
              "description": "Enable zone redundancy of registry replica. Requires replica location to support availability zones."
            }
          }
        },
        "resources": [
          {
            "comments": "Container registry for storing docker images",
            "type": "Microsoft.ContainerRegistry/registries",
            "apiVersion": "2020-11-01",
            "name": "[parameters('acrName')]",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('acrSku')]",
              "tier": "[parameters('acrSku')]"
            },
            "tags": {
              "displayName": "Container Registry",
              "container.registry": "[parameters('acrName')]"
            },
            "properties": {
              "adminUserEnabled": "[parameters('acrAdminUserEnabled')]",
              "zoneRedundancy": "[parameters('acrZoneRedundancy')]"
            }
          },
          {
            "type": "Microsoft.ContainerRegistry/registries/replications",
            "apiVersion": "2020-11-01",
            "name": "[concat(parameters('acrName'), '/', parameters('acrReplicaLocation'))]",
            "location": "[parameters('acrReplicaLocation')]",
              "dependsOn": [
              "[resourceId('Microsoft.ContainerRegistry/registries/', parameters('acrName'))]"
            ],
            "properties": {
              "zoneRedundancy": "[parameters('acrReplicaZoneRedundancy')]"
            }
          }
        ],
        "outputs": {
          "acrLoginServer": {
            "value": "[reference(resourceId('Microsoft.ContainerRegistry/registries',parameters('acrName')),'2019-12-01').loginServer]",
            "type": "string"
          }
        }
      }
    ```
    
1. Run the following [az deployment group create](/cli/azure/deployment/group#az-deployment-group-create) command to create the registry using the preceding template file (replace `<resource-group-name>`, `<registry-name>`, and `<replica-location>` with your values). 

     >[!NOTE]
     > If you deploy the template without parameters, it create a unique name for you.
    
    ```azurecli
    az deployment group create \
      --resource-group <resource-group-name> \
      --template-file registryZone.json \
      --parameters acrName=<registry-name> acrReplicaLocation=<replica-location>
    ```

1. In the command output, note the `zoneRedundancy` property for the registry and replica. When `zoneRedundancy` is set to `"Enabled"`, the registry is zone redundant:


    ```JSON
    {
     [...]
    "zoneRedundancy": "Enabled",
    }
    ```

---
