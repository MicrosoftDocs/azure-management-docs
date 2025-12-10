---
title: Recover Deleted Artifacts in Azure Container Registry
description: Learn how to enable the soft delete policy in Azure Container Registry to manage and recover accidentally deleted artifacts with a set retention period.
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to #Don't change
ms.date: 01/22/2024
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: As a cloud administrator, I want to enable the soft delete policy for Azure Container Registry artifacts so that I can recover accidentally deleted items within a specified retention period, ensuring data integrity and minimizing potential data loss.
---

# Recover deleted artifacts with soft delete policy in Azure Container Registry (Preview)

Azure Container Registry (ACR) allows you to enable the *soft delete policy* that allows you to recover any accidentally deleted artifacts for a set retention period.

:::image type="content" source="./media/container-registry-soft-delete/02-soft-delete.png" alt-text="Diagram of soft delete artifacts lifecycle.":::


The soft delete policy can be enabled/disabled at any time. Once you enable the soft-delete policy in ACR, any deleted artifacts are treated as soft deleted artifacts with a set retention period. Within the retention period, you can list, filter, and restore all deleted artifacts. After the retention period expires, the soft deleted artifacts are permanently deleted and can no longer be restored.

### Retention period

The default retention period for soft deleted artifacts is seven days, but you can select any value value between 1 and 90 days. You can set, update, and change the retention policy value. The soft deleted artifacts expire once the retention period is complete.

The autopurge runs every 24 hours and always considers the current value of retention days before permanently deleting artifacts. For example, after five days from deleting an artifact, if you change the value of retention days from 7 days to 14 days, the artifact will only expire after 14 days from the initial soft delete.

## Availability and pricing information

This feature is available in all the service tiers (also known as SKUs). For information about registry service tiers, see [Azure Container Registry service tiers](container-registry-skus.md).

> [!NOTE]
> Soft deleted artifacts are billed as per active SKU pricing for storage.

## Current limitations

> [!IMPORTANT]
> The soft delete policy is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

* ACR currently doesn't support manually purging soft deleted artifacts.
* The soft delete policy doesn't support registries configured for zone redundancy or geo-replication.
* ACR doesn't allow enabling both the [retention policy](container-registry-retention-policy.md) and the soft delete policy.

## Prerequisites

* To perform soft delete operations, a user requires following permissions (at the container registry level):

| Permission                                                    | Description                   |
| ------------------------------------------------------------- | ----------------------------- |
| `Microsoft.ContainerRegistry/registries/deleted/read`           | List soft-deleted artifacts   |
| `Microsoft.ContainerRegistry/registries/deleted/restore/action` | Restore soft-deleted artifact |

* You can use the Azure Cloud Shell or a local installation of the Azure CLI to run the command examples in this article. If you'd like to use it locally, version 2.0.74 or later is required; we recommend running the most recent version. If you need to install or upgrade, see [HJow to install the Azure CLI](/cli/azure/install-azure-cli).

* If you don't have an Azure account, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

## Enable soft delete policy for registry - CLI

1. Update soft delete policy for a given `MyRegistry` ACR with a retention period set between 1 to 90 days.

    ```azurecli-interactive
    az acr config soft-delete update -r MyRegistry --days 7 --status <enabled/disabled>
    ```

2. Show configured soft delete policy for a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr config soft-delete show -r MyRegistry 
    ```

### List the soft deleted artifacts- CLI

The `az acr repository list-deleted` commands enable fetching and listing of the soft deleted repositories. For more information use `--help`.

1. List the soft deleted repositories in a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr repository list-deleted -n MyRegistry
    ```

   The `az acr manifest list-deleted` commands enable fetching and listing of the soft delete manifests.

1. List the soft deleted manifests of a `hello-world` repository in a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr manifest list-deleted -r MyRegistry -n hello-world
    ```

   The `az acr manifest list-deleted-tags` commands  enable fetching and listing of the soft delete tags.

1. List the soft delete tags of a `hello-world` repository in a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr manifest list-deleted-tags -r MyRegistry -n hello-world
    ```

