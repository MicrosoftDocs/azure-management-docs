---
title: Build RPM packages for Azure Linux
description: Learn how to build RPM packages for Azure Linux and optionally add them to a custom image. Building custom RPMs lets you package proprietary software, apply patches to existing packages, or bundle internal tools for consistent deployment across your Azure Linux environments.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Build RPM packages for Azure Linux

This article explains how to build RPM packages for Azure Linux and optionally add them to a custom image. Building custom RPMs lets you package proprietary software, apply patches to existing packages, or bundle internal tools for consistent deployment across your Azure Linux environments.

Azure Linux supports two approaches. Select the option that best fits your workflow:

- **`mock` + Image Customizer**: Build RPMs in isolated chroot environments using `mock`, then customize an Azure Linux base image with Image Customizer. Best for developers who have an established RPM build workflow and need to customize Azure Linux virtual machine (VMs) images as a separate step.
- **azldev**: A single CLI tool that handles RPM builds and image customization as part of a unified, end-to-end workflow. Best for developers working within Azure Linux who want a streamlined experience from a single tool.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure Linux system (VM or container) or a Fedora/RHEL-based host.
- `sudo` access to install packages.
- A source RPM (`.src.rpm`) or a spec file and source tarball for the package you want to build.

## Build RPMs with mock

`mock` creates isolated chroot environments to build RPM packages without polluting your host system. Azure Linux publishes a `mock` RPM on [packages.microsoft.com (PMC)](https://packages.microsoft.com) that includes the Azure Linux build configuration, so you can start building immediately.

> [!NOTE]
> Currently, the recommended approach is to use the mock package shipped with Azure Linux 4.0, which includes pre-configured build profiles for Azure Linux. Upstream mock configuration isn't yet available.
>
> The default OS disk size for Azure Linux VMs might not be sufficient for building packages with mock. If you encounter "No space left on device" errors, you need to expand the OS disk. For more information, see [Expand virtual hard disks on a Linux VM](/azure/virtual-machines/linux/expand-disks?tabs=ubuntu).

### Install mock

Install `mock` from PMC using the following command:

```bash
sudo dnf install -y mock
```

### Verify the Azure Linux mock configuration

The `mock` package from PMC includes configuration profiles for Azure Linux. List the available configurations using the following command:

```bash
ls /etc/mock/azure-linux-4-*.cfg
```

You should see configuration files for Azure Linux (for example, `azure-linux-4-x86_64.cfg`).

### Build an RPM from a source RPM

Build an RPM from an existing source RPM using the following command:

```bash
mock /path/to/your-package.src.rpm
```

`mock` downloads the required build dependencies from the Azure Linux repositories, builds the package in a clean chroot, and places the output RPMs in `/var/lib/mock/azure-linux-4-x86_64/result/`.

### Build an RPM from a spec file

If you have a spec file and source tarball instead of a source RPM (SRPM), first create the SRPM and then rebuild it:

1. Create the SRPM using the following command:

    ```bash
    mock --buildsrpm --spec /path/to/your-package.spec --sources /path/to/sources/
    ```

    `mock` packages the spec file and source tarballs into an SRPM and places it in `/var/lib/mock/azure-linux-4-x86_64/result/`.

1. Build the RPM from the SRPM using the following command:

    ```bash
    mock /var/lib/mock/azure-linux-4-x86_64/result/your-package-*.src.rpm
    ```

1. After a successful build, find your RPMs in the result directory:

    ```bash
    ls /var/lib/mock/azure-linux-4-x86_64/result/*.rpm
    ```

## Build RPMs with azldev

For a more integrated development workflow, you can use **azldev**, a CLI tool from the [Azure Linux Dev Tools](https://github.com/microsoft/azure-linux-dev-tools) project. `azldev` simplifies common Azure Linux development tasks including building, testing, and iterating on packages.

## Add the RPM to an Azure Linux image with Image Customizer

After building your RPM, use [Image Customizer](./customize-images.md) to create a custom Azure Linux image that includes your package.

## Related content

To learn more about Azure Linux, see the following resources:

- [Certificate management in Azure Linux](./certificate-management.md)
- [Customize Azure Linux images with Image Customizer](./customize-images.md)
