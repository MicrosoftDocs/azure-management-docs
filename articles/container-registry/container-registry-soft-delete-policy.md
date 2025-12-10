---
title: Recover deleted artifacts with the soft delete policy in Azure Container Registry (preview)
description: Learn how to enable the soft delete policy in Azure Container Registry to manage and recover accidentally deleted artifacts with a set retention period.
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: how-to #Don't change
ms.date: 12/10/2025
ms.custom:
  - devx-track-azurecli
  - sfi-image-nochange
# Customer intent: As a cloud administrator, I want to enable the soft delete policy for Azure Container Registry artifacts so that I can recover accidentally deleted items within a specified retention period, ensuring data integrity and minimizing potential data loss.
---

# Recover deleted artifacts with the soft delete policy in Azure Container Registry (preview)

Azure Container Registry (ACR) allows you to enable the *soft delete policy* that lets you recover accidentally deleted artifacts for a set retention period.

> [!IMPORTANT]
> The soft delete policy is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

You can enable or disable the soft delete policy at any time in the Azure portal or by using Azure CLI. When you enable the soft delete policy in ACR, the registry treats any deleted artifacts as soft-deleted artifacts with a set retention period. Within the retention period, you can list, filter, and restore all deleted artifacts. After the retention period expires, the soft-deleted artifacts are permanently deleted and can't be restored.

:::image type="content" source="./media/container-registry-soft-delete/02-soft-delete.png" alt-text="Diagram of soft delete artifact lifecycle.":::

The default retention period for soft-deleted artifacts is seven days, but you can select any value between 1 and 90 days. You can set, update, and change the retention policy value. The soft-deleted artifacts expire once the retention period is complete.

The autopurge runs every 24 hours and always considers the current value of retention days before permanently deleting artifacts. For example, if you deleted an artifact five days ago, then change the retention value from 7 days to 14 days, the artifact expires after 14 days from the date it was deleted.

This preview feature is available in all [service tiers](container-registry-skus.md) (also known as SKUs).

> [!NOTE]
> Soft-deleted artifacts are billed as per active SKU pricing for storage.

Keep in mind the following current limitations:

* Azure Container Registry currently doesn't support manually purging soft-deleted artifacts.
* The soft delete policy doesn't support registries configured for zone redundancy or geo-replication.
* Azure Container Registry doesn't allow enabling both the [retention policy](container-registry-retention-policy.md) and the soft delete policy.

## Prerequisites

* If you don't have an Azure account, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.

* To perform soft delete operations, a user requires the following permissions at the container registry level:

  * `Microsoft.ContainerRegistry/registries/deleted/read`: List soft-deleted artifacts
  * `Microsoft.ContainerRegistry/registries/deleted/restore/action`: Restore soft-deleted artifacts

* For Azure CLI, you can use the Azure Cloud Shell or a local installation to run the commands listed in this article. We recommend using the most recent version of the Azure CLI. If you need to install or upgrade, see [How to install the Azure CLI](/cli/azure/install-azure-cli).

## Enable soft delete policy

You can enable the soft delete policy for your Azure Container Registry in the Azure portal or by using Azure CLI.

### [Azure portal](#tab/azure-portal)

1. Go to your Azure Container Registry in the Azure portal.
1. In **Overview**, check the status of **Soft delete (Preview)**.
1. If the **Status** is **Disabled**, select **Disabled** to open the **Properties** pane.
1. Select the **Soft delete** checkbox.
1. Enter a number of days between 1 and 90 to retain deleted artifacts.
1. Select **Save**.

When soft delete is enabled, and you perform actions such as untagging a manifest or deleting an artifact, you can view these tags and artifacts by selecting **Managed deleted artifacts** before the retention period expires, as described in the next section.

### [Azure CLI](#tab/azure-cli)

### Enable soft delete policy for registry

1. Update the soft delete policy for a given `MyRegistry` ACR with a retention period set between 1 to 90 days.

    ```azurecli-interactive
    az acr config soft-delete update -r MyRegistry --days 7 --status <enabled/disabled>
    ```

