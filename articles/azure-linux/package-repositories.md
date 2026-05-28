---
title: Overview of Azure Linux Package Repositories
description: Overview of how Azure Linux packages are published and organized in Microsoft's package repositories, including guidance on discovering available packages and understanding repository structure.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux package repositories overview

Azure Linux packages are published through [packages.microsoft.com](https://packages.microsoft.com/) (PMC), Microsoft's centralized service for hosting and distributing Linux packages. PMC provides authoritative, signed RPM repositories that Microsoft teams use to publish validated packages and deliver them to Azure, on-premises, and customer environments through standard Linux package managers such as DNF.

This article describes how to discover Azure Linux packages on PMC, how the Azure Linux repositories are organized, and what each repository contains.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Discover available packages

The supported and authoritative way to discover packages that are available for Azure Linux is to query repository metadata with a standard package manager such as `dnf`. For example:

```bash
dnf search <KEYWORD>
dnf list --available
```

- **`dnf search <KEYWORD>`**: Searches package names and summaries across every enabled repository for the given keyword. Use this when you know roughly what you want (for example, `dnf search nginx`) and need to find the matching package name.
- **`dnf list --available`**: Lists every package that's available in the enabled repositories but isn't currently installed. Pipe it through `grep` or `less` to scan large result sets, for example `dnf list --available | grep -i kernel`.

## Repository structure

For each major Azure Linux release, the repositories on PMC are organized indicate release, stage, name, and architecture. Within each release, repositories are nested in the following order:

1. **Azure Linux release version**: For example, `azurelinux/4.0`.
1. **Release stage**: For example, `beta`.
1. **Repository name**: For example, `base` or `microsoft`.`.
1. **Target architecture**: `x86_64` or `aarch64`.

### Repository layout

The repositories are structured as follows:

| Repository name | Purpose |
| --------------- | ------- |
| `base` | Official Azure Linux distribution packages. |
| `microsoft` | Supplementary software that's published by Microsoft. |

> [!TIP]
> Any component included in the `base` repository has its source package in a `/azurelinux/4.0/beta/base/srpms` repository.

> [!NOTE]
> Additional packages are published to `/azurelinux/4.0/beta/sdk` and `azurelinux/4.0/beta/sdk/srpms`. These packages are only provided to satisfy development requirements and aren't officially supported.

## Related content

To learn more about Azure Linux package management, see [Package management on Azure Linux overview](./package-management-overview.md).
