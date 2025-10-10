---
title: Quickstart - Create Connected Registry Using the CLI and Portal
description: Use Azure CLI commands or Azure portal to create a connected Azure container registry resource that can synchronize images and other artifacts with the cloud registry.
ms.topic: quickstart
ms.date: 05/20/2025
ms.author: rayoflores
author: rayoef
ms.custom: mode-api, devx-track-azurecli
ms.devlang: azurecli
ms.service: azure-container-registry
# Customer intent: "As a cloud infrastructure engineer, I want to create a connected registry in Azure using the CLI or portal, so that I can synchronize images and artifacts between my local environment and the cloud registry."
---

# Quickstart: Create a connected registry using the Azure CLI or Azure portal

In this quickstart, you use the Azure CLI and Azure portal to create a [connected registry](intro-connected-registry.md) resource in Azure. The connected registry feature of Azure Container Registry allows you to deploy a registry remotely or on your premises and synchronize images and other artifacts with the cloud registry. 

Here you create two connected registry resources for a cloud registry: one connected registry allows read and write (artifact pull and push) functionality and one allows read-only functionality. 

After creating a connected registry, you can follow other guides to deploy and use it on your on-premises or remote infrastructure.

## Prerequisites

#### [Azure portal](#tab/azure-portal)

* Azure Container registry - If you don't already have a container registry, [create one](container-registry-get-started-portal.md) (Premium tier required) in a [region](intro-connected-registry.md#available-regions) that supports connected registries. 

To import images to the container registry, use the Azure CLI
[!INCLUDE [Prepare Azure CLI environment](~/reusable-content/azure-cli/azure-cli-prepare-your-environment-no-header.md)]

#### [Azure CLI](#tab/azure-cli)

* Azure Container registry - If you don't already have a container registry, [create one](container-registry-get-started-azure-cli.md) (Premium tier required) in a [region](intro-connected-registry.md#available-regions) that supports connected registries. 

[!INCLUDE [Prepare Azure CLI environment](~/reusable-content/azure-cli/azure-cli-prepare-your-environment-no-header.md)]

---

## Enable the dedicated data endpoint for the cloud registry

#### [Azure portal](#tab/azure-portal)

Enable the [dedicated data endpoint](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) for the Azure container registry in the cloud. This step is needed for a connected registry to communicate with the cloud registry.

