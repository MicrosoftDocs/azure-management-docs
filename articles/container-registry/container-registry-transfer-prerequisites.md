---
title: ACR Transfer Overview and Prerequisites
description: Get an in-depth overview of ACR Transfer, including essential prerequisites and key features for managing and transferring container images in Azure Container Registry.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.date: 02/11/2026
ms.service: azure-container-registry
# Customer intent: As a DevOps engineer, I want to learn how to configure the transfer of container images between Azure container registries through intermediary storage accounts so that I can efficiently manage and migrate artifacts across different environments and clouds.
---

# Transfer artifacts to another registry

## What is ACR Transfer?

ACR Transfer is an Azure Container Registry feature for transferring container images and other OCI artifacts between registries that may reside in different clouds or physically disconnected environments. The transfer works by using an intermediary storage account:

* Artifacts from a source registry are exported to a blob in a source storage account
* The blob is copied from the source storage account to a target storage account (across network boundaries if needed)
* The blob in the target storage account is imported as artifacts in the target registry

You can set up the import pipeline to trigger automatically whenever an artifact blob arrives in the target storage. This makes ACR Transfer ideal for air-gapped or cross-cloud scenarios where direct registry-to-registry connectivity is not available.

## How it works - pipelines and pipeline runs

ACR Transfer is built around three resource types that work together to replicate artifacts between registries:

* **ExportPipeline** - A long-lived resource associated with a source registry and a storage account. It defines where exported artifacts are written (the storage blob container) and how the pipeline authenticates to storage.

* **ImportPipeline** - A long-lived resource associated with a target registry and a storage account. It defines where to read artifact blobs from and how the pipeline authenticates to storage. By default, an ImportPipeline includes a source trigger that automatically imports new blobs as they arrive.

* **PipelineRun** - A one-time execution resource that triggers either an ExportPipeline or an ImportPipeline. For exports, you specify which artifacts (by tag or digest) to export. For imports, you specify which blob to import. Currently a maximum of **50 artifacts** can be transferred with each PipelineRun.

Think of pipelines as the configuration and pipeline runs as the execution.

> [!IMPORTANT]
> ACR Transfer supports artifacts with layer sizes up to 8 GB due to technical limitations.

## Storage access modes

ACR Transfer supports two storage access modes for authenticating pipelines to storage accounts:

* **SAS Token** - The pipeline authenticates to the storage account using a shared access signature (SAS) token stored as a secret in Azure Key Vault. The pipeline's managed identity reads the SAS token secret from the key vault.

* **Managed Identity** - The pipeline authenticates directly to the storage account using a Microsoft Entra managed identity (system-assigned or user-assigned). No Key Vault or SAS token is required for storage access.

You specify the mode using the `--storage-access-mode` parameter (CLI) or `storageAccessMode` property (ARM templates) when creating pipelines. The mode you choose determines which prerequisites you need to complete below.

### Architecture diagrams

The following diagrams illustrate the architecture for an export/import pipeline setup. This example demonstrates transferring images through an intermediary storage account for cross-cloud or cross-tenant image transfer scenarios without direct ACR-to-ACR access.

**Managed Identity mode:**

```
Source Cloud                                                                     Target Cloud
+-------------------------------------------------+                              +-------------------------------------------------+
|                                                 |                              |                                                 |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|  | Source ACR                        |          |                              |          | Target ACR                        |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|                   |                             |                              |                       ^                         |
|                   v                             |                              |                       |                         |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|  | ExportPipeline                    |          |                              |          | ImportPipeline                    |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|                   |                             |                              |                       ^                         |
|                   |                             |                              |                       |                         |
|                   |                             |                              |                       |                         |
|                   | Pipeline accesses           |                              |                       | Pipeline accesses       |
|                   | storage account directly    |                              |                       | storage account         |
|                   | (using Pipeline's Managed   |                              |                       | directly (using         |
|                   | Identity) to export image   |                              |                       | Pipeline's Managed      |
|                   |                             |                              |                       | Identity) to import     |
|                   | RBAC: Storage Blob Data     |                              |                       | image                   |
|                   | Contributor                 |                              |                       |                         |
|                   |                             |                              |                       | RBAC: Storage Blob Data |
|                   |                             |                              |                       | Contributor             |
|                   |                             |                              |                       |                         |
|                   v                             |                              |                       |                         |
|  +-----------------------------------+          |      Cross-Cloud or          |          +-----------------------------------+  |
|  | Storage Account                   |----------|----- Cross-Tenant -----------|--------->| Storage Account                   |  |
|  | (Blob Container)                  |          |      Syndication Process     |          | (Blob Container)                  |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|                                                 |                              |                                                 |
+-------------------------------------------------+                              +-------------------------------------------------+
```

