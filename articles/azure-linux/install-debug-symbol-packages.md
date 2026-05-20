---
title: Install Debug Symbol Packages on Azure Linux
description: Learn how to install debug symbol (`debuginfo`) packages on Azure Linux by enabling the dedicated `debuginfo` repository and using DNF to install the corresponding symbol packages for installed binaries.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Install debug symbol packages on Azure Linux

Debug symbol (`debuginfo`) RPMs contain the symbol tables that debuggers and profilers, such as `gdb`, use to resolve addresses to function names and source lines. Azure Linux publishes a `debuginfo` package alongside most binary packages, but the `debuginfo` repository isn't enabled by default. This avoids pulling large symbol packages onto production systems unless you explicitly opt in.

This article explains where Azure Linux publishes `debuginfo` packages, how to enable the `debuginfo` repository, and how to install symbol packages on a system that needs them.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure Linux system that you can authenticate to with shell access.
- A user account with `sudo` privileges, which is required to write `.repo` files under `/etc/yum.repos.d/` and to install packages.

## Where Azure Linux publishes `debuginfo` packages

Azure Linux publishes `debuginfo` RPMs to [packages.microsoft.com](https://packages.microsoft.com/) (PMC) in a dedicated location, organized by release version, release stage, and architecture. For Azure Linux 4.0, the layout is:

```text
https://packages.microsoft.com/azurelinux/4.0/<STAGE>/sdk/debuginfo/<ARCH>/
```

- **`<STAGE>`** is `prod` for production-validated symbols or `preview` for symbols that match preview packages.
- **`<ARCH>`** is the target architecture, such as `x86_64` or `aarch64`.

The version, stage, and architecture must match the system the symbols will be loaded on. Mismatched `debuginfo` packages don't resolve symbols correctly.

## Enable the `debuginfo` repository

1. To install `debuginfo` packages, configure DNF to read from the Azure Linux `debuginfo` repository for your release and architecture:

    ```bash
    dnf install azurelinux-repos-debuginfo
    ```

1. After the repository is in place, refresh the cached metadata so DNF picks up the new repository:

    ```bash
    sudo dnf clean all
    sudo dnf makecache
    ```

1. Confirm that the `debuginfo` repository is enabled using the following command:

    ```bash
    dnf repolist
    ```

    The output should list the new `debuginfo` repository alongside the default Azure Linux repositories.

## Install a debug symbol package

After the `debuginfo` repository is enabled, install symbol packages with `dnf install`. Debug symbol packages share the binary package's name and add a `-debuginfo` suffix. For example, to install symbols for `glibc`, run:

```bash
sudo dnf install glibc-debuginfo
```

To install symbols for several packages at once, list each `-debuginfo` package on the command line. For example:

```bash
sudo dnf install glibc-debuginfo openssl-debuginfo
```

After installation, debuggers and profilers automatically pick up the new symbols from `/usr/lib/debug/`.

> [!TIP]
> Match the `debuginfo` package version to the binary package version that's installed on the system. Mismatched versions cause the debugger to fall back to address-only stack traces. Run `dnf info <package>-debuginfo` to confirm the available version before you install.

## Related content

To learn more about Azure Linux package management, see [Package management on Azure Linux overview](./package-management-overview.md).
