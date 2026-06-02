---
title: Azure Linux Release Cadence and Lifecycle
description: Learn about the guiding principles behind Azure Linux release cadence, lifecycle, and update strategy.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/26/2026
---

# Azure Linux release cadence and lifecycle

Azure Linux follows a lifecycle model designed to balance stability, security, and access to modern Linux capabilities. This page describes the guiding principles that shape how Azure Linux releases, supports, and retires kernels and packages.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Azure Linux lifecycle principles

The following table summarizes the four guiding principles that shape how Azure Linux releases, supports, and retires kernels and major packages:

| Principle | What it means for you |
| --------- | --------------------- |
| **Predictability** | You see published start dates and end-of-support dates for every kernel and major package, so you can plan upgrades on your schedule instead of reacting to surprises. |
| **Security first** | You receive upstream security fixes on defined SLAs. Critical and High CVEs are fast-tracked, so you don't wait for the next monthly window to be protected. |
| **Controlled change** | When a new kernel or major component releases, you keep support for the previous version through an overlap window. You have time to validate and migrate before the older version is retired. |
| **Cloud-native lifecycle** | You get an operating system (OS) designed for environments that update continuously, not one pinned to a single kernel for long periods of time. |

## What to expect

Azure Linux applies these principles across three areas:

- **Kernel lifecycle**: Azure Linux tracks upstream Linux Long-Term Support (LTS) kernels for production stability and introduces Hardware Enablement (HWE) kernels annually to support new hardware platforms, GPU drivers, and AI/ML accelerators. Azure Linux 4.0 ships with **kernel 6.18 LTS**.
- **Package updates**: Core OS components (kernel, glibc, OpenSSL, systemd) prioritize stability and receive security backports without disruptive version changes. Language runtimes and higher-level packages follow a structured refresh cadence with planned update windows.
- **Security servicing**: Azure Linux ships monthly security updates that backport CVE fixes to every supported package. Critical and High severity vulnerabilities are fast-tracked outside the regular monthly cycle. For details on the CVE pipeline, see [Manage CVEs](./manage-cves.md).

> [!NOTE]
> Specific support durations and update policies will be published ahead of General Availability (GA).

## Related content

- [Azure Linux architecture overview](./architecture.md)
- [Azure Linux release notes](./azure-linux-overview.md#azure-linux-release-notes)