**SAS Token mode:**

```
Source Cloud                                                                     Target Cloud
+-------------------------------------------------+                              +-------------------------------------------------+
|                                                 |                              |                                                 |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|  | Source ACR                        |          |                              |          | Target ACR                        |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|                   |                             |                              |                       ^                         |
|                   v                             |                              |                       |                         |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|  | ExportPipeline                    |          |                              |          | ImportPipeline                    |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|               |           |                     |                              |                   ^           |                 |
|               |           |                     |                              |                   |           |                 |
|               |           | Access Key Vault    |                              |                   |           | Access Key      |
|               |           | (using Pipeline's   |                              |                   |           | Vault (using    |
|               |           | Managed Identity)   |                              |                   |           | Pipeline's      |
|               |           | to fetch Storage    |                              |                   |           | Managed         |
|               |           | SAS Secret          |                              |                   |           | Identity) to    |
|               |           v                     |                              |                   |           | fetch Storage   |
|               |  +------------------------+     |                              |                   |           | SAS Secret      |
|               |  | Source Key Vault       |     |                              |                   |           v                 |
|               |  | (SAS Token Secret)     |     |                              |                   |  +------------------------+ |
|               |  +------------------------+     |                              |                   |  | Target Key Vault       | |
|               |                                 |                              |                   |  | (SAS Token Secret)     | |
|               | Pipeline accesses storage       |                              |                   |  +------------------------+ |
|               | account using SAS Secret to     |                              |                   |                             |
|               | export image                    |                              |                   | Pipeline accesses storage   |
|               |                                 |                              |                   | account using SAS Secret    |
|               |                                 |                              |                   | to import image             |
|               |                                 |                              |                   |                             |
|               v                                 |                              |                   |                             |
|  +-----------------------------------+          |      Cross-Cloud or          |          +-----------------------------------+  |
|  | Storage Account                   |----------|----- Cross-Tenant -----------|--------->| Storage Account                   |  |
|  | (Blob Container)                  |          |      Syndication Process     |          | (Blob Container)                  |  |
|  +-----------------------------------+          |                              |          +-----------------------------------+  |
|                                                 |                              |                                                 |
+-------------------------------------------------+                              +-------------------------------------------------+
```

## Consider your use-case

Transfer is ideal for copying content between two Azure container registries in physically disconnected clouds, mediated by storage accounts in each cloud.

For other scenarios, consider these alternatives:
* If you want to copy images from container registries in connected clouds including Docker Hub and other cloud vendors, [image import](container-registry-import-images.md) is recommended.
* If you want to cache images from public registries to improve pull performance, see [Artifact cache](artifact-cache-overview.md).

## Prerequisites

### Common prerequisites (both storage access modes)

The following resources are required regardless of which storage access mode you choose:

* **Container registries** - You need an existing source registry with artifacts to transfer, and a target registry. Both registries must be in the **Premium** service tier. ACR transfer is intended for movement across physically disconnected clouds. For testing, the source and target registries can be in the same or a different Azure subscription, Microsoft Entra tenant, or cloud.

   For information about registry service tiers and limits, see [Azure Container Registry tiers](container-registry-skus.md). If you need to create a registry, see [Quickstart: Create a private container registry using the Azure CLI](container-registry-get-started-azure-cli.md).

