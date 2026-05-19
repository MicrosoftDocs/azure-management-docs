---
title: Overview of Azure Linux Deployment Options
description: Learn about the different deployment options for Azure Linux, including AKS container hosts, Azure Container Linux (ACL), virtual machines, and container images, and how to choose the right option for your workloads.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# Azure Linux deployment options overview

Azure Linux is available across multiple Azure experiences, each optimized for a specific scenario while sharing the same trusted Azure Linux foundation. Whether you're deploying virtual machines (VMs), running Kubernetes workloads, building containerized applications, or setting up a local developer environment, Azure Linux provides a consistent platform with Microsoft-managed engineering, security response, and Azure-native integration.

This article provides an overview of the different Azure Linux deployment options, explains the scenarios each option is optimized for, and helps you determine which deployment approach is best suited for your workloads.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Deployment options

The following table summarizes the primary Azure Linux deployment options, the scenarios they are optimized for, and the key features and benefits each option provides:

| Use case | Deployment option | Features and benefits |
| -------- | ----------------- | --------------------- |
| **Run Kubernetes workloads on a secure, Azure-native container host** | [Azure Linux container host for Azure Kubernetes Service (AKS)](./azure-linux-aks-overview.md) | A container host optimized for AKS that delivers fast boot times, predictable updates, and strong security defaults with tight AKS lifecycle integration. Microsoft owns the full stack from kernel to CVE response, simplifying operations and reducing customer overhead. |
| **Run containers on an immutable, hardened operating system (OS) with a minimal attack surface** | [Azure Container Linux (ACL)](./azure-container-linux-overview.md) | An immutable, container-optimized OS for AKS derived from Flatcar Container Linux with Azure Linux packages and servicing. The OS uses kernel-enforced immutability (dm-verity), SELinux, Trusted Launch with Secure Boot, and a minimal package set to reduce the attack surface and block tampering. Automated weekly image-based updates and supply-chain signing keep nodes current and verifiable. |
| **Run general-purpose VM workloads with Microsoft-supported Linux** | [Azure Linux for Virtual Machines](./azure-linux-vm-vmss-overview.md) | A Microsoft-supported, Azure-optimized Linux distribution for scalable web apps, infrastructure services, and AI workloads. It integrates natively with Azure services, extensions, and tooling, and includes Microsoft-managed security response and lifecycle support. |
| **Build and ship containerized applications on a consistent base image** | [Azure Linux Container Images](./container-images-overview.md) | Microsoft-maintained base images built from the Azure Linux supply chain that provide consistency from build to production. Customers can choose from full-featured base images for flexibility, runtime-optimized images for Node.js, Python, Java, and .NET, or hardened distroless images for the smallest possible attack surface. |

> [!IMPORTANT]
> Azure Linux is open source, but Microsoft support and lifecycle commitments apply only to **Azure scenarios**. Specifically:
>
> - Azure Linux VM/VMSS, AKS container host, and container images are supported.
> - Bare metal, ISO images, on-premises, and other clouds aren't supported.
> - Customized images are supported only when built on top of a prebuilt Azure Linux image (for example, with [Image Customizer](./customize-images.md)). Images built from scratch from the [Azure Linux sources on GitHub](https://github.com/microsoft/azurelinux) aren't covered.

## Related content

To get started with Azure Linux, see the [Azure Linux getting started guide](./get-started-azure-linux.md).
