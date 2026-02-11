---
title: "Artifact Streaming (Preview) in Azure Container Registry"
description: "Artifact streaming is a preview feature in Azure Container Registry to enhance managing, scaling, and deploying artifacts through containerized platforms."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: concept-article #Don't change
ms.date: 01/08/2026
ai-usage: ai-assisted
ms.custom:
  - devx-track-azurecli

# Customer intent: "As a developer, I want to utilize artifact streaming in my container registry so that I can efficiently manage and deploy containerized applications across multiple regions with reduced latency and improved scalability."
---

# Artifact streaming in Azure Container Registry (Preview)

Artifact streaming (preview) is a feature in Azure Container Registry that you can use to store and manage container images within a single registry. You can stream the container images to Azure Kubernetes Service (AKS) clusters in multiple regions. This feature accelerates containerized workloads for Azure customers using AKS. By using artifact streaming, you can scale workloads without waiting for slow pull times for your node.

> [!IMPORTANT]
> Artifact streaming is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## How artifact streaming works

Customers with new and existing registries can start artifact streaming for specific repositories or tags. You can store both the original and the streaming artifact in the same container registry. You maintain access to the original and the streaming artifact even after turning off artifact streaming.

Artifact streaming can be particularly useful in the following scenarios:

**Deploying containerized applications to multiple regions**: By using artifact streaming, you can store container images within a single registry and stream them to AKS clusters in multiple regions.

**Reducing image pull latency**: Artifact streaming can reduce time-to-pod readiness by over 15%, depending on the size of the image. This reduction is especially helpful for images that are larger than 30 GB. This feature reduces image pull latency and helps containers start up faster.

**Effective scaling of containerized applications**:  Artifact streaming makes it easier to design, build, and deploy containerized applications at a high scale.

Artifact streaming works across regions, regardless of whether geo-replication is started or not. You can also use artifact streaming with private endpoints.

The state of artifact streaming in a repository (inactive or active) determines whether newly pushed compatible images are automatically converted. By default, all repositories are in an inactive state for artifact streaming. This means that when new compatible images are pushed to the repository, artifact streaming isn't triggered, and the images aren't automatically converted. To enable automatic conversion of newly pushed images, set the repository's artifact streaming to the active state. Once the repository is in the active state, any new compatible container images that are pushed to the repository are automatically converted.

## Pricing and availability

Artifact streaming is currently available only for the **Premium** [service tier](container-registry-skus.md) (SKU).