* **Storage accounts** - Create source and target storage accounts in a subscription and location of your choice. For testing purposes, you can use the same subscription or subscriptions as your source and target registries. For cross-cloud scenarios, typically you create a separate storage account in each cloud.

  If needed, create the storage accounts with the [Azure CLI](/azure/storage/common/storage-account-create?tabs=azure-cli) or other tools. Create a blob container for artifact transfer in each account. For example, create a container named *transfer*.

### Additional prerequisites for SAS Token mode

If you plan to use **SAS Token** storage access mode, you also need:

* **Key vaults** - Create source and target key vaults to store SAS token secrets. Create them in the same Azure subscription or subscriptions as your source and target registries. For demonstration purposes, the templates and commands used in this article also assume that the source and target key vaults are located in the same resource groups as the source and target registries, respectively. This use of common resource groups isn't required, but it simplifies the templates and commands used in this article.

   If needed, create key vaults with the [Azure CLI](/azure/key-vault/secrets/quick-create-cli) or other tools.

* **SAS tokens** - You'll need to generate SAS tokens for the storage account containers and store them in the key vaults. See [Create and store SAS tokens](#create-and-store-sas-tokens) below for detailed steps.

* **Key Vault access** - The pipeline's managed identity must have `secret get` access policy permissions on the Key Vault to read the SAS token secret.

### Additional prerequisites for Managed Identity mode

If you plan to use **Managed Identity** storage access mode, you also need:

* **RBAC role assignment** - The pipeline's managed identity (system-assigned or user-assigned) must have the appropriate RBAC role on the storage account. For example, assign the `Storage Blob Data Contributor` role to allow the pipeline to read and write blobs.

* **API version requirement** - Managed Identity mode requires API version `2025-06-01-preview` or later (for ARM templates) or Azure CLI extension `acrtransfer` version 2.0.0 or later (for CLI).

> [!NOTE]
> When using Managed Identity mode, Key Vaults and SAS token secrets are **not required** for storage account access. The pipeline authenticates directly to the storage account using its managed identity.

### Environment variables

For example commands in this article, set the following environment variables for the source and target environments. All examples are formatted for the Bash shell.

```console
SOURCE_RG="<source-resource-group>"
TARGET_RG="<target-resource-group>"
SOURCE_KV="<source-key-vault>"  # Only needed for SAS Token mode
TARGET_KV="<target-key-vault>"  # Only needed for SAS Token mode
SOURCE_SA="<source-storage-account>"
TARGET_SA="<target-storage-account>"
```

## Things to know

* The ExportPipeline and ImportPipeline will typically be in different Microsoft Entra tenants associated with the source and destination clouds. This scenario requires separate managed identities and key vaults (when using SAS Token mode) for the export and import resources. For testing purposes, these resources can be placed in the same cloud, sharing identities.
* By default, the ExportPipeline and ImportPipeline templates each enable a system-assigned managed identity. The templates also support a user-assigned identity that you provide.
  * When using **SAS Token mode**, the identity reads SAS token secrets from the key vault
  * When using **Managed Identity mode**, the identity authenticates directly to the storage account

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

Now that you've completed the prerequisites, follow one of the tutorials below to create your ACR Transfer pipelines and pipeline runs:

* [ACR Transfer with Az CLI](./container-registry-transfer-cli.md) - Recommended for most use cases
* [ACR Transfer with ARM templates](./container-registry-transfer-arm-template.md) - For automation and infrastructure-as-code scenarios

If you encounter issues, see [ACR Transfer Troubleshooting](./container-registry-transfer-troubleshooting.md) for guidance.

<!-- LINKS - Internal -->
[az-keyvault-secret-set]: /cli/azure/keyvault/secret#az-keyvault-secret-set
[az-storage-container-generate-sas]: /cli/azure/storage/container#az-storage-container-generate-sas
[kv-managed-sas]: /azure/key-vault/secrets/overview-storage-keys