1. Show the configured soft delete policy for a given `MyRegistry` ACR.

    ```azurecli-interactive
    az acr config soft-delete show -r MyRegistry 
    ```

---

## View and restore soft-deleted artifacts

You can view and restore soft-deleted artifacts during the current retention period set for a repository. Keep in mind the following considerations:

* You can't import a soft-deleted image at both source and target resources.
* Pushing an image to a soft-deleted repository restores that repository.
* Pushing an image that shares the same manifest digest with the soft-deleted image isn't allowed. Instead, restore the soft-deleted image.

### [Azure portal](#tab/azure-portal)

### Restore soft-deleted artifacts

1. Go to your Azure Container Registry in the Azure portal.
1. In the service menu, under **Services**, select **Repositories**.
1. In **Repositories**, select a repository.
1. Select **Manage deleted artifacts**.

   :::image type="content" source="./media/container-registry-soft-delete/soft-delete-manage-deleted-artifacts.png" alt-text="Screenshot of manage deleted artifacts." lightbox="./media/container-registry-soft-delete/soft-delete-manage-deleted-artifacts.png":::

1. In the row for the deleted artifact that you want to restore, select **Restore**.
1. In the **Restore Artifact** pane, select the tag to restore. You can only select one tag with which to restore your artifact. To recover additional tags, you must restore them separately.
1. Select **Restore**.

### Restore soft-deleted repositories

1. Go to your Azure Container Registry in the Azure portal.
1. In the service menu, under **Services**, select **Repositories**.
1. In **Repositories**, select a repository.
1. Select **Manage Deleted Respositories**.
1. In the row for the deleted repository that you want to restore, select **Restore**.
1. In the **Restore Artifact** pane, select the tag to restore. You can only select one tag with which to restore your repository. To recover additional tags, you must restore them separately.
1. Select **Restore**

### [Azure CLI](#tab/azure-cli)

### List soft-deleted artifacts

The [`az acr repository list-deleted`](/cli/azure/acr/repository#az-acr-repository-list-deleted) command enables fetching and listing of soft-deleted repositories. Use one or more of the following options to list soft-deleted artifacts.

List the soft-deleted repositories in the `MyRegistry` repository:

```azurecli-interactive
az acr repository list-deleted -n MyRegistry
```

List the soft-deleted manifests of the `hello-world` repository in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest list-deleted -r MyRegistry -n hello-world
```

List the soft-deleted tags of the `hello-world` repository in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest list-deleted-tags -r MyRegistry -n hello-world
```

Filter the soft-deleted tags of the `hello-world` repository to match tag `latest` in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest list-deleted-tags -r MyRegistry -n hello-world:latest
```

### Restore soft-deleted artifacts

The [`az acr manifest restore`](/cli/azure/acr/manifest#az-acr-manifest-restore) command restores a single deleted image by tag and digest. Use one or more of the following options to restore     soft-deleted artifacts.

Restore the `hello-world` repository image by tag `latest` and digest `sha256:abc123` in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest restore -r MyRegistry -n hello-world:latest -d sha256:abc123
```

Restore the most recently deleted manifest of tje `hello-world` repository by tag `latest` in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest restore -r MyRegistry -n hello-world:latest
```

*Force restore* overwrites the existing tag with the same name in the repository. To use force restore, add the specific argument `--force, -f`. If the soft delete policy is enabled when the force restore command is used, then the overwritten tag is soft-deleted according to the retention period.

Force restore the `hello-world` repository image by tag `latest` and digest `sha256:abc123` in the `MyRegistry` repository:

```azurecli-interactive
az acr manifest restore -r MyRegistry -n hello-world:latest -d sha256:abc123 -f
```

> [!IMPORTANT]
> Restoring a [manifest list](push-multi-architecture-images.md#manifest-list) doesn't recursively restore any underlying soft-deleted manifests.
>
> If you restore soft-deleted [ORAS artifacts](container-registry-manage-artifact.md), then restoring a subject doesn't recursively restore the referrer chain. Also, the subject has to be restored before you can restore a referrer manifest.

---

## Next steps

* Learn more about options to [delete images and repositories](container-registry-delete.md) in Azure Container Registry.
