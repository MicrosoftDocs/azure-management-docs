---
title: ACR Transfer Troubleshooting
description: Find solutions to common issues with Azure Container Registry (ACR) Transfer, including deployment failures, Key Vault access, and storage access.
author: rayoef
ms.author: rayoflores
ms.date: 02/11/2026
ms.topic: troubleshooting
ms.service: azure-container-registry
# Customer intent: As a DevOps engineer, I want to troubleshoot deployment failures and access issues with container registries, so that I can ensure smooth and efficient transfer of container images and artifacts.
---

# ACR Transfer troubleshooting

This article provides troubleshooting guidance for common issues with ACR Transfer. If you haven't completed the prerequisites, see [ACR Transfer prerequisites](./container-registry-transfer-prerequisites.md).

> [!IMPORTANT]
> ACR Transfer supports artifacts with layer sizes up to 8 GB due to technical limitations. If your transfer fails, verify that all artifact layers are within this size limit.

## Template deployment failures or errors

  * If a pipeline run fails, look at the `pipelineRunErrorMessage` property of the run resource.
  * For common template deployment errors, see [Troubleshoot ARM template deployments](/azure/azure-resource-manager/templates/template-tutorial-troubleshoot).

## Problems accessing Key Vault

> [!NOTE]
> Key Vault access issues only apply when using **SAS Token** storage access mode. If you are using **Managed Identity** storage access mode, Key Vault is not required for storage account access.

  * If your PipelineRun deployment fails with a `403 Forbidden` error when accessing Azure Key Vault, verify that your pipeline managed identity has adequate permissions.
  * A PipelineRun uses the ExportPipeline or ImportPipeline managed identity to fetch the SAS token secret from your Key Vault. ExportPipelines and ImportPipelines are provisioned with either a system-assigned or user-assigned managed identity. This managed identity is required to have `secret get` permissions on the Key Vault in order to read the SAS token secret. Ensure that an access policy for the managed identity was added to the Key Vault. For more information, reference [Give the ExportPipeline identity Key Vault policy access](./container-registry-transfer-cli.md#give-the-exportpipeline-identity-keyvault-policy-access) and [Give the ImportPipeline identity Key Vault policy access](./container-registry-transfer-cli.md#give-the-importpipeline-identity-keyvault-policy-access).

## Problems accessing storage

### SAS Token storage access mode

  * If you see a `403 Forbidden` error from storage, you likely have a problem with your SAS token.
  * The SAS token might not currently be valid. The SAS token might be expired or the storage account keys might have changed since the SAS token was created. Verify that the SAS token is valid by attempting to use the SAS token to authenticate for access to the storage account container. For example, put an existing blob endpoint followed by the SAS token in the address bar of a new Microsoft Edge InPrivate window or upload a blob to the container with the SAS token by using `az storage blob upload`.
  * The SAS token might not have sufficient Allowed Resource Types. Verify that the SAS token has been given permissions to Service, Container, and Object under Allowed Resource Types (`srt=sco` in the SAS token).
  * The SAS token might not have sufficient permissions. For export pipelines, the required SAS token permissions are Read, Write, List, and Add. For import pipelines, the required SAS token permissions are Read, Delete, and List. (The Delete permission is required only if the import pipeline has the `DeleteSourceBlobOnSuccess` option enabled.)
  * The SAS token might not be configured to work with HTTPS only. Verify that the SAS token is configured to work with HTTPS only (`spr=https` in the SAS token).

### Managed Identity storage access mode

  * If you see a `403 Forbidden` error from storage when using Managed Identity mode, verify that the pipeline's managed identity has the appropriate RBAC role (such as `Storage Blob Data Contributor`) assigned on the storage account.
  * If using a **system-assigned** managed identity, ensure that the identity is enabled on the pipeline resource and that the RBAC role assignment was created for the pipeline's system-assigned identity principal ID.
  * If using a **user-assigned** managed identity, ensure the correct identity resource ID was provided during pipeline creation, and that the RBAC role assignment was created for the user-assigned identity.
  * Verify that you are using API version `2025-06-01-preview` or later. The Managed Identity storage access mode is not available in earlier API versions.
  * When using Managed Identity mode, the `keyVaultUri` field in the pipeline response may appear as `null` or an empty string - this is expected behavior and not an error.
  * When using a user-assigned managed identity, the top-level `principalId` and `tenantId` in the pipeline identity response may appear as `null`. The actual identity details are found inside the `userAssignedIdentities` section. This is expected behavior.

## Problems with export or import of storage blobs

  * SAS token may be invalid, or may have insufficient permissions for the specified export or import run. See [Problems accessing storage](#problems-accessing-storage).
  * If using **Managed Identity** storage access mode, verify the managed identity has sufficient RBAC permissions on the storage account (such as `Storage Blob Data Contributor`).
  * Existing storage blob in source storage account might not be overwritten during multiple export runs. Confirm that the OverwriteBlobs option is set in the export run and the SAS token (or managed identity) has sufficient permissions.
  * Storage blob in target storage account might not be deleted after successful import run. Confirm that the DeleteSourceBlobOnSuccess option is set in the import run and the SAS token (or managed identity) has sufficient permissions.
  * Storage blob not created or deleted. Confirm that container specified in export or import run exists, or specified storage blob exists for manual import run.

## Problems with Source Trigger Imports

  * When using **SAS Token** storage access mode, the SAS token must have the List permission for Source Trigger imports to work. When using **Managed Identity** mode, the managed identity must have the appropriate RBAC role on the storage account that permits listing blobs.
  * Source Trigger imports will only fire if the Storage Blob has a Last Modified time within the last 60 days.
  * The Storage Blob must have a valid ContentMD5 property in order to be imported by the Source Trigger feature.
  * The Storage Blob must have the "category":"acr-transfer-blob" blob metadata in order to be imported by the Source Trigger feature. This metadata is added automatically during an Export Pipeline Run, but may be stripped when moved from storage account to storage account depending on the method of copy.

## AzCopy issues

  * See [Troubleshoot AzCopy issues](/azure/storage/common/storage-use-azcopy-configure).

## Artifacts transfer problems

  * Not all artifacts, or none, are transferred. Confirm spelling of artifacts in export run, and name of blob in export and import runs. Confirm you're transferring a maximum of 50 artifacts.
  * Pipeline run might not have completed. An export or import run can take some time.
  * For other pipeline issues, provide the deployment [correlation ID](/azure/azure-resource-manager/templates/deployment-history) of the export run or import run to the Azure Container Registry team.
  * To create ACR Transfer resources such as ExportPipelines, ImportPipelines, and PipelineRuns, the user must have at least `Container Registry Transfer Pipeline Contributor` access on the ACR subscription. Otherwise, they'll see authorization to perform the transfer denied or scope is invalid errors.

## Problems pulling the image in a physically isolated environment

  * If you see errors regarding foreign layers or attempts to resolve mcr.microsoft.com when attempting to pull an image in a physically isolated environment, your image manifest likely has nondistributable layers. Due to the nature of a physically isolated environment, these images will often fail to pull. You can confirm that this is the case by checking the image manifest for any references to external registries. If so, you'll need to push the nondistributable layers to your public cloud ACR prior to deploying an export pipeline-run for that image. For guidance on how to do this, see [How do I push nondistributable layers to a registry?](./container-registry-faq.yml#how-do-i-push-nondistributable-layers-to-a-registry-).

## Storage access mode warnings

  * If you see a CLI warning message about the `--storage-access-mode` parameter, explicitly specify `--storage-access-mode SasToken` to use SAS Token authentication, or `--storage-access-mode ManagedIdentity` to use Managed Identity authentication for storage access. Always specify this parameter when creating pipelines.
  * Breaking changes to the CLI extension follow the **May and November** schedule. Extensions can technically introduce breaking changes outside of this schedule, but the ACR Transfer team follows the May/November schedule.

