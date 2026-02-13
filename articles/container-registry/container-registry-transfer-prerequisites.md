---
title: Overview and Prerequisites for Transfer artifacts
description: Get an in-depth overview of ACR Transfer, including essential prerequisites and key features for managing and transferring container images in Azure Container Registry.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 02/11/2026
ms.service: azure-container-registry
---

# Customer intent: As a DevOps engineer, I want to learn how to configure the transfer of container images between Azure container registries so that I can efficiently manage and migrate artifacts across different environments and clouds.

# Transfer artifacts to another registry

This article shows how to transfer collections of images or other registry artifacts from one Azure container registry to another registry. The source and target registries can be in the same or different subscriptions, Active Directory tenants, Azure clouds, or physically disconnected clouds.

To transfer artifacts, you create a *transfer pipeline* that replicates artifacts between two registries by using [blob storage](/azure/storage/blobs/storage-blobs-introduction):

* Artifacts from a source registry are exported to a blob in a source storage account
* The blob is copied from the source storage account to a target storage account
* The blob in the target storage account gets imported as artifacts in the target registry. You can set up the import pipeline to trigger whenever the artifact blob updates in the target storage.

In this article, you create the prerequisite resources to create and run the transfer pipeline. The Azure CLI is used to provision the associated resources such as storage secrets. Azure CLI version 2.2.0 or later is recommended. If you need to install or upgrade the CLI, see [Install Azure CLI][azure-cli].

This feature is available in the **Premium** container registry service tier. For information about registry service tiers and limits, see [Azure Container Registry tiers](container-registry-skus.md).

> [!IMPORTANT]
> This feature is currently in preview. Previews are made available to you on the condition that you agree to the [supplemental terms of use][terms-of-use]. Some aspects of this feature may change prior to general availability (GA).

## Consider your use-case

Transfer is ideal for copying content between two Azure container registries in physically disconnected clouds, mediated by storage accounts in each cloud. If instead you want to copy images from container registries in connected clouds including Docker Hub and other cloud vendors, [image import](container-registry-import-images.md) is recommended.

## Prerequisites

* **Container registries** - You need an existing source registry with artifacts to transfer, and a target registry. ACR transfer is intended for movement across physically disconnected clouds. For testing, the source and target registries can be in the same or a different Azure subscription, Active Directory tenant, or cloud.

   If you need to create a registry, see [Quickstart: Create a private container registry using the Azure CLI](container-registry-get-started-azure-cli.md).
* **Storage accounts** - Create source and target storage accounts in a subscription and location of your choice. For testing purposes, you can use the same subscription or subscriptions as your source and target registries. For cross-cloud scenarios, typically you create a separate storage account in each cloud.

  If needed, create the storage accounts with the [Azure CLI](/azure/storage/common/storage-account-create?tabs=azure-cli) or other tools.

  Create a blob container for artifact transfer in each account. For example, create a container named *transfer*.

* **Key vaults** - Key vaults are needed to store SAS token secrets when using **SAS Token** storage access mode. Create the source and target key vaults in the same Azure subscription or subscriptions as your source and target registries. For demonstration purposes, the templates and commands used in this article also assume that the source and target key vaults are located in the same resource groups as the source and target registries, respectively. This use of common resource groups isn't required, but it simplifies the templates and commands used in this article.

   > [!NOTE]
   > If you plan to use **Managed Identity** storage access mode (`--storage-access-mode ManagedIdentity` or `-m ManagedIdentity`), key vaults and SAS token secrets are **not required** for storage account access. Instead, ensure the pipeline's managed identity has the appropriate RBAC role (such as `Storage Blob Data Contributor`) on the storage account.

   If needed, create key vaults with the [Azure CLI](/azure/key-vault/secrets/quick-create-cli) or other tools.

* **Environment variables** - For example commands in this article, set the following environment variables for the source and target environments. All examples are formatted for the Bash shell.
  ```console
  SOURCE_RG="<source-resource-group>"
  TARGET_RG="<target-resource-group>"
  SOURCE_KV="<source-key-vault>"
  TARGET_KV="<target-key-vault>"
  SOURCE_SA="<source-storage-account>"
  TARGET_SA="<target-storage-account>"
  ```

## Scenario overview

You create the following three pipeline resources for image transfer between registries. All are created using PUT operations. These resources operate on your *source* and *target* registries and storage accounts.

Storage authentication uses one of the following methods, configured via the `--storage-access-mode` (`-m`) parameter. Always specify this parameter when creating pipelines.

* **SAS Token** — The pipeline authenticates to the storage account using a shared access signature (SAS) token stored as a secret in Azure Key Vault. The pipeline's managed identity reads the SAS token secret from the key vault.
* **Managed Identity** — The pipeline authenticates to the storage account directly using an Entra managed identity (system-assigned or user-assigned). When using this mode, a Key Vault and SAS token secret are **not required** for storage account access. The `--secret-uri` parameter must be omitted.

> [!NOTE]
> The `--storage-access-mode` parameter requires API version `2025-06-01-preview` or later.

