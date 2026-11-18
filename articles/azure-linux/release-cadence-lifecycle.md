---
title: Azure Linux release cadence and maintenance
description: Learn how Azure Linux 4 uses semiannual minor releases, monthly security updates, and package maintenance policies.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 10/30/2026
ai-usage: ai-assisted
---

# Azure Linux release cadence and maintenance

Azure Linux follows a predictable release model for planned package changes while continuing to deliver security and high-impact reliability fixes between releases. This article explains when changes occur and how package maintenance policies affect update behavior.

## Azure Linux lifecycle principles

The following table summarizes the four guiding principles that shape how Azure Linux releases, supports, and retires kernels and major packages:

| Principle | What it means for you |
| --------- | --------------------- |
| **Security first** | You receive security fixes monthly. Critical and High CVEs are fast-tracked, so you don't wait for the next monthly window to be protected. |
| **Predictability** | You see published start dates and end-of-support dates for every kernel and major package, so you can plan upgrades on your schedule instead of reacting to surprises. |
| **Controlled change** | When a new kernel or major component releases, you keep support for the previous version through an overlap window. You have time to validate and migrate before the older version is retired. |
| **Cloud-native lifecycle** | You get a distribution designed for environments that update continuously, not one pinned to a single kernel for long periods of time. |

## Major and minor releases

Azure Linux 4 is a major release. Semiannual minor releases, such as 4.1 and 4.2, introduce new features and package version upgrades. Azure Linux always aims to preserve application programming interface (API) and application binary interface (ABI) compatibility across these minor releases.

These releases provide regular points for you to review, validate, and adopt changes. Each package's maintenance policy guides its updates.

## Servicing between minor releases

Azure Linux ships security fixes monthly. Between minor releases, servicing also focuses on high-impact reliability fixes and patch-level package updates where the package's maintenance policy permits them.

Azure Linux might make a servicing exception between minor releases to address a security or high-impact reliability issue. Such changes aim to minimize disruption and preserve compatibility.

## Package maintenance policies

Each package follows a maintenance policy that balances compatibility and software freshness. Some packages receive backported fixes without version upgrades, while others track newer upstream releases.

Upgrade behavior depends on the package. A newer version might require explicit adoption, become the default while an older version remains available, or coexist with an older version before becoming the default.

## Plan for updates

Before you adopt a minor release:

1. Review the [Azure Linux release notes](https://github.com/microsoft/azurelinux/releases) and package changelogs.
1. Test your workload and its package dependencies in a nonproduction environment.
1. Rebuild and validate custom VM images.
1. Roll out the update by using your normal deployment and rollback process.

For package-level changes, use `dnf changelog <PACKAGE_NAME>` and `dnf info <PACKAGE_NAME>`.

## Related content

- [Manage packages with DNF](./manage-packages.md)
- [Azure Linux package repositories](./package-repositories.md)
- [Manage CVEs](./manage-common-vulnerabilities-exposures.md)
