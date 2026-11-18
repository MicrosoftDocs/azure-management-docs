---
title: Overview of Azure Linux package repositories
description: Learn how to discover Azure Linux packages, understand the Azure Linux 4 repository structure, and distinguish production packages from preview and unsupported build dependencies.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 10/30/2026
ai-usage: ai-assisted
---

# Azure Linux package repositories overview

Azure Linux packages are published through [packages.microsoft.com](https://packages.microsoft.com/) (PMC), Microsoft's centralized service for hosting and distributing Linux packages. PMC provides authoritative repositories of signed RPM packages that Microsoft teams use to publish validated packages and deliver them to Azure, on-premises, and customer environments through standard Linux package managers such as DNF.

This article describes how to discover Azure Linux packages on PMC, how the Azure Linux 4 repositories are organized, and how repository purpose and release stage affect package support.

## Discover available packages

The supported and authoritative way to discover packages that are available for Azure Linux is to query repository metadata with a standard package manager such as `dnf`. For example:

```bash
dnf search <KEYWORD>
dnf list --available
```

- **`dnf search <KEYWORD>`**: Searches package names and summaries across every enabled repository for the given keyword. Use this when you know roughly what you want (for example, `dnf search nginx`) and need to find the matching package name.
- **`dnf list --available`**: Lists every package that's available in the enabled repositories but isn't currently installed. Pipe it through `grep` or `less` to scan large result sets, for example `dnf list --available | grep -i kernel`.

These commands query all enabled repositories. The `preview` channel is disabled by default.

PMC also provides a browsable directory listing for convenience. The HTML directory listing isn't a supported API. Use repository metadata rather than the directory listing for automation.

## Repository structure

Azure Linux 4 repositories on PMC are organized by major release, stage, repository name, and architecture:

1. **Azure Linux major release**: `azurelinux/4`. Azure Linux 4.0 and later 4.x image versions share this repository namespace.
1. **Release stage**: `prod` or `preview`.
1. **Repository name**: For example, `base`, `builddeps`, or `microsoft`.
1. **Target architecture**: `x86_64` or `aarch64`.

For example, the production repository for Azure Linux distribution packages on x86-64 systems is:

```text
https://packages.microsoft.com/azurelinux/4/prod/base/x86_64/
```

Azure Linux 4 uses `4`, not `4.0`, in production and preview repository paths. Use the major-version namespace rather than a separate repository path for each 4.x image version.

### Release stages

The release stage identifies the package channel, not a blanket support guarantee:

- **`prod`**: The production package channel. Use this channel for production packages, subject to the repository's purpose and the software's support terms.
- **`preview`**: A public channel for testing and validating packages. Enable this channel in nonproduction environments to evaluate packages before potential promotion to `prod`.

Publishing a repository under `prod` doesn't make unsupported packages supported. In particular, the `builddeps` repository remains unsupported.

### Repository layout

The repository names identify the following purposes. Architecture, source-package, and debug-symbol availability vary by repository.

| Repository name | Purpose | Support guidance |
| --------------- | ------- | ---------------- |
| `base` | Official Azure Linux distribution packages. | Use `prod/base` for production distribution packages. Azure Linux support applies to supported Azure scenarios; see [Azure Linux official support options](./support-options.md). |
| `builddeps` | Azure Linux-specific dependencies required to build Azure Linux itself. | Unsupported. These packages aren't provided as a general-purpose SDK or as supported production application packages. |
| `microsoft` | Supplementary Microsoft-authored software, including open-source and proprietary packages. | Consult the software's support documentation. Publication in this repository doesn't establish Azure Linux distribution support for every package. |
| `cloud-native` | Cloud Native Computing Foundation (CNCF) and cloud-native packages. | Consult the software's support documentation rather than assuming that PMC hosting establishes support. |

This layout applies to Azure Linux 4. Repository names for earlier Azure Linux releases differ.

### Source packages and debug symbols

Source packages and debug symbols use separate paths where they're available:

| Content | Path pattern |
| ------- | ------------ |
| Binary packages | `/azurelinux/4/<STAGE>/<REPOSITORY>/<ARCH>/` |
| Debug symbol packages | `/azurelinux/4/<STAGE>/<REPOSITORY>/debuginfo/<ARCH>/` |
| Source packages | `/azurelinux/4/<STAGE>/<REPOSITORY>/srpms/` |

Replace `<STAGE>` with `prod` or `preview`, `<REPOSITORY>` with the repository name, and `<ARCH>` with the target architecture. Not every repository provides every architecture or content type. Don't assume that proprietary software provides source code.

Source packages for components in the production `base` repository are published to `/azurelinux/4/prod/base/srpms/`. Source packages for components exclusive to `builddeps` use the corresponding `builddeps/srpms/` path.

Symbols for packages in `base` are published to `base/debuginfo/<ARCH>/`. Symbols for packages exclusive to `builddeps` are published to `builddeps/debuginfo/<ARCH>/`. If a source package produces subpackages in both repositories, its debug symbol package is published in the `base` debug symbol repository.

Match debug symbols to the installed binary package version, release stage, and architecture.

> [!NOTE]
> The `builddeps` repository contains packages that satisfy Azure Linux-specific build requirements. These packages are needed to build Azure Linux itself; the repository isn't a general-purpose SDK for application development. These packages aren't officially supported. This support limitation also applies to the corresponding `builddeps` debug symbol packages. Their presence in the `prod` channel doesn't change their support status.

## Related content

To learn more about Azure Linux package management, see [Package management on Azure Linux overview](./package-management-overview.md).