* **[ExportPipeline](./container-registry-transfer-cli.md#create-exportpipeline-with-the-acrtransfer-az-cli-extension)** - Long-lasting resource that contains high-level information about the *source* registry and storage account. This information includes the source storage blob container URI and, when using SAS token mode, the key vault managing the source SAS token.
* **[ImportPipeline](./container-registry-transfer-cli.md#create-importpipeline-with-the-acrtransfer-az-cli-extension)** - Long-lasting resource that contains high-level information about the *target* registry and storage account. This information includes the target storage blob container URI and, when using SAS token mode, the key vault managing the target SAS token. An import trigger is enabled by default, so the pipeline runs automatically when an artifact blob lands in the target storage container.
* **[PipelineRun](./container-registry-transfer-cli.md#create-pipelinerun-for-export-with-the-acrtransfer-az-cli-extension)** - Resource used to invoke either an ExportPipeline or ImportPipeline resource.
  * You run the ExportPipeline manually by creating a PipelineRun resource and specify the artifacts to export.
  * If an import trigger is enabled, the ImportPipeline runs automatically. It can also be run manually using a PipelineRun.
  * Currently a maximum of **50 artifacts** can be transferred with each PipelineRun.

### Things to know
* The ExportPipeline and ImportPipeline will typically be in different Active Directory tenants associated with the source and destination clouds. This scenario requires separate managed identities and key vaults for the export and import resources. For testing purposes, these resources can be placed in the same cloud, sharing identities.
* By default, the ExportPipeline and ImportPipeline templates each enable a system-assigned managed identity. When using SAS Token mode, this identity reads SAS token secrets from the key vault. When using Managed Identity mode, this identity authenticates directly to the storage account. The ExportPipeline and ImportPipeline templates also support a user-assigned identity that you provide.
* ACR Transfer supports two storage access modes: **SAS Token** and **Managed Identity**. When using Managed Identity mode, the pipeline authenticates directly to the storage account using an Entra managed identity, eliminating the need for key vaults and SAS token secrets for storage access. See [Storage access modes](#storage-access-modes) below for more details.

## Storage access modes

ACR Transfer supports two storage access modes for authenticating to storage accounts. Specify the mode using the `--storage-access-mode` (`-m`) parameter when creating export and import pipelines.

### SAS Token mode

In **SAS Token** mode, the pipeline uses a shared access signature (SAS) token, stored as a secret in Azure Key Vault, to authenticate to the storage account.

**Requirements for SAS Token mode:**
- An Azure Key Vault with a secret containing a valid SAS token
- The pipeline's managed identity must have `secret get` access policy permissions on the Key Vault
- The SAS token must have the appropriate permissions (see [Create and store SAS keys](#create-and-store-sas-keys) below)

### Managed Identity mode

In **Managed Identity** mode, the pipeline authenticates directly to the storage account using an Entra managed identity (system-assigned or user-assigned). This eliminates the need for Key Vault and SAS token management for storage access.

**Requirements for Managed Identity mode:**
- The pipeline's managed identity (system-assigned or user-assigned) must have the appropriate RBAC role on the storage account (for example, `Storage Blob Data Contributor`)
- The `--secret-uri` parameter must be omitted when creating the pipeline
- Requires API version `2025-06-01-preview` or later

## Create and store SAS tokens

> [!NOTE]
> The following SAS token setup is only required when using **SAS Token** storage access mode. If you plan to use **Managed Identity** storage access mode, skip this section and proceed to [Next steps](#next-steps).

Transfer uses shared access signature (SAS) tokens to access the storage accounts in the source and target environments. Generate and store tokens as described in the following sections.
> [!IMPORTANT]
> While ACR Transfer will work with a manually generated SAS token stored in a Key Vault Secret, for production workloads we *strongly* recommend using [Key Vault Managed Storage SAS Definition Secrets][kv-managed-sas] instead.


### Generate SAS token for export

Run the [az storage container generate-sas][az-storage-container-generate-sas] command to generate a SAS token for the container in the source storage account, used for artifact export.

*Recommended token permissions*: Read, Write, List, Add.

In the following example, command output is assigned to the EXPORT_SAS environment variable, prefixed with the '?' character. Update the `--expiry` value for your environment:

```azurecli
EXPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $SOURCE_SA \
  --expiry 2027-01-01 \
  --permissions alrw \
  --https-only \
  --output tsv)
```

### Store SAS token for export

Store the SAS token in your source Azure key vault using [az keyvault secret set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrexportsas \
  --value $EXPORT_SAS \
  --vault-name $SOURCE_KV
```

### Generate SAS token for import

Run the [az storage container generate-sas][az-storage-container-generate-sas] command to generate a SAS token for the container in the target storage account, used for artifact import.

*Recommended token permissions*: Read, Delete, List

In the following example, command output is assigned to the IMPORT_SAS environment variable, prefixed with the '?' character. Update the `--expiry` value for your environment:

```azurecli
IMPORT_SAS=?$(az storage container generate-sas \
  --name transfer \
  --account-name $TARGET_SA \
  --expiry 2027-01-01 \
  --permissions dlr \
  --https-only \
  --output tsv)
```

### Store SAS token for import

Store the SAS token in your target Azure key vault using [az keyvault secret set][az-keyvault-secret-set]:

```azurecli
az keyvault secret set \
  --name acrimportsas \
  --value $IMPORT_SAS \
  --vault-name $TARGET_KV
```

## Next steps

* Follow one of the below tutorials to create your ACR Transfer resources. For most non-automated use-cases, we recommend using the Az CLI Extension.

  * [ACR Transfer with Az CLI](./container-registry-transfer-cli.md)
  * [ACR Transfer with ARM templates](./container-registry-transfer-images.md)

<!-- LINKS - External -->
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/

<!-- LINKS - Internal -->
[azure-cli]: /cli/azure/install-azure-cli
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-storage-container-generate-sas]: /cli/azure/storage/container#az-storage-container-generate-sas
[kv-managed-sas]: /azure/key-vault/secrets/overview-storage-keys
