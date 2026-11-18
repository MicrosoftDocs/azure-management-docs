---
title: Azure Linux package maintenance policies
description: Learn how Azure Linux package maintenance policies balance compatibility, security servicing, and access to newer software.
author: poorvinarang
ms.author: poorvinarang
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: concept-article
ms.date: 10/29/2026
ai-usage: ai-assisted
---

# Azure Linux package maintenance policies

Azure Linux balances application compatibility, security servicing, and access to newer software. Maintenance approaches vary by component, taking into account its role in the operating system and its upstream release lifecycle.

Azure Linux aims to preserve application programming interface (API) and application binary interface (ABI) compatibility within a major release. Security requirements can make this impractical and might require a newer component version or a compatibility-affecting change. Review release notes and validate your applications before adopting updates, particularly when changing runtime or toolchain versions.

## Package maintenance approaches

The following table describes general maintenance approaches, not fixed policy assignments or identical support periods for every package.

| Component needs | General maintenance approach |
| --- | --- |
| **Foundational component stability** | Generally favors backported security and high-impact reliability fixes, or targeted version updates where appropriate, with the aim of minimizing application disruption. |
| **Faster-moving components** | Plans version refreshes around minor releases to provide a predictable opportunity to adopt newer software, while allowing exceptions for security needs. |
| **Multiple maintained versions** | Can provide selected runtime or toolchain versions as separate packages so applications can adopt a newer version after validation. Availability and maintenance depend on the component. |

These approaches don't imply that every upstream release will be packaged or that every available version will be maintained for the full Azure Linux major-release lifecycle.

## Faster-moving components

Some components evolve more quickly than foundational operating-system libraries. Azure Linux takes those upstream lifecycles into account and plans to move selected components forward through semiannual minor releases, such as 4.1 and 4.2. These releases are intended to provide predictable points for version updates rather than track every upstream release.

Rust and Go are examples of faster-moving toolchains. A minor release can introduce a newer Rust compiler and Cargo, or a newer Go toolchain. Security requirements might require an update between minor releases.

Toolchain updates can change compiler diagnostics, generated binaries, or dependency behavior even when source compatibility is preserved. Record the compiler and RPM versions used for your builds, and rebuild and test applications after a toolchain update. Cargo lock files record application dependencies but don't fix the Rust compiler version.

## Components with multiple versions

Some applications need to remain on a particular runtime or toolchain release while others adopt newer versions. For components such as Node.js, .NET, OpenJDK, and LLVM, separately packaged versions can allow selected releases to be maintained concurrently. This doesn't mean that every version is available, that versions can always be installed together, or that they share the same maintenance period.

Where separately packaged versions are provided, a newer major version can be introduced alongside an existing version rather than requiring every application to migrate at once. Select the intended runtime or toolchain version explicitly, review its support information, and validate your application before switching. For LLVM and Clang, use matching compiler, linker, runtime, and C++ library packages from the same LLVM version.

Not every runtime follows this model. Azure Linux 4 provides its default Python runtime through the `python3` package; don't assume that additional Python minor versions are available as separate packages.

## Inspect available package versions

Query the repositories enabled on your system instead of relying on a fixed version list. Replace `<PACKAGE_NAME>` with the package you want to check, for example `llvm`:

```bash
dnf list --available <PACKAGE_NAME>
dnf info <PACKAGE_NAME>
dnf repoquery --info <PACKAGE_NAME>
dnf changelog <PACKAGE_NAME>
```

These commands show package availability, version, architecture, source repository, and changelog information. They don't identify a package's maintenance approach or establish a support commitment.

Repository purpose and the software's support terms also matter. A production repository location alone doesn't make a package supported; for example, the `builddeps` repository contains unsupported dependencies for building Azure Linux itself.

For repository boundaries, see [Azure Linux package repositories](./package-repositories.md).

Before adopting updates, review the [Azure Linux release notes](https://github.com/microsoft/azurelinux/releases) and the component's support information. For the release schedule, see [Azure Linux release cadence and lifecycle](./release-cadence-lifecycle.md).

## Related content

- [Azure Linux release cadence and lifecycle](./release-cadence-lifecycle.md)
- [Manage packages with DNF](./manage-packages.md)
- [Azure Linux package repositories](./package-repositories.md)
