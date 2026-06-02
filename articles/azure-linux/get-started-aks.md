---
title: Get started with Azure Linux for Azure Kubernetes Service (AKS)
description: Learn about the Azure Linux deployment options available for Azure Kubernetes Service (AKS) and how to choose the right one for your workloads.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: get-started 
ms.date: 04/28/2026
---

# Get started with Azure Linux for Azure Kubernetes Service (AKS)

Azure Linux offers two container-optimized operating system (OS) options for Azure Kubernetes Service (AKS): the [Azure Linux Container Host](./azure-linux-aks-overview.md) and [Azure Container Linux (ACL)](./azure-container-linux-overview.md). Both are built on the Azure Linux foundation and maintained by Microsoft, but they differ in design philosophy, update model, and security posture.

This article helps you understand the differences between the two Azure Linux container-optimized OS options for AKS and provides guidance on selecting the right option based on your workload requirements.

## Azure Linux Container Host and Azure Container Linux (ACL) comparison

The following table summarizes the key differences between Azure Linux Container Host and Azure Container Linux (ACL) across several dimensions, including base OS, filesystem model, update mechanism, OS upgrade channels, and other operational and security characteristics:

| Aspect | Azure Linux Container Host | Azure Container Linux (ACL) |
| ------ | -------------------------- | --------------------------- |
| **Based on** | Azure Linux | Flatcar Container Linux with Azure Linux packages and servicing |
| **Filesystem model** | Mutable (standard read-write) | Immutable (`/usr` is read-only, protected by dm-verity) |
| **Update mechanism** | Package-based OS updates | Weekly image-based updates (no individual package updates) |
| **OS upgrade channels** | All AKS node OS upgrade channels supported | `NodeImage` and `None` only |
| **Package management** | RPM-based (`dnf`) | No package management (immutable OS) |
| **Customization** | Package-level customization possible | Workload customization primarily via containers; when host-level software delivery is needed, extensibility is provided via systemd extensions |
| **Security model** | Hardened Linux kernel with Azure optimizations, CIS Level 1 compliant | Kernel-enforced immutability (dm-verity), SELinux enforcing by default, Trusted Launch with Secure Boot required |
| **Boot requirements** | Standard AKS virtual machine (VM) sizes | Trusted Launch with Secure Boot and vTPM required (Generation 2 VMs only) |
| **Architecture support** | AMD64 and ARM64 | AMD64 and ARM64 |
| **GPU support** | All NVIDIA GPU AMD64 VM sizes on AKS | All NVIDIA GPU AMD64 VM sizes on AKS |
| **FIPS support** | Supported | Not yet supported |
| **Pod Sandboxing** | Supported | Not supported |
| **Artifact Streaming** | Supported | Not yet supported |
| **Confidential VMs** | Supported | Not yet supported |

## When to use Azure Linux Container Host

Use Azure Linux Container Host when you need:

- **Broad feature compatibility**: Full support for all AKS OS upgrade channels, FIPS, Pod Sandboxing, Artifact Streaming, and Confidential VMs.
- **Package-level flexibility**: The ability to install, update, or customize individual packages on your nodes.
- **General-purpose container workloads**: A lightweight, secure, and stable container host that works with existing AKS extensions, add-ons, and open-source tooling.

To get started with Azure Linux Container Host on AKS, see [Quickstart: Deploy an Azure Linux Container Host for AKS cluster using the Azure CLI](./quick-create-azure-linux-aks.md).

## When to use Azure Container Linux (ACL)

Use ACL when you need:

- **Tamper-proof infrastructure**: Kernel-enforced immutability with dm-verity ensures the OS image can't be modified at runtime.
- **Smallest attack surface**: Only the components required to run containers are included, reducing the number of packages and potential entry points for attackers.
- **Consistent, automated updates**: Weekly image-based auto updates keep every node at the same OS version with the latest security patches.
- **Strict security and compliance**: SELinux (enforcing by default) and Trusted Launch with Secure Boot (via a Unified Kernel Image) are enabled by default.

To get started with ACL on AKS, see [Quickstart: Deploy an Azure Container Linux (ACL) AKS cluster using the Azure CLI](./quick-deploy-azure-container-linux-aks-cli.md).

## Related content

- [Azure Linux Container Host overview](./azure-linux-aks-overview.md)
- [Azure Container Linux (ACL) overview](./azure-container-linux-overview.md)
- [Migrate existing nodes to ACL](./tutorial-migrate-azure-container-linux-aks.md)