1. Filter the soft delete tags of a `hello-world` repository to match tag `latest` in a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr manifest list-deleted-tags -r MyRegistry -n hello-world:latest
    ```

### Restore the soft deleted artifacts - CLI

The `az acr manifest restore` commands restore a single image by tag and digest. 

1. Restore the image of a `hello-world` repository by tag `latest`and digest `sha256:abc123` in a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr manifest restore -r MyRegistry -n hello-world:latest -d sha256:abc123
    ```

1. Restore the most recently deleted manifest of a `hello-world` repository by tag `latest` in a given `MyRegistry` ACR.

   ```azurecli-interactive
   az acr manifest restore -r MyRegistry -n hello-world:latest
   ```

   Force restore overwrites the existing tag with the same name in the repository. If the soft delete policy is enabled during force restore. The overwritten tag is soft deleted. You can force restore with specific arguments `--force, -f`.  

1. Force restore the image of a `hello-world` repository by tag `latest`and digest `sha256:abc123` in a given `MyRegistry` ACR.

   ```azurecli-interactive
   az acr manifest restore -r MyRegistry -n hello-world:latest -d sha256:abc123 -f
   ```

   > [!IMPORTANT]
   > Restoring a [manifest list](push-multi-architecture-images.md#manifest-list) won't recursively restore any underlying soft deleted manifests.
   > If you're restoring soft deleted [ORAS artifacts](container-registry-manage-artifact.md), then restoring a subject doesn't recursively restore the referrer chain. Also, the subject has to be restored first, only then a referrer manifest is allowed to restore. Otherwise it throws an error.

## Enable soft delete policy for registry - Portal

You can also enable a registry's soft delete policy in the [Azure portal](https://portal.azure.com).

1. Go to your Azure Container Registry in the Azure portal.
1. In **Overview**, verify the status of **Soft delete** (Preview).
1. If the **Status** is **Disabled**, select **Disabled** to open the **Properties** pane.
1. Select **Soft delete** checkbox.
1. Enter the number of days between 0 and 90 for retaining deleted artifacts.
1. Select **Save**.

When soft delete is enabled, and you perform actions such as untagging a manifest or deleting an artifact, you can view these tags and artifacts by selecting **Managed deleted artifacts** before the number of retention days expire, as described in the next section.

### Restore soft deleted artifacts - Portal

1. Go to your Azure Container Registry in the Azure portal.
1. In the service menu, under **Services**, select **Repositories**.
1. In the **Repositories**, select a repository.
1. Select **Manage deleted artifacts**.

   :::image type="content" source="./media/container-registry-soft-delete/soft-delete-manage-deleted-artifacts.png" alt-text="Screenshot of manage deleted artifacts." lightbox="./media/container-registry-soft-delete/soft-delete-manage-deleted-artifacts.png":::

1. In the row for the deleted artifact that you want to restore, select **Restore**.
1. In the **Restore Artifact** pane, select the tag to restore. You can only select one tag with which to restore your artifact. To recover additional tags, you must restore them separately.
1. Select **Restore**.

### Restore soft deleted repositories - Portal

1. Go to your Azure Container Registry in the Azure portal.
1. In the service menu, under **Services**, select **Repositories**.
1. In the **Repositories**, select a repository.
1. Select **Manage Deleted Respositories**.
1. In the row for the deleted repository that you want to restore, select **Restore**.
1. In the **Restore Artifact** pane, select the tag to restore. You can only select one tag with which to restore your repository. To recover additional tags, you must restore them separately.
1. Select **Restore**.

> [!IMPORTANT]
>  Importing a soft deleted image at both source and target resources is blocked.
>  Pushing an image to the soft deleted repository will restore the soft deleted repository.
>  Pushing an image that shares a same manifest digest with the soft deleted image is not allowed. Instead restore the soft deleted image.

## Next steps

* Learn more about options to [delete images and repositories](container-registry-delete.md) in Azure Container Registry.
