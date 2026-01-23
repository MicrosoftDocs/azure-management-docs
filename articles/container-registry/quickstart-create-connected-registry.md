---
title: Create a connected registry for your Azure Container Registry
description: Use Azure CLI commands or Azure portal to create a connected Azure container registry resource that can synchronize images and other artifacts with the cloud registry.
ms.topic: how-to
ms.date: 01/23/2026
ms.author: rayoflores
author: rayoef
ms.custom: mode-api, devx-track-azurecli
ms.devlang: azurecli
ms.service: azure-container-registry
# Customer intent: "As a cloud infrastructure engineer, I want to create a connected registry in Azure using the CLI or portal, so that I can synchronize images and artifacts between my local environment and the cloud registry."
---

# Create a connected registry for your Azure Container Registry

This article shows how to use the Azure CLI or Azure portal to create a [connected registry](intro-connected-registry.md) resource in Azure. The [connected registry feature of Azure Container Registry](intro-connected-registry.md) allows you to deploy a registry remotely or on your premises and synchronize images and other artifacts with a cloud=based Azure container registry.

In this article, you create two connected registry resources for an existing cloud registry: one that supports read and write (artifact pull and push) functionality, and one that supports read-only functionality. 

After creating a connected registry, you [can deploy and use it on your on-premises or remote infrastructure](quickstart-connected-registry-arc-cli.md).

## Prerequisites

You must have an Azure Container registry in the [**Premium** SKU (pricing plan)](container-registry-skus.md) to use the connected registry feature. If you don't already have a container registry, [create one](container-registry-get-started-portal.md), being sure to select **Premium** for **Pricing plan**.

Even if you use the Azure portal to create your connected registry resource, these steps use Azure CLI to import images to the container registry.

[!INCLUDE [Prepare Azure CLI environment](~/reusable-content/azure-cli/azure-cli-prepare-your-environment-no-header.md)]

## Enable the dedicated data endpoint for the cloud registry

