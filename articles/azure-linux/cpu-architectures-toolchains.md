---
title: CPU architectures and toolchain compatibility in Azure Linux 4
description: Learn how to identify Azure Linux CPU architectures, inspect compiler toolchains, and validate native applications when moving to Azure Linux 4.
author: poorvinarang
ms.author: poorvinarang
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: concept-article
ms.date: 10/12/2026
ai-usage: ai-assisted
---

# CPU architectures and toolchain compatibility in Azure Linux 4

Azure Linux 4 provides 64-bit VM images for x64 and ARM64. Use the architecture-specific image and package repositories that match your Azure VM size and workload.

## Supported architectures

| Architecture | Linux architecture value | Azure Marketplace SKU |
| --- | --- | --- |
| x64 or AMD64 | `x86_64` | `4` |
| ARM64 | `aarch64` | `4-arm64` |

Azure Linux 4 doesn't provide 32-bit VM images. Package architecture values can include `x86_64`, `aarch64`, and `noarch`.

Identify the architecture of a running system:

```bash
uname -m
rpm --eval '%{_arch}'
```

List the architecture of an installed package:

```bash
rpm -q --qf '%{NAME} %{ARCH}\n' <PACKAGE_NAME>
```

## Select an Azure VM architecture

Use an Azure VM size that supports the selected image architecture. List the available Azure Linux 4 image SKUs in a region:

```azurecli-interactive
az vm image list-skus \
  --location <REGION> \
  --publisher microsoftazurelinux \
  --offer azurelinux-4 \
  --output table
```

Don't copy binaries between x64 and ARM64 systems unless the artifact is architecture independent. Rebuild native applications and RPM packages for each target architecture.

## Inspect the compiler toolchain

Compiler and library package versions can change through Azure Linux servicing and minor releases. Query the system and enabled repositories for the versions you build against:

```bash
gcc --version
clang --version
ld --version
rpm -q gcc clang llvm binutils glibc glibc-devel
dnf info gcc clang llvm binutils glibc glibc-devel
```

Not every compiler package is installed in the base VM image. Install build dependencies from the appropriate package repository before compiling software.

## Validate native software when moving to Azure Linux 4

Azure Linux 4 updates the compiler, C library, linker, RPM tooling, and other build components from Azure Linux 3. Treat a major-version migration as a rebuild and validation event.

1. Rebuild native applications and RPM packages on Azure Linux 4.
1. Resolve compiler warnings and errors instead of assuming Azure Linux 3 build behavior.
1. Run application, performance, and integration tests on every target architecture.
1. Verify dynamically linked dependencies:

   ```bash
   ldd <BINARY_PATH>
   ```

1. Inspect an RPM's architecture and dependencies:

   ```bash
   rpm -qp --qf '%{NAME} %{VERSION}-%{RELEASE} %{ARCH}\n' <RPM_PATH>
   rpm -qpR <RPM_PATH>
   ```

1. Confirm that required packages are available in production repositories before deployment.

## Avoid architecture assumptions

Use the Marketplace image and Azure VM-size requirements as the compatibility boundary. Don't assume a specific x86-64 microarchitecture level, ARM instruction-set revision, or optional CPU extension unless the selected Azure VM size and your own application validation establish that requirement.

## Related content

- [Azure Linux for VMs and VM Scale Sets](./azure-linux-vm-vmss-overview.md)
- [Build RPM packages](./build-rpm-packages.md)
- [Azure Linux package repositories](./package-repositories.md)
