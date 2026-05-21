---
title: Azure Linux Container Host Support Lifecycle for Azure Kubernetes Service (AKS)
description: Learn about the support lifecycle for the Azure Linux Container Host when used with Azure Kubernetes Service (AKS), including minor and major image releases, and the mapping of Azure Linux versions to supported AKS versions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: overview
ms.date: 04/28/2026
---

# Azure Linux Container Host support lifecycle for Azure Kubernetes Service (AKS)

This article describes the support lifecycle for the Azure Linux Container Host for Azure Kubernetes Service (AKS).

> [!IMPORTANT]
> Microsoft is committed to meeting this support lifecycle and reserves the right to make changes to the support agreement and new scenarios that require modifications at any time with proper notice to customers and partners.

## Image releases

### Minor releases

At the beginning of each month, Azure Linux releases a minor image version containing medium, high, and critical package updates from the previous month. This release also includes minor kernel updates and bug fixes.

For more information on the Common Vulnerabilities and Exposures (CVE) service level agreement (SLA), see [CVE infrastructure](./manage-cves.md).

### Major releases

For information on Azure Linux major release cadence and lifecycle, see [Azure Linux release cadence and lifecycle](./release-cadence-lifecycle.md).

The following table outlines the first and last AKS release supported by each Azure Linux version:

| Azure Linux version | First supported preview AKS version | First supported GA AKS version | Last supported AKS version |
| ------------------- | ----------------------------------- | ------------------------------ | -------------------------- |
| Azure Linux 2.0 | 1.24 | 1.26 | 1.31 |
| Azure Linux 3.0 | 1.31 | 1.32 | Roughly three years after 1.32 release |

### AKS Long-Term Support (LTS) releases

| AKS version | Azure Linux version during AKS Standard support | Azure Linux version during AKS LTS |
| ----------- | ----------------------------------------------- | ---------------------------------- |
| 1.27 | Azure Linux 2.0 | Azure Linux 2.0 |
| 1.28 - 1.31 | Azure Linux 2.0 | Azure Linux 2.0 (migrate to 3.0 by Nov 2025) |
| 1.32+ | Azure Linux 3.0 | Azure Linux 3.0 |

## Related content

- [What is the Azure Linux Container Host for Azure Kubernetes Service (AKS)?](./azure-linux-aks-overview.md)
- [Deploy an Azure Linux Container Host for AKS cluster using the Azure CLI](./deploy-azure-linux-aks-cli.md)
