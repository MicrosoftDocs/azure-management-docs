---
title: Overview of Package Management on Azure Linux
description: Overview of how package management works on Azure Linux, including DNF 5, RPM packages, and compatibility with YUM-era tooling.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Package management on Azure Linux overview

Azure Linux uses **DNF 5** as its package manager and ships software as **RPM packages**. This article covers the specifics of how Azure Linux is packaged, what's new in DNF 5, and where to look when paths or tools still carry YUM-era names.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## DNF 5

DNF 5 is the latest major release of [DNF](https://github.com/rpm-software-management/dnf5), the package manager maintained by the upstream RPM software management community. Compared to DNF 4, DNF 5 provides:

- Faster dependency resolution.
- Reduced memory usage.
- An improved internal architecture.
- Full backward compatibility with existing YUM repository configurations.

The command-line surface is intentionally close to `yum` and `dnf` 4, so existing scripts and CI pipelines generally work without changes. The `dnf` and `yum` commands on Azure Linux are provided by DNF 5.

## RPM packages

Software on Azure Linux is delivered as RPM packages. Each `.rpm` file bundles:

- Binaries, libraries, scripts, and other payload files.
- Configuration files marked with `%config` so they survive upgrades.
- Metadata: name, version, release, architecture, dependencies, file list, signatures, and changelog.

DNF resolves dependencies, fetches the required `.rpm` files from configured repositories, verifies their signatures, and hands them to the `rpm` library to install or upgrade. You can interact with installed packages directly using `rpm` (for example, `rpm -qa`, `rpm -qf <path>`, `rpm -V <pkg>`), but for installs and upgrades use DNF so that dependency resolution stays correct.

## YUM, DNF, and DNF 5

RPM-based distributions have moved through three generations of front-end tooling:

```text
YUM → DNF → DNF 5
```

Each generation kept the on-disk repository format compatible, so you still see paths and names like `/etc/yum.repos.d/` and `yum.conf` on current Azure Linux systems. They're aliases into DNF 5, not separate tools, and you can use either name in scripts.

## Related content

For more information about DNF 5 and the RPM ecosystem, see the following resources:

- [DNF 5 documentation](/guides/content/editing-an-existing-page): Official user and reference documentation for the DNF 5 package manager.
- [RPM software management on GitHub](https://github.com/rpm-software-management): Upstream organization that maintains DNF, RPM, and related tooling.