Use of artifact streaming might increase overall registry storage consumption. If consumption exceeds the included 500 GiB Premium SKU threshold, you might incur additional charges, as outlined in the [pricing](https://azure.microsoft.com/pricing/details/container-registry/).

## Current limitations and requirements

Artifact streaming is currently in preview. The following limitations apply:

* Only images with Linux AMD64 architecture are supported in the preview release.
* The preview release doesn't support Windows-based container images and ARM64 images.
* The preview release partially supports multi-architecture images; only the AMD64 architecture is supported.
* To create an Ubuntu-based node pool in AKS, you must use Ubuntu version 20.04 or higher.
* For Kubernetes, you must use Kubernetes version 1.26 or higher.
* CMK (Customer-Managed Keys) registries aren't currently supported.
* Kubernetes `regcred` isn't currently supported.

Use the [Azure portal](https://portal.azure.com/) or the Azure CLI to manage artifact streaming. For Azure CLI, you can use the [Azure Cloud Shell][Azure Cloud Shell] or a local installation of the Azure CLI to run the command examples in this article. To install or upgrade, see [Install Azure CLI][Install Azure CLI]. We recommend using the latest version of the Azure CLI. The commands in this article require Azure CLI version 2.54.0 or above.

> [!NOTE]
> If you use artifact streaming with a [soft delete policy](container-registry-soft-delete-policy.md) enabled, and you delete an artifact, both the original and artifact streaming versions are deleted. However, only the original version can be [viewed or restored](container-registry-soft-delete-policy.md#view-and-restore-soft-deleted-artifacts) during the retention period.

## Enable and manage artifact streaming

Start artifact streaming to enable pushing, importing, and generating streaming artifacts for container images in an Azure container registry. These instructions outline the process for generating a streaming artifact and managing automatic conversion of streaming artifacts.

### [Azure CLI](#tab/azure-cli)

To enable artifact streaming, you must use a **Premium** [service tier (SKU)](container-registry-skus.md) registry. If you don't already have one, [create a new registry](container-registry-get-started-azure-cli.md) and select the **Premium** service tier, or [change the SKU for an existing registry](container-registry-skus.md#change-registry-sku).

These examples use the Azure CLI to work with a premium Azure Container Registry named `mystreamingtest` in the `my-streaming-test` resource group located in the West US region, along with an example Jupyter Notebook image. Replace these names with your own values.

## Import an image and create artifact streaming

First, run the [az config](/cli/azure/azure-cli-configuration) command to set your registry name as the default for `az acr` commands:

```azurecli-interactive
az config set defaults.acr="mystreamingtest"
```

If you don't already have an image that you want to use, run the [az acr import][az-acr-import] command to import a Jupyter Notebook image from Docker Hub:

```azurecli-interactive
az acr import --source docker.io/jupyter/all-spark-notebook:latest -t jupyter/all-spark-notebook:latest
```

> [!NOTE]
> If you see an error from the docker registry, it might be due to [rate limiting](/troubleshoot/azure/azure-container-instances/configuration-setup/docker-hub-rate-limit-registryerrorresponse). To avoid this error, consider authenticating with Docker Hub by [providing your Docker ID and password](container-registry-import-images.md#import-from-docker-hub) by using the `--username` and `--password` parameters in the `az acr import` command.

To create a streaming artifact from the image, run the [az acr artifact-streaming create][az-acr-artifact-streaming-create] command:

```azurecli-interactive
az acr artifact-streaming create --image jupyter/all-spark-notebook:latest
```

An operation ID is generated during this process. If you want to stop the process of creating the streaming artifact, run [az acr artifact-streaming operation cancel][az-acr-artifact-streaming-operation-cancel] with the operation ID:

   ```azurecli-interactive
   az acr artifact-streaming operation cancel --repository jupyter/all-spark-notebook --id c015067a-7463-4a5a-9168-3b17dbe42ca3
   ```

After the streaming artifact is generated, you can confirm its creation by using [az acr manifest list-referrers][az-acr-manifest-list-referrers] to list streaming artifacts:

```azurecli-interactive
az acr manifest list-referrers -n jupyter/all-spark-notebook:latest
```

## Manage artifact streaming for new images in the repository

By default, newly pushed or imported images in a repository aren't automatically enabled for artifact streaming. To ensure that new images pushed into the repository automatically trigger the generation of streaming artifacts, use the [az acr artifact-streaming update][az-acr-artifact-streaming-update] command on the repository:

```azurecli-interactive
az acr artifact-streaming update --repository jupyter/all-spark-notebook --enable-streaming true
```

> [!NOTE]
> Existing images in the repository aren't automatically converted when you run this command.

To verify that automatic conversion is working, push a new image to the repository, and then run the  [az acr artifact-streaming operation show][az-acr-artifact-streaming-operation-show] command:

```azurecli-interactive
az acr artifact-streaming operation show --image jupyter/all-spark-notebook:newtag
```

After confirming that the conversion is working, you can [stream images from ACR to Azure Kubernetes Service (AKS) clusters](/azure/aks/artifact-streaming).

To disable automatic conversion of streaming artifacts in the repository, run [az acr artifact-streaming update][az-acr-artifact-streaming-update] and set `--enable-streaming` to `false`:

```azurecli-interactive
az acr artifact-streaming update --repository jupyter/all-spark-notebook --enable-streaming false
```

# [Azure portal](#tab/azure-portal)

To enable artifact streaming, you must use a **Premium** [service tier (SKU)](container-registry-skus.md) registry. If you don't already have one, [create a new registry](container-registry-get-started-portal.md) and select the **Premium** service tier, or [change the SKU for an existing registry](container-registry-skus.md#change-registry-sku).

Follow the steps to enable artifact streaming for an Azure container repository in the [Azure portal](https://portal.azure.com).

1. Go to your Azure container registry in the Azure portal.
1. In the service menu, under  **Services**, select **Repositories**.
1. Select **Start artifact streaming.**

   :::image type="content" source="media/container-registry-artifact-streaming/start-artifact-streaming.png" lightbox="media/container-registry-artifact-streaming/start-artifact-streaming.png" alt-text="Screenshot of an Azure container registry with the option to enable artifact streaming.":::

The status of **Artifact streaming** changes to **Active**. Any new compatible container images that you push to the repository are automatically converted to streaming artifacts. To change this setting so that new images aren't converted, select **Stop artifact streaming**.

When you enable artifact streaming on a repository, existing images in the repository aren't automatically converted to use artifact streaming. To create a streaming artifact for a previously imported image, follow these steps:

1. From the **Repositories** pane, select the image that you want to convert.
1. Select **Create streaming artifact.**

Once the process is complete, you can view the streaming artifact in the **Referrers** tab of the image.

After you enable artifact streaming for images in the repository, you can [stream images from ACR to Azure Kubernetes Service (AKS) clusters](/azure/aks/artifact-streaming).

---

## Related content

* Get tips for [troubleshooting artifact streaming problems](troubleshoot-artifact-streaming.md).
* Learn more about [registries, repositories, and artifacts](container-registry-concepts.md).

<!-- LINKS - External -->
[Install Azure CLI]: /cli/azure/install-azure-cli
[Azure Cloud Shell]: /azure/cloud-shell/quickstart
[az-acr-import]: /cli/azure/acr#az-acr-import
[az-acr-artifact-streaming-create]: /cli/azure/acr/artifact-streaming#az-acr-artifact-streaming-create
[az-acr-manifest-list-referrers]: /cli/azure/acr/manifest#az-acr-manifest-list-referrers
[az-acr-artifact-streaming-operation-cancel]: /cli/azure/acr/artifact-streaming/operation#az-acr-artifact-streaming-operation-cancel
[az-acr-artifact-streaming-operation-show]: /cli/azure/acr/artifact-streaming/operation#az-acr-artifact-streaming-operation-show
[az-acr-artifact-streaming-update]: /cli/azure/acr/artifact-streaming#az-acr-artifact-streaming-update
