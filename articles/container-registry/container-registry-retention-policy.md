---
title: Set a Retention Policy to Retain Untagged Manifests
description: Learn how to enable a retention policy in your Premium Azure container registry, for automatic deletion of untagged manifests after a defined period.
ms.topic: how-to
ms.custom: devx-track-azurecli
author: rayoef
ms.author: rayoflores
ms.date: 12/18/2025
ms.service: azure-container-registry
# Customer intent: As a DevOps engineer, I want to set retention policies for untagged image manifests in a container registry, so that I can manage storage efficiently and avoid unnecessary costs.
---

# Set a retention policy to retain untagged manifests

Azure Container Registry gives you the option to set a *retention policy* for stored image manifests that don't have any associated tags. When you enable a retention policy, the registry automatically deletes untagged manifests after the number of days you set. This feature prevents the registry from filling up with unneeded artifacts and helps you save on storage costs.

To set a retention policy for untagged manifests, use the Azure portal or the Azure CLI. For the Azure CLI, run commands in either Azure Cloud Shell or a local installation with the latest version of the Azure CLI. To install or upgrade, see [How to install the Azure CLI][azure-cli].

A retention policy for untagged manifests is currently a preview feature of **Premium** container registries. For information about registry service tiers, see [Azure Container Registry service tiers](container-registry-skus.md).

> [!WARNING]
> Set a retention policy with care; deleted image data is **unrecoverable**. If you have systems that pull images by manifest digest (as opposed to image name), don't set a retention policy for untagged manifests. Deleting untagged images prevents those systems from pulling the images from your registry. Instead of pulling by manifest, consider adopting a *unique tag* scheme, a [recommended best practice](container-registry-image-tag-version.md#unique-tags).

## About the retention policy

Azure Container Registry does reference counting for manifests in the registry. When you untag a manifest, the registry checks to see if there's a retention policy. If a retention policy is enabled, and the `delete-enabled` attribute of the manifest is set to `true`, the registry schedules a manifest delete operation for a specific date and time, according to the number of days set in the retention policy.

As an example, suppose you untagged two manifests, one hour apart, in a registry with a retention policy of 30 days. The registry schedules delete operations for each of the manifests. Then, 30 days later, approximately one hour apart, the manifests are deleted, unless the retention policy is disabled before the scheduled deletion date.

You can exclude untagged manifests from being deleted by a retention policy by setting its `delete-enabled` attribute to `false`. For more information, see [Lock a container image in an Azure container registry](container-registry-image-lock.md).

> [!IMPORTANT]
> The retention policy applies only to untagged manifests with timestamps *after* the policy is enabled. Untagged manifests in the registry with earlier timestamps aren't subject to the policy. For other options to delete image data, see examples in [Delete container images in Azure Container Registry](container-registry-delete.md).
>
> Untagged manifests that use the media type `application/vnd.oci.image.index.v1+json` aren't supported by the retention policy. Only `v2` manifests are supported.

## Set a retention policy

By default, container registries don't have a retention policy for untagged manifests. To set or update a retention policy, use either the Azure CLI or the Azure portal. 

The default retention period for a retention policy is seven days, but you can specify any number of days between 0 and 365. After the retention period, the registry automatically deletes untagged manifests. Setting the value to 0 removes untagged manifests as soon as they become untagged.

# [Azure CLI](#tab/azure-cli)

To set or update a retention policy, run the [az acr config retention update][az-acr-config-retention-update] command in the Azure CLI.

The following example sets a retention policy of 30 days for untagged manifests in the registry *myregistry*:

```azurecli
az acr config retention update --registry myregistry --status enabled --days 30 --type UntaggedManifests
```

### Verify the retention policy

If you enable the preceding policy with a retention period of 0 days, you can quickly verify that untagged manifests are deleted:

1. Push a test image `hello-world:latest` image to your registry, or substitute another test image of your choice.
1. Untag the `hello-world:latest` image by using the [az acr repository untag][az-acr-repository-untag] command. This command doesn't delete the untagged manifest from the registry.

    ```azurecli
    az acr repository untag \
      --name myregistry --image hello-world:latest
    ```

1. Within a few seconds, because of your retention policy, the untagged manifest is deleted. Verify the deletion by using the [az acr manifest list-metadata][az-acr-manifest-list-metadata] command to list all manifests in the repository. If the test image was the only one in the repository, the repository itself is also deleted.

### Show the retention policy

To show the retention policy set in a registry, run the [az acr config retention show][az-acr-config-retention-show] command:

```azurecli
az acr config retention show --registry myregistry
```

# [Azure portal](#tab/azure-portal)

You can enable a registry's retention policy in the [Azure portal](https://portal.azure.com). 

1. Go to your Azure container registry.
1. In the service menu, under **Policies**, select **Retention (Preview)**.
1. In **Status**, select **Enabled**.
1. Specify a number of days between 0 and 365 to retain the untagged manifests.
1. Select **Save**.

:::image type="content" source="media/container-registry-retention-policy/set-retention-policy-untagged-manifests.png" alt-text="Screenshot showing the option to set a retention policy for untagged manifests for a container repository.":::

---

## Disable a retention policy

# [Azure CLI](#tab/azure-cli)

To disable a retention policy in a registry, run the [az acr config retention update][az-acr-config-retention-update] command and set `--status disabled`:

```azurecli
az acr config retention update \
  --registry myregistry --status disabled \
  --type UntaggedManifests
```

# [Azure portal](#tab/azure-portal)

### Disable a retention policy

1. Go to your Azure container registry.
1. In the service menu, under **Policies**, select **Retention (Preview)**.
1. In **Status**, select **Disabled**.
1. Select **Save**.

---

## Related content

* Learn about options to [delete images and repositories](container-registry-delete.md) in Azure Container Registry.
* Learn how to [automatically purge](container-registry-auto-purge.md) selected images and manifests from a registry.
* Learn about options to [lock images and manifests](container-registry-image-lock.md) in a registry.

<!-- LINKS - internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-acr-config-retention-update]: /cli/azure/acr/config/retention#az-acr-config-retention-update
[az-acr-config-retention-show]: /cli/azure/acr/config/retention#az-acr-config-retention-show
[az-acr-manifest-list-metadata]: /cli/azure/acr/manifest#az-acr-manifest-list-metadata
[az-acr-repository-untag]: /cli/azure/acr/repository#az-acr-repository-untag
