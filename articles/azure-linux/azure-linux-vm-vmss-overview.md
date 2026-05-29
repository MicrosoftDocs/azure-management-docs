---
title: Overview of Deploying Azure Linux on Azure Virtual Machines (VMs) and Virtual Machine Scale Sets
description: This article provides an overview of deploying Azure Linux on Azure Virtual Machines (VMs) and Virtual Machine Scale Sets, including supported images and VM sizes.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Deploying Azure Linux on Azure VMs and Virtual Machine Scale Sets

This article provides an overview of how to deploy Azure Linux on Azure Virtual Machines (VMs) and Virtual Machine Scale Sets.

## Overview of Azure Linux

Azure Linux is a lightweight, security-hardened Linux distribution with a small on-disk footprint, built and maintained for Azure workloads. You can run the core image as-is or use it as the starting point for a customized build tailored to your workload.

The core image is published to Azure Marketplace with a curated set of base packages. To customize it, add packages from [packages.microsoft.com](https://packages.microsoft.com) and VM extensions from the Azure Marketplace, or use [Image Customizer](./customize-images.md) to automate the process. Once your image is ready, store it in an Azure Compute Gallery and deploy it consistently across your environment. For more information, see [Package management](./package-management-overview.md).

For a single workload, deploy Azure Linux to an individual VM. To scale out automatically based on demand, use virtual machine scale sets instead.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Available images

Azure Linux 4.0 images are published to Azure Marketplace under the `microsoftazurelinux` publisher. Use the URN that matches your architecture and VM generation:

| Architecture | Generation | URN |
| ------------ | ---------- | --- |
| x86_64 | Gen2 | `microsoftazurelinux:azurelinux-4:4:latest` |
| x86_64 | Gen1 | Not available for Preview |
| ARM64 | Gen2 | `microsoftazurelinux:azurelinux-4:4-arm64:latest` |

You can browse these images in the Azure portal using the guidance in the [Launch an Azure Linux VM](./create-vm-azure-linux-4-0.md) quickstart or query them programmatically using the following Azure CLI commands:

```azurecli-interactive
# List all offers from the publisher
az vm image list-offers --publisher microsoftazurelinux --location eastus

# List all SKUs for an offer
az vm image list-skus --publisher microsoftazurelinux --offer azurelinux-4 --location eastus

# List all versions of a SKU
az vm image list --publisher microsoftazurelinux --offer azurelinux-4 --sku 4 --all

# Show details for a specific image version
az vm image show --urn microsoftazurelinux:azurelinux-4:4:latest
```

## Supported VM sizes

Azure Linux runs on a broad range of Azure VM sizes, including:

- General Purpose Compute SKUs (for example, A, Dav#, Dv#-Series)
- SGX SKU (DCv2)
- Memory Optimized (E-Series)
- Compute Optimized (F-series)
- Storage Optimized (L-series)
- ARM64 SKU (v5 Ampere and v6 Cobalt)
- GPU (NVIDIA v100, T4, NC A100 V4, NDasr A100 V4, NDm A100 V4, NCads H100 V5, ND-H100-v5, ND-H200-v5, ND GB200-v6)

## Disk size

The Azure Linux VM disk size is set at _five GBs_. If you need to expand the Azure Linux disk size, use the parameter `--os-disk-size-gb <NUMBER_OF_GB_DESIRED>` or follow the guidance in [Expand virtual hard disks on a Linux VM](/azure/virtual-machines/linux/expand-disks).

## Related content

- [Create an Azure Linux VM](./create-vm-azure-linux-4-0.md)
- [Update an Azure Linux VM](./update-vm-azure-linux.md)