1. In the [Azure portal](https://portal.azure.com), navigate to your container registry.
1. Select **Networking > Public access**.
Select the **Enable dedicated data endpoint** checkbox.
1. Select **Save**.

:::image type="content" source="media/dedicated-data-endpoints/data-endpoint-connected-registry.png" alt-text="Screenshot of enabling dedicated data endpoint." lightbox="media/dedicated-data-endpoints/data-endpoint-connected-registry.png":::

#### [Azure CLI](#tab/azure-cli)

Enable the dedicated data endpoint for the Azure container registry in the cloud by using the [az acr update][az-acr-update] command. This step is needed for a connected registry to communicate with the cloud registry.

```azurecli
# Set the REGISTRY_NAME environment variable to identify the existing cloud registry
REGISTRY_NAME=<container-registry-name>

az acr update --name $REGISTRY_NAME \
  --data-endpoint-enabled
```

---

[!INCLUDE [container-registry-connected-import-images](./includes/container-registry-connected-import-images.md)]

## Create a connected registry resource for read and write functionality

#### [Azure portal](#tab/azure-portal)

The following steps create a connected registry in [ReadWrite mode](intro-connected-registry.md#modes) that is linked to the cloud registry.

1. In the [Azure portal](https://portal.azure.com), navigate to your container registry.
1. Select **Connected registries (Preview) > + Create**.
1. Enter or select the values in the following table, and select **Save**.


|Item  |Description  |
|---------|---------|
|Parent     | Select **No parent** for a connected registry linked to the cloud registry.        |
|Mode     | Select **ReadWrite**.         |
|Name     | The connected registry name must start with a letter and contain only alphanumeric characters. It must be 5 to 40 characters long and unique in the hierarchy for this Azure container registry.       |
|Logging properties     | Accept the default settings.       |
|Sync properties    | Accept the default settings. Because there's no synchronization schedule defined by default, the repositories are synchronized between the cloud registry and the connected registry without interruptions.      |
|Repositories     | Select or enter the names of the repositories you imported in the previous step. The specified repositories are synchronized between the cloud registry and the connected registry once deployed.     |

:::image type="content" source="media/quickstart-connected-registry-portal/create-readwrite-connected-registry.png" alt-text="Create a connected registry in ReadWrite mode" lightbox="media/quickstart-connected-registry-portal/create-readwrite-connected-registry.png":::

#### [Azure CLI](#tab/azure-cli)

You can also use the [az acr connected-registry create][az-acr-connected-registry-create] command to create a connected registry with read-only functionality. 

```azurecli
# Set the CONNECTED_REGISTRY_READ environment variable to provide a name for the connected registry with read-only functionality
CONNECTED_REGISTRY_RO=<connnected-registry-name>
az acr connected-registry create --registry $REGISTRY_NAME \
  --parent $CONNECTED_REGISTRY_RW \
  --name $CONNECTED_REGISTRY_RO \
  --repository "hello-world" "acr/connected-registry" \
  --mode ReadOnly
```

This command creates a connected registry resource whose name is the value of *$CONNECTED_REGISTRY_RO* and links it to the cloud registry named with the value of *$REGISTRY_NAME*. 
* The specified repositories are synchronized between the parent registry named with the value of *$CONNECTED_REGISTRY_RW* and the connected registry once deployed.
* The resource is created in the [ReadOnly mode](intro-connected-registry.md#modes), which enables read-only (artifact pull) functionality once deployed. 
* The repositories are synchronized between the parent registry and the connected registry without interruptions because there's no synchronization schedule defined for this connected registry.

---

## Create a connected registry resource for read-only functionality

#### [Azure portal](#tab/azure-portal)

The following steps create a connected registry in [ReadOnly mode](intro-connected-registry.md#modes)  whose parent is the connected registry you created in the previous section. This connected registry enables read-only (artifact pull) functionality once deployed.

1. In the [Azure portal](https://portal.azure.com), navigate to your container registry.
1. Select **Connected registries (Preview) > + Create**.
1. Enter or select the values in the following table, and select **Save**.


|Item  |Description  |
|---------|---------|
|Parent     | Select the connected registry you created previously.        |
|Mode     | Select **ReadOnly**.         |
|Name     | The connected registry name must start with a letter and contain only alphanumeric characters. It must be 5 to 40 characters long and unique in the hierarchy for this Azure container registry.      |
|Logging properties     | Accept the default settings.       |
|Sync properties    | Accept the default settings. Because there's no synchronization schedule defined by default, the repositories are synchronized between the cloud registry and the connected registry without interruptions.      |
|Repositories     | Select or enter the names of the repositories you imported in the previous step. The specified repositories are synchronized between the parent registry and the connected registry once deployed.     |

:::image type="content" source="media/quickstart-connected-registry-portal/create-readonly-connected-registry.png" alt-text="Create a connected registry in ReadOnly mode" lightbox="media/quickstart-connected-registry-portal/create-readonly-connected-registry.png":::

#### [Azure CLI](#tab/azure-cli)

You can also use the [az acr connected-registry create][az-acr-connected-registry-create] command to create a connected registry with read-only functionality. 

```azurecli
# Set the CONNECTED_REGISTRY_READ environment variable to provide a name for the connected registry with read-only functionality
CONNECTED_REGISTRY_RO=<connnected-registry-name>
az acr connected-registry create --registry $REGISTRY_NAME \
  --parent $CONNECTED_REGISTRY_RW \
  --name $CONNECTED_REGISTRY_RO \
  --repository "hello-world" "acr/connected-registry" \
  --mode ReadOnly
```

This command creates a connected registry resource whose name is the value of *$CONNECTED_REGISTRY_RO* and links it to the cloud registry named with the value of *$REGISTRY_NAME*. 
* The specified repositories are synchronized between the parent registry named with the value of *$CONNECTED_REGISTRY_RW* and the connected registry once deployed.
* The resource is created in the [ReadOnly mode](intro-connected-registry.md#modes), which enables read-only (artifact pull) functionality once deployed. 
* The repositories are synchronized between the parent registry and the connected registry without interruptions because there's no synchronization schedule defined for this connected registry.

---

## Verify that the resources are created

#### [Azure portal](#tab/azure-portal)

Select a connected registry in the portal to view its properties, such as its connection status (Offline, Online, or Unhealthy) and whether it activated (deployed on-premises). In the following example, the connected registry isn't deployed. The connection state of "Offline" indicates that it disconnected from the cloud.

:::image type="content" source="media/quickstart-connected-registry-portal/connected-registry-properties.png" alt-text="View connected registry properties" lightbox="media/quickstart-connected-registry-portal/connected-registry-properties.png":::

From this view, you can also generate a connection string and optionally generate passwords for the [sync token](overview-connected-registry-access.md#sync-token). A connection string contains configuration settings used for deploying a connected registry and synchronizing content with a parent registry.

#### [Azure CLI](#tab/azure-cli)

You can use the connected registry [az acr connected-registry list][az-acr-connected-registry-list] command to verify that the resources are created. 

```azurecli
az acr connected-registry list \
  --registry $REGISTRY_NAME \
  --output table
```

You should see a response as follows. Because the connected registries aren't yet deployed, the connection state of "Offline" indicates that they're currently disconnected from the cloud.

```
NAME                 MODE        CONNECTION STATE    PARENT               LOGIN SERVER    LAST SYNC (UTC)
-------------------  --------    ------------------  -------------------  --------------  -----------------
myconnectedregrw    ReadWrite    Offline
myconnectedregro    ReadOnly     Offline             myconnectedregrw
```

---

## Next steps

In this quickstart, you used the Azure CLI and Azure portal to create two connected registry resources in Azure. Those new connected registry resources are tied to your cloud registry and allow synchronization of artifacts with the cloud registry.

Continue to the connected registry deployment guides to learn how to deploy and use a connected registry in your infrastructure.

> [!div class="nextstepaction"]
> [Quickstart: Deploy connected registry to Azure Arc][quickstart-connected-registry-arc-cli]

<!-- LINKS - internal -->
[az-acr-connected-registry-create]: /cli/azure/acr/connected-registry#az_acr_connected_registry_create
[az-acr-connected-registry-list]: /cli/azure/acr/connected-registry#az_acr_connected_registry_list
[az-acr-create]: /cli/azure/acr#az_acr_create
[az-acr-update]: /cli/azure/acr#az_acr_update
[az-acr-import]: /cli/azure/acr#az_acr_import
[az-group-create]: /cli/azure/group#az_group_create
[container-registry-intro]: container-registry-intro.md
[container-registry-skus]: container-registry-skus.md
[quickstart-connected-registry-arc-cli]:quickstart-connected-registry-arc-cli.md
