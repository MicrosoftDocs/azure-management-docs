---
title: Azure Linux Container Host for AKS basic core concepts
description: Learn the basic core concepts that make up the Azure Linux Container Host for AKS. 
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.topic: concept-article 
ms.date: 08/18/2024
ms.custom: template-concept, linux-related-content
# Customer intent: "As a cloud administrator, I want to understand the core concepts of the Azure Linux Container Host for AKS, so that I can effectively manage security vulnerabilities and optimize cluster performance."
---

# Core concepts for the Azure Linux Container Host for AKS

Microsoft Azure Linux is an open-source project maintained by Microsoft. Microsoft is responsible for the entire Azure Linux Container Host stack, from the Linux kernel to the Common Vulnerabilities and Exposures (CVE) infrastructure, support, and end-to-end validation. With Azure Linux, you can easily create an AKS cluster without worrying about details such as verification and critical security vulnerability patches from a third-party distribution.

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

## CVE infrastructure

One of Microsoft's responsibilities in maintaining the Azure Linux Container Host is establishing a process for CVEs. This process includes identifying applicable CVEs, publishing CVE fixes, and adhering to defined Service Level Agreements (SLAs) for package fixes. The Azure Linux team builds and maintains the SLA for package fixes for production purposes. For more information, see the [Azure Linux package repository structure](https://github.com/microsoft/CBL-Mariner/blob/2.0/toolkit/docs/building/building.md#packagesmicrosoftcom-repository-structure). 

For packages included in the Azure Linux Container Host, Azure Linux scans for security vulnerabilities twice daily by checking CVEs in the [National Vulnerability Database (NVD)](https://nvd.nist.gov/).

Azure Linux publishes CVEs in the [Security Update Guide (SUG) Common Vulnerability Reporting Framework (CVRF) API](https://github.com/microsoft/MSRC-Microsoft-Security-Updates-API). This API allows you to get detailed information about Microsoft security updates for security vulnerabilities investigated by the [Microsoft Security Response Center (MSRC)](https://www.microsoft.com/msrc). Through collaboration with MSRC, Azure Linux can quickly and consistently discover, evaluate, and patch CVEs, and contribute critical fixes back to upstream projects.

Microsoft takes high and critical CVEs seriously and may release them out-of-band as a package update before a new AKS node image becomes available. Medium and low CVEs are included in the next scheduled image release.

> [!NOTE]
> At this time, the scan results aren't published publicly.

## Feature additions and upgrades

Since Microsoft owns the entire Azure Linux Container Host stack, including the CVE infrastructure and other support streams, the process for submitting feature requests is streamlined. You can communicate directly with the Microsoft team that owns the Azure Linux Container Host, ensuring an accelerated process for submitting and implementing feature requests. If you have a feature request, file an issue in the [AKS GitHub repository](https://github.com/Azure/AKS/issues).

## Testing

Before an Azure Linux node image is released for production use, it undergoes a series of Azure Linux and AKS-specific tests to ensure the image meets AKS requirements. This quality testing approach helps catch and mitigate issues before deployment to your production nodes. Some tests focus on performance, measuring CPU, network, storage, memory, and cluster metrics such as cluster creation and upgrade times. These performance tests ensure that the Azure Linux Container Host doesn't experience performance regressions as Microsoft upgrades the image.

Additionally, the Azure Linux packages published to [packages.microsoft.com](https://packages.microsoft.com/cbl-mariner/) receive extra validation through comprehensive testing. Both the Azure Linux node image and packages run through a suite of tests that simulate an Azure environment. This testing includes Build Verification Tests (BVTs) that validate [AKS extensions and add-ons](/azure/aks/integrations) support on each Azure Linux Container Host release. Microsoft also tests patches against the current Azure Linux node image before release to ensure no regressions occur. This significantly reduces the likelihood of releasing a corrupt package to your production nodes.

## Next steps

This article covers core Azure Linux Container Host concepts such as CVE infrastructure and testing. For more information about Azure Linux Container Host concepts, see the following articles: 

- [Azure Linux Container Host overview](./intro-azure-linux.md)
- [Azure Linux Container Host for AKS packages](./concepts-packages.md)
