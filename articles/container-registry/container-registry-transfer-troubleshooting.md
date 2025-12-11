---
title: Troubleshooting Common Issues and Solutions ACR Transfer
description: Find solutions to common issues with Azure Container Registry (ACR) Transfer, including deployment failures, Key Vault access, and storage access.
author: rayoef
ms.author: rayoflores
ms.date: 10/31/2023
ms.topic: troubleshooting
ms.service: azure-container-registry
# Customer intent: As a DevOps engineer, I want to troubleshoot deployment failures and access issues with container registries, so that I can ensure smooth and efficient transfer of container images and artifacts.
---

# ACR Transfer troubleshooting

## Template deployment failures or errors

  * If a pipeline run fails, look at the `pipelineRunErrorMessage` property of the run resource.
  * For common template deployment errors, see [Troubleshoot ARM template deployments](/azure/azure-resource-manager/templates/template-tutorial-troubleshoot)

## Problems accessing Key Vault

  * If your pipelineRun deployment fails with a `403 Forbidden` error when accessing Azure Key Vault, verify that your pipeline managed identity has adequate permissions.
  * A pipelineRun uses the exportPipeline or importPipeline managed identity to fetch the SAS token secret from your Key Vault. ExportPipelines and importPipelines are provisioned with either a system-assigned or user-assigned managed identity. This managed identity is required to have `secret get` permissions on the Key Vault in order to read the SAS token secret. Ensure that an access policy for the managed identity was added to the Key Vault. For more information, reference [Give the ExportPipeline identity keyvault policy access](./container-registry-transfer-cli.md#give-the-exportpipeline-identity-keyvault-policy-access) and [Give the ImportPipeline identity keyvault policy access](./container-registry-transfer-cli.md#give-the-importpipeline-identity-keyvault-policy-access).

## Problems accessing storage

  * If you see a `403 Forbidden` error from storage, you likely have a problem with your SAS token.
  * The SAS token might not currently be valid. The SAS token might be expired or the storage account keys might have changed since the SAS token was created. Verify that the SAS token is valid by attempting to use the SAS token to authenticate for access to the storage account container. For example, put an existing blob endpoint followed by the SAS token in the address bar of a new Microsoft Edge InPrivate window or upload a blob to the container with the SAS token by using `az storage blob upload`.
  * The SAS token might not have sufficient Allowed Resource Types. Verify that the SAS token has been given permissions to Service, Container, and Object under Allowed Resource Types (`srt=sco` in the SAS token).
  * The SAS token might not have sufficient permissions. For export pipelines, the required SAS token permissions are Read, Write, List, and Add. For import pipelines, the required SAS token permissions are Read, Delete, and List. (The Delete permission is required only if the import pipeline has the `DeleteSourceBlobOnSuccess` option enabled.)
  * The SAS token might not be configured to work with HTTPS only. Verify that the SAS token is configured to work with HTTPS only (`spr=https` in the SAS token).

## Problems with export or import of storage blobs

  * SAS token may be invalid, or may have insufficient permissions for the specified export or import run. See [Problems accessing storage](#problems-accessing-storage).
  * Existing storage blob in source storage account might not be overwritten during multiple export runs. Confirm that the OverwriteBlob option is set in the export run and the SAS token has sufficient permissions.
  * Storage blob in target storage account might not be deleted after successful import run. Confirm that the DeleteBlobOnSuccess option is set in the import run and the SAS token has sufficient permissions.
  * Storage blob not created or deleted. Confirm that container specified in export or import run exists, or specified storage blob exists for manual import run.

## Problems with Source Trigger Imports

  * The SAS token must have the List permission for Source Trigger imports to work.
  * Source Trigger imports will only fire if the Storage Blob has a Last Modified time within the last 60 days.
  * The Storage Blob must have a valid ContentMD5 property in order to be imported by the Source Trigger feature.
  * The Storage Blob must have the "category":"acr-transfer-blob" blob metadata in order to be imported by the Source Trigger feature. This metadata is added automatically during an Export Pipeline Run, but may be stripped when moved from storage account to storage account depending on the method of copy.

## AzCopy issues

  * See [Troubleshoot AzCopy issues](/azure/storage/common/storage-use-azcopy-configure).

## Artifacts transfer problems

  * Not all artifacts, or none, are transferred. Confirm spelling of artifacts in export run, and name of blob in export and import runs. Confirm you're transferring a maximum of 50 artifacts.
  * Pipeline run might not have completed. An export or import run can take some time.
  * For other pipeline issues, provide the deployment [correlation ID](/azure/azure-resource-manager/templates/deployment-history) of the export run or import run to the Azure Container Registry team.
  * To create ACR Transfer resources such as `exportPipelines`,` importPipelines`, and `pipelineRuns`, the user must have at least `Container Registry Transfer Pipeline Contributor` access on the ACR subscription. Otherwise, they'll see authorization to perform the transfer denied or scope is invalid errors.

## Problems pulling the image in a physically isolated environment

  * If you see errors regarding foreign layers or attempts to resolve mcr.microsoft.com when attempting to pull an image in a physically isolated environment, your image manifest likely has non-distributable layers. Due to the nature of a physically isolated environment, these images will often fail to pull. You can confirm that this is the case by checking the image manifest for any references to external registries. If so, you'll need to push the non-distributable layers to your public cloud ACR prior to deploying an export pipeline-run for that image. For guidance on how to do this, see [How do I push non-distributable layers to a registry in the ACR FAQ](./container-registry-faq.yml)