Enable the [dedicated data endpoint](container-registry-firewall-access-rules.md#enable-dedicated-data-endpoints) for the Azure container registry in the cloud. This step is necessary to allow the connected registry to communicate with the cloud registry.

### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), go to your container registry.
1. In the service menu, under **Settings**, select **Networking**.
1. In the **Public access** section, select the **Enable dedicated data endpoint** checkbox.
1. Select **Save**.

### [Azure CLI](#tab/azure-cli)

Use the [az acr update][az-acr-update] command:

```azurecli
# Set the REGISTRY_NAME environment variable to identify the existing cloud registry
REGISTRY_NAME=<container-registry-name>

az acr update --name $REGISTRY_NAME \
  --data-endpoint-enabled
```

---

[!INCLUDE [container-registry-connected-import-images](./includes/container-registry-connected-import-images.md)]

## Create a connected registry resource for read and write functionality

Follow these steps to create a connected registry in [ReadWrite mode](intro-connected-registry.md#modes) that is linked to your cloud registry.

### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com), go to your container registry.
1. In the service menu, under **Services**, select **Connected registries**.
1. Select **Create**.
1. Enter or select the values in the following table, and select **Save**.

   |Item  |Description  |
   |---------|---------|
   |Parent     | Select **No parent** for a connected registry linked directly to the cloud registry.        |
   |Mode     | Select **ReadWrite**.         |
   |Name     | The connected registry name must start with a letter and contain only alphanumeric characters. It must be 5 to 40 characters long and unique in the hierarchy for this Azure container registry. |
   |Logging properties     | Keep the default settings.       |
   |Sync properties    | Keep the default settings. Because there's no synchronization schedule defined by default, the repositories synchronize between the cloud registry and the connected registry without interruptions.      |
   |Repositories     | Select or enter the names of the repositories you imported in the previous step. The specified repositories synchronize between the cloud registry and the connected registry once deployed.     |

:::image type="content" source="media/quickstart-connected-registry-portal/create-readwrite-connected-registry.png" alt-text="Screenshot of the options to create a connected registry in ReadWrite mode.":::

#### [Azure CLI](#tab/azure-cli)

Use the [az acr connected-registry create][az-acr-connected-registry-create] command to create a connected registry with read-write functionality.

```azurecli
# Set the CONNECTED_REGISTRY_READ environment variable to provide a name for the connected registry with read-write functionality
CONNECTED_REGISTRY_RW=<connnected-registry-name>
az acr connected-registry create --registry $REGISTRY_NAME \
  --name $CONNECTED_REGISTRY_RW \
  --repository "hello-world" "acr/connected-registry" \
  --mode ReadWrite
```

This command creates a connected registry resource using the value stored in *$CONNECTED_REGISTRY_RW* and links it to the cloud registry that you specified earlier for *$REGISTRY_NAME*.

* The specified repositories synchronize between the parent registry and the connected registry.
* The resource is created in the [ReadWrite mode](intro-connected-registry.md#modes), which enables push and pull functionality.
* The repositories synchronize between the parent registry and the connected registry without interruptions because there's no synchronization schedule defined for this connected registry.

---

## Create a connected registry resource for read-only functionality

Next, create a connected registry in [ReadOnly mode](intro-connected-registry.md#modes)  whose parent is the connected registry you created in the previous section. This connected registry enables read-only (artifact pull) functionality once deployed.

### [Azure portal](#tab/azure-portal)

1. In **Connected registries**, select **Create** again.
1. Repeat the steps to create a connected registry, with the following changes to the values in the previous table:

   |Item  |Description  |
   |---------|---------|
   |Parent     | Select the connected registry you created previously.        |
   |Mode     | Select **ReadOnly**.         |

1. Select **Save**.

#### [Azure CLI](#tab/azure-cli)

Use the [az acr connected-registry create][az-acr-connected-registry-create] command to create a connected registry with read-only functionality.

```azurecli
# Set the CONNECTED_REGISTRY_READ environment variable to provide a name for the connected registry with read-only functionality
CONNECTED_REGISTRY_RO=<connnected-registry-name>
az acr connected-registry create --registry $REGISTRY_NAME \
  --parent $CONNECTED_REGISTRY_RW \
  --name $CONNECTED_REGISTRY_RO \
  --repository "hello-world" "acr/connected-registry" \
  --mode ReadOnly
```

This command creates a connected registry resource named with the value stored in *$CONNECTED_REGISTRY_RO* and links it to the same cloud registry that you specified earlier for *$REGISTRY_NAME*.

* The specified repositories synchronize between the *$CONNECTED_REGISTRY_RW* parent connected registry that you created in the previous step and the new connected registry.
* The resource is created in the [ReadOnly mode](intro-connected-registry.md#modes), which enables read-only (artifact pull) functionality.
* The repositories synchronize between the parent registry and the connected registry without interruptions because there's no synchronization schedule defined for this connected registry.

---

## Verify that the resources are created

After you create the connected registry resources, verify that they exist and view their properties.

#### [Azure portal](#tab/azure-portal)

Select a connected registry in the portal to view its properties, such as its connection status (**Offline**, **Online**, or **Unhealthy**) and whether it activated (deployed on-premises). In the following example, the connected registry isn't deployed yet, so the connection state is **Offline**.

:::image type="content" source="media/quickstart-connected-registry-portal/connected-registry-properties.png" alt-text="Screenshot of a connected registry showing its connection status and other properties.":::

From this view, you can select **Connection string** to generate a connection string, which contains configuration settings used for deploying a connected registry and synchronizing content with a parent registry. You can also optionally generate passwords for the [sync token](overview-connected-registry-access.md#sync-token).

#### [Azure CLI](#tab/azure-cli)

Use the connected registry [az acr connected-registry list][az-acr-connected-registry-list] command to verify that the resources are created.

```azurecli
az acr connected-registry list \
  --registry $REGISTRY_NAME \
  --output table
```

You see a response similar to the example below. Because the connected registries aren't yet deployed, the connection state of `Offline` indicates that they're currently disconnected from the cloud.

```
NAME                 MODE        CONNECTION STATE    PARENT               LOGIN SERVER    LAST SYNC (UTC)
-------------------  --------    ------------------  -------------------  --------------  -----------------
myconnectedregrw    ReadWrite    Offline
myconnectedregro    ReadOnly     Offline             myconnectedregrw
```

---

## Configure the connected registry sync schedule and window

In this article, you create the connected registries without a defined synchronization schedule and window. YOu can optionally use a specific schedule. To do so, use the [az acr connected-registry update][az-acr-connected-registry-update] to set a schedule. Use CRON expressions to define the sync schedule and the ISO 8601 duration format for the sync window.

For example, the following command configures the connected registry to schedule a daily sync at 12:00 PM UTC, wiht a  4 hour synchronization window (PT4H):

```azurecli 
az acr connected-registry update --registry myacrregistry \ 
--name myconnectedregistry \ 
--sync-schedule "0 12 * * *" \
--sync-window PT4H
```

In this example, the connected registry will sync every minute with the cloud registry:

```azurecli 
az acr connected-registry update --registry myacrregistry \ 
--name myconnectedregistry \ 
--sync-schedule "* * * * *"    
```


## Next steps

In this quickstart, you used the Azure CLI and Azure portal to create two connected registry resources in Azure. These new connected registry resources tie to your cloud registry and allow synchronization of artifacts with the cloud registry.

Next, learn how to [deploy connected registry to an Azure Arc-enabled Kubernetes cluster][quickstart-connected-registry-arc-cli].

<!-- LINKS - internal -->
[az-acr-connected-registry-create]: /cli/azure/acr/connected-registry#az-acr-connected-registry-create
[az-acr-connected-registry-list]: /cli/azure/acr/connected-registry#az-acr-connected-registry-list
[az-acr-update]: /cli/azure/acr#az-acr-update
[quickstart-connected-registry-arc-cli]:quickstart-connected-registry-arc-cli.md
