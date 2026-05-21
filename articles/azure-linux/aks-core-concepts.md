---
title: Azure Linux Container Host for Azure Kubernetes Service (AKS) Core Concepts
description: Learn about the core concepts of the Azure Linux Container Host when used with Azure Kubernetes Service (AKS), including CVE infrastructure, feature additions, upgrades, and testing.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: concept-article
ms.date: 04/28/2026
---

# Core concepts for the Azure Linux Container Host for Azure Kubernetes Service (AKS)

Microsoft Azure Linux is an open-source project maintained by Microsoft, which means that Microsoft is responsible for the entire Azure Linux Container Host stack, from the Linux kernel to the Common Vulnerabilities and Exposures (CVE) infrastructure, support, and end-to-end validation.

This article explains the core concepts of the Azure Linux Container Host for AKS, including how Microsoft manages CVEs, implements feature requests, performs upgrades, and validates images through testing.

## CVE infrastructure

The Azure Linux team scans packages shipped with the Azure Linux Container Host twice daily against the [National Vulnerability Database (NVD)](https://nvd.nist.gov/) and collaborates with the [Microsoft Security Response Center (MSRC)](https://www.microsoft.com/msrc) to evaluate, patch, and publish fixes. High and critical CVEs might be released out-of-band as a package update ahead of the next node image; medium and low CVEs are rolled into the next image release.

For the full CVE pipeline, published advisories, SLAs, and delivery models across all Azure Linux and Azure Container Linux deployment options, see [Manage CVEs on Azure Linux and Azure Container Linux](./manage-cves.md).

## Feature additions and upgrades

Given that Microsoft owns the entire Azure Linux Container Host stack, including the CVE infrastructure and other support streams, the process of submitting a feature request is streamlined. You can communicate directly with the Microsoft team that owns the Azure Linux Container Host, which ensures an accelerated process for submitting and implementing feature requests. If you have a feature request, please file an issue on the [AKS GitHub repository](https://github.com/Azure/AKS/issues).

## Testing

Before an Azure Linux node image is released for testing, it undergoes a series of Azure Linux and AKS specific tests to ensure that the image meets AKS requirements. This approach to quality testing helps catch and mitigate issues before they're deployed to your production nodes. Part of these tests are performance related, testing CPU, network, storage, memory, and cluster metrics such as cluster creation and upgrade times. This ensures that the performance of the Azure Linux Container Host doesn't regress as we upgrade the image.

We run Azure Linux node images and packages published to [packages.microsoft.com](https://packages.microsoft.com/azurelinux/) through a suite of tests that simulate an Azure environment, including Build Verification Tests (BVTs) that validate [AKS extensions and add-ons](/azure/aks/integrations) are supported on each release of the Azure Linux Container Host. We also test patches against the current Azure Linux node image before their release to ensure there are no regressions.

## Related content

This article covers some of the core Azure Linux Container Host concepts such as CVE infrastructure and testing.

For more information on the Azure Linux Container Host concepts, see the [Azure Linux Container Host overview](./azure-linux-overview.md).
