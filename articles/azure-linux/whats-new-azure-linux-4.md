---
title: What's New in Azure Linux 4.0?
description: Learn about the new features and updates in Azure Linux 4.0, including the updated kernel, package manager, and core libraries.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: whats-new
ms.date: 04/26/2026
---

# What's new in Azure Linux 4.0?

Azure Linux 4.0 is the next major release of Microsoft's Linux distribution for Azure. It brings a newer kernel (6.18 LTS), a new package manager (dnf5), and updated core libraries across the stack. For a deeper look at how the operating system (OS) is structured, see the [Architecture overview](./architecture.md).

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Key components

The following table lists the core component versions included in Azure Linux 4.0:

| Component | Version | Highlights |
| --------- | ------- | ---------- |
| **Kernel** | 6.18 LTS | New hardware drivers, improved Hyper-V integration, GPU/AI accelerator support |
| **glibc** | 2.42 | Performance improvements in string operations, memory allocation, and thread handling |
| **OpenSSL** | 3.5.4 | Modern cryptographic algorithms, deprecated older cipher suites removed |
| **systemd** | 258.4 | Fast boot sequences, improved service management and logging |
| **Python 3** | 3.14.3 | JIT compiler, new syntax features, updated standard library |
| **bash** | 5.3.9 | Scripting improvements and bug fixes |
| **binutils** | 2.45.1 | Modern toolchain |
| **coreutils** | 9.7 | Security fixes and utility improvements |
| **curl** | 8.15.0 | Protocol updates and security patches |
| **util-linux** | 2.41.3 | Updated system utilities |
| **rpm** | 6.0.1 | Modernized database backend, improved signature verification |
| **Package manager** | dnf5 | Faster dependency resolution, lower memory usage |
| **FIPS 140-3** | In progress | Certification pending; use a certified release for FIPS-required workloads |

## Package manager: dnf5

Azure Linux 4.0 uses **dnf5** as its package manager. dnf5 is a complete rewrite that delivers faster dependency resolution and lower memory usage compared to older package managers.

If you have scripts, Dockerfiles, or CI pipelines that reference `tdnf` (the package manager used in earlier Azure Linux releases), update those references to `dnf5` or `dnf` before deploying on Azure Linux 4.0.

## Related content

- For a deeper understanding of the OS structure, see the [Architecture overview](./architecture.md).
- To get started with Azure Linux 4.0, see the [Quickstart guide](./get-started-azure-linux.md).
- To plan your update strategy, review the [Release cadence and lifecycle](./release-cadence-lifecycle.md).
