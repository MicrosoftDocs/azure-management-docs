---
title: Introduction to Azure Linux
description: Learn about Azure Linux, the Microsoft-maintained, open-source Linux distribution optimized for running workloads on Azure.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/27/2026
---

# What is Azure Linux?

:::image type="icon" source="./media/azure-linux-logo-new.png":::

Azure Linux is a Microsoft-maintained, open-source Linux distribution built for Azure. It provides a lightweight, security-hardened operating system for virtual machines (VMs), containers, and Kubernetes clusters running on Azure infrastructure.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Why choose Azure Linux?

Azure Linux is built on a robust open-source foundation from the [Fedora](https://fedoraproject.org/) ecosystem and enhanced with Azure-specific innovations. This gives you the familiarity of RPM package ecosystem, while adding Azure-native security, compliance, and operational capabilities.

The following table highlights the key features and benefits of Azure Linux:

| Feature | Description |
| ------- | ----------- |
| **Hardened security posture** | Hardened from the kernel up with kernel lockdown, dm-verity for verified boot, SELinux for mandatory access controls, and FIPS 140-3 cryptographic modules. Critical and high CVEs are fast-tracked with [SLA-driven patch delivery](./manage-cves.md). |
| **Minimal footprint** | Ships only the packages required for cloud workloads, resulting in a smaller attack surface, fewer CVEs, and faster boot times and lower memory consumption. |
| **Azure-optimized kernel** | Purpose-built [kernel](./whats-new-azure-linux-4.md) (6.18 LTS) with Hyper-V guest drivers, Azure-specific performance tuning, and security hardening validated across Azure environments like Virtual Machines (VM) / Virtual Machine Scale Sets and AKS. Supports both [LTS and HWE kernel tracks](./release-cadence-lifecycle.md) for new hardware and GPU enablement. |
| **Supply chain security** | Critical CVEs are fast-tracked under [SLA-driven patch delivery](./manage-cves.md) with zero-reboot kernel livepatching. |
| **Native Azure integration** | Ships with validated Azure agents, VM extensions, and developer tools including Microsoft Defender for Cloud, Azure Monitor, Azure CLI, and more. See the full list in [Supported Azure services](./supported-azure-services.md). |
| **Consistent across environments** | Same operating system (OS) foundation, package set, and update mechanisms whether running on [AKS](./azure-linux-aks-overview.md), [Azure VM / Virtual Machine Scale Sets](./azure-linux-vm-vmss-overview.md), or [container images](./container-images-overview.md). See the [architecture overview](./architecture.md). |
| **Predictable lifecycle** | Clearly defined release cadence with LTS kernels maintained for four years and HWE kernels introduced annually for new hardware and GPU support. See [Release cadence and lifecycle](./release-cadence-lifecycle.md). |

<!-- Every package is GPG-signed with [signed SBOMs and SLSA provenance](../4.%20security-and-compliance/14.%20supply-chain-security.md).-->

## How to use Azure Linux

Azure Linux is available across multiple Azure environments, each optimized for a specific Azure scenario while sharing the same trusted OS foundation, security baseline, and Microsoft-managed lifecycle. For full details, see the [deployment options overview](./deployment-options.md).

| Platform | Best for | Description |
| -------- | -------- | ----------- |
| **[Azure Linux for VMs](./azure-linux-vm-vmss-overview.md)** | VM-based infrastructure | Run infrastructure workloads on a Microsoft-supported, Azure-optimized Linux distribution. From web apps to AI workloads, Azure Linux delivers consistency, security, and performance while integrating natively with Azure services, extensions, and tooling. |
| **[Azure Linux for AKS](./azure-linux-aks-overview.md)** | Kubernetes at scale | Already powering millions of cores running mission-critical workloads, Azure Linux is the container host for AKS. Offered as a standard host or as [Azure Container Linux (ACL)](./azure-container-linux-overview.md), an immutable variant for teams with stricter security and compliance requirements. Optimized for fast boot, predictable updates, and end-to-end Microsoft ownership from kernel to CVE response. |
| **[Azure Linux Container Images](./container-images-overview.md)** | Cloud-native applications | Build and run containerized applications on Microsoft-maintained base images to ensure consistency from build to production. Choose full-featured base images for flexibility, runtime-optimized images for Node.js, Python, Java, and .NET, or hardened distroless images for the smallest possible attack surface. |
| **[Azure Linux ISO](https://github.com/microsoft/azurelinux/releases)** | Local testing and evaluation | Download Azure Linux ISO artifacts from the [Azure Linux GitHub repository](https://github.com/microsoft/azurelinux/releases) for local testing and evaluation. |

> [!IMPORTANT]
> Azure Linux is open source, but Microsoft support and lifecycle commitments apply only to **Azure scenarios**. Specifically:
>
> - Azure Linux VM / Virtual Machine Scale Sets, AKS container host, and container images are supported.
> - Bare metal, ISO images, on-premises, and other clouds aren't supported.
> - Customized images are supported only when built on top of a prebuilt Azure Linux image (for example, with [Image Customizer](./customize-images.md)). Images built from scratch from the [Azure Linux sources on GitHub](https://github.com/microsoft/azurelinux) aren't covered.

## Azure Linux release notes

Each Azure Linux release includes security updates, package upgrades, and bug fixes. See the  [Azure Linux release notes](https://github.com/microsoft/azurelinux/releases) on GitHub for more information.

## Azure Linux 4.0

| Release | Release notes |
| ------- | ------------- |
| 4.0 Preview 1 (June 2026) | [Release notes](https://github.com/microsoft/azurelinux/releases/tag/4.0-preview.1) |

## Related content

- For a deeper understanding of the OS structure, see the [Architecture overview](./architecture.md).
- To get started with Azure Linux 4.0, see the [Quickstart guide](./get-started-azure-linux.md).
- To plan your update strategy, review the [Release cadence and lifecycle](./release-cadence-lifecycle.md).
