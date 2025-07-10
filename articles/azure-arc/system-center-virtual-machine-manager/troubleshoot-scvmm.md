---
title: Troubleshoot SCVMM-specific Azure Arc resource bridge deployment errors
description: Learn how to troubleshoot SCVMM-specific Azure Arc resource bridge deployment errors. 
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
ms.topic: how-to 
ms.date: 07/10/2025
keywords: "VMM, Arc, Azure, System Center"
ms.custom:
  - build-2025
---

# Troubleshoot SCVMM-specific Azure Arc resource bridge deployment errors

This article provides troubleshooting steps that help you resolve the errors encountered during the deployment of Azure Arc resource bridge to onboard to Azure Arc-enabled SCVMM.

## CreateConfigKvaCustomerError

|**Error code ID**|**Cause and recommended action**|
|---|---|
|PSSessionAccessDenied|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2298170).|
|PSSessionGet-SCVMMServer|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2297694).|
|PSSessionMIResultFailed|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2298171).|

## KVAInvalidEntityCustomerError

|**Error code ID**|**Cause and recommended action**|
|---|---|
|ValidateInsufficientLibSharePermission|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2297975).|
|ValidateInsufficientPrivilege|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2298222).|
|ValidateVlanIDNotAvailableOnVMNetwork|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2297976).|

## PostOperationsError

|**Error code ID**|**Cause and recommended action**|
|---|---|
|PostOperationTimeout|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2300587).|
|PostOperationsErrorKubeadmControlPlane|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2301242).|

## UpgradeError

|**Error code ID**|**Cause and recommended action**|
|---|---|
|Upgrade_PreflightCheckErrorOnPrem|[Learn about the cause and recommended action](https://go.microsoft.com/fwlink/?linkid=2304710).|

## Next steps

- [Troubleshoot common deployment errors](/azure/azure-arc/resource-bridge/troubleshoot-resource-bridge).
- [Support matrix for Azure Arc-enabled System Center Virtual Machine Manager](support-matrix-for-system-center-virtual-machine-manager.md).
