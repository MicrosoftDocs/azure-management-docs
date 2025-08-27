---
title: Azure Linux Container Host for AKS support lifecycle
description: Learn about the support lifecycle for the Azure Linux Container Host for AKS.
ms.service: microsoft-linux
ms.custom: linux-related-content
author: suhuruli
ms.author: suhuruli
ms.topic: overview
ms.date: 09/29/2023
# Customer intent: As an AKS administrator, I want to understand the support lifecycle for Azure Linux Container Host, so that I can ensure my Kubernetes environment remains up-to-date and compliant with the necessary security and performance standards.
---

# Azure Linux Container Host support lifecycle

This article describes the support lifecycle for the Azure Linux Container Host for AKS.

> [!IMPORTANT]
> Microsoft is committed to meeting this support lifecycle and reserves the right to make changes to the support agreement and new scenarios that require modifications at any time with proper notice to customers and partners.

## Image releases

### Minor releases

At the beginning of each month, Mariner releases a minor image version containing medium, high, and critical package updates from the previous month. This release also includes minor kernel updates and bug fixes.

For more information on the CVE service level agreement (SLA), see [CVE infrastructure](./concepts-core.md#cve-infrastructure).

### Major releases

About every three years, Azure Linux releases a major image version containing new packages and package versions, an updated kernel, and enhancements to security, tooling, performance, and developer experience. Azure Linux releases a beta version of the major release about three months before the general availability (GA) release.

An Azure Linux release supports an AKS version throughout its standard community support. For example, Azure Linux 2.0 will support AKS v1.31 throughout the standard [support lifecycle](/azure/aks/supported-kubernetes-versions).

The following table outlines the first and last AKS release supported by each version of Azure Linux:

> [!NOTE]
> AKS customers will automatically move to Azure Linux 3.0 when upgrading their AKS versions from 1.31 to 1.32. No additional action is required.

| Azure Linux version | First supported AKS Version in Preview  |  First supported AKS version in GA   | Last supported AKS version  |
|---|---|---|---|
| Azure Linux 2.0   | 1.24  | 1.26  | 1.31 |
| Azure Linux 3.0   | 1.31  | 1.32  | Roughly three years after 1.32 release |

### AKS LTS Releases

- Azure Linux 2.0 is the default node OS for AKS versions v1.27 to v1.31 during both Standard and Long-Term Support (LTS). However, Azure Linux 2.0 goes End of Life during the LTS period of AKS v1.28–v1.31. To maintain support and security updates, customers running Azure Linux 2.0 on AKS v1.27–v1.31 LTS are requested to migrate to Azure Linux 3.0 by November 2025. After November 2025, Azure Linux 2.0 will no longer receive updates, security patches, or support, which may put your systems at risk. To migrate existing node pools from Azure Linux 2.0 to 3.0 while remaining on the same Kubernetes version, you can perform an in-place node pool update from `--os-sku AzureLinux` to `--os-sku AzureLinux3`. Your node pool will automatically reimage to the equivalent Azure Linux 3.0 node image. See documentation [here](/azure/aks/upgrade-os-version).
- Azure Linux 3.0 is the default node OS for AKS version v1.32 and above during both Standard and Long-Term Support (LTS).

| AKS version |  Azure Linux version during AKS Standard Support | Azure Linux version during AKS Long-Term Support  |
|---|---|---|
|1.27 | Azure Linux 2.0   | Azure Linux 2.0 |
| 1.28 - 1.31 | Azure Linux 2.0   | 	Azure Linux 2.0 (migrate to 3.0 by Nov 2025)   |
| 1.32+ | Azure Linux 3.0   | Azure Linux 3.0  |

## Next steps

- Learn more about [Azure Linux Container Host support](./support-help.md).
