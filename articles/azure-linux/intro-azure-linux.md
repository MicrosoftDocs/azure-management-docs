---
title: Introduction to the Azure Linux Container Host for AKS
description: Learn about the Azure Linux Container Host.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 12/12/2023
# Customer intent: As a cloud architect, I want to understand the Azure Linux Container Host for AKS, so that I can determine its benefits and compatibility for optimizing and securing my container workloads in the Azure environment.
---

# What is the Azure Linux Container Host for AKS?

The Azure Linux Container Host is an operating system image that's optimized for running container workloads on [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes). Microsoft maintains the Azure Linux Container Host and based it on [CBL-Mariner][cbl-mariner], an open-source Linux distribution created by Microsoft.

The Azure Linux Container Host is lightweight, containing only the packages needed to run container workloads. The container host is hardened based on significant validation tests and internal usage and is compatible with Azure agents. It provides reliability and consistency from cloud to edge across AKS, AKS for Azure Stack HCI (Hyper Converged Infrastructure), and Azure Arc. You can deploy Azure Linux node pools in a new cluster, add Azure Linux node pools to your existing clusters, or migrate your existing nodes to Azure Linux nodes.

To learn more about Azure Linux, see the [Azure Linux GitHub repository](https://github.com/microsoft/CBL-Mariner).

> [!NOTE]
> Azure Linux 3.0 is generally available on AKS.
>
> AKS customers will automatically move to Azure Linux 3.0 when upgrading their AKS versions from 1.31 to 1.32. No additional action is required.
> To learn more, see [Quickstart: Enable Azure Linux 3.0](./how-to-enable-azure-linux-3.md).

[!INCLUDE [azure-linux-retirement](./includes/azure-linux-retirement.md)]

## Azure Linux Container Host key benefits

The Azure Linux Container Host offers the following key benefits:

- **Small and lightweight**
  - The Azure Linux Container Host only includes the necessary set of packages needed to run container workloads. As a result, it consumes limited disk and memory resources and produces faster cluster operations (create, upgrade, delete, scale, node creation, and pod creation) on AKS.
  - Azure Linux has only 500 packages, and as a result takes up the least disk space by up to *5 GB* on AKS.
- **Secure supply chain**
  - The Linux and AKS teams at Microsoft build, sign, and validate the [Azure Linux Container Host packages][azure-linux-packages] from source, and host packages and sources in Microsoft-owned and secured platforms.
  - Before we release a package, each package runs through a full set of unit tests and end-to-end testing on the existing image to prevent regressions. The extensive testing, in combination with the smaller package count, reduces the chances of disruptive updates to applications.
  - Azure Linux has a focus on stability, often backporting fixes in core components like the kernel or openssl. It also limits substantial changes or significant version bumps to major release boundaries (for example, Azure Linux 2.0 to 3.0), which prevents customer outages.
- **Secure by default**
  - The Azure Linux Container Host has an emphasis on security. It follows the secure-by-default principles, including using a hardened Linux kernel with Azure cloud optimizations and flags tuned for Azure. It also provides a reduced attack surface and eliminates patching and maintenance of unnecessary packages.
  - Microsoft monitors the CVE (Common Vulnerabilities and Exposures) database and releases security patches monthly and critical updates within days if necessary.
  - Azure Linux passes all the [CIS Level 1 benchmarks][cis-benchmarks], making it the only Linux distribution on AKS that does so.
  - For more information on Azure Linux Container Host security principles, see the [AKS security concepts](/azure/aks/concepts-security).
- **Maintains compatibility with existing workloads**
  - All existing and future AKS extensions, add-ons, and open-source projects on AKS support Azure Linux. Extension support includes support for runtime components like Dapr, IaC tools like Terraform, and monitoring solutions like Dynatrace.
  - Azure Linux ships with `containerd` as its container runtime and the upstream Linux kernel, which enables existing containers based on Linux images (like Alpine) to work seamlessly on Azure Linux.

## What's new with Azure Linux 3.0?

Azure Linux 3.0 is generally available to use on AKS. Every three years, Azure Linux releases a new version of its operating system with upgrades to major components. 

The following table outlines information about the upgrades made to major components as part of this release: 

|Component| Version|
|--|--|
|Kernel| 6.6 |
|ContainerD| 2.0 |
|SystemD | v255 | 
|Crypto Library| [SymCrypt](https://github.com/microsoft/SymCrypt)

For information on Azure Linux 2.0 and Azure Linux 3.0 support lifecycles, see [Azure Linux Container Host support lifecycle](./support-cycle.md).

## Azure Linux Container Host supported GPU Virtual Machine Sizes

- [NVIDIA V100][nvidia-v100]
- [NVIDIA T4][nvidia-t4]
- [NVIDIA NC A100 V4][nvidia-nc-a100-v4]
- [NVIDIA NDasr A100 V4][nvidia-ndasr-a100-v4]
- [NVIDIA NDm A100 V4][nvidia-ndm-a100-v4]
- [NVIDIA NCads H100 V5][nvidia-ncads-h100-v5]

Get started deploying NVIDIA GPU workloads on AKS with Azure Linux [here][use-nvidia-gpu].

> [!NOTE]
> - If there are specific GPU series you'd like to see supported on AKS with Azure Linux, or if you have other prioritization requests, please file an issue in the [AKS GitHub repository](https://github.com/Azure/AKS/issues).

## Next steps

- Learn more about [Azure Linux Container Host core concepts](./concepts-core.md).
- Follow our tutorial to [Deploy, manage, and update applications](./tutorial-azure-linux-create-cluster.md).
- Get started by [Creating an Azure Linux Container Host for AKS cluster using Azure CLI](./quickstart-azure-cli.md).

<!-- LINKS - internal -->
[nvidia-v100]: /azure/virtual-machines/ncv3-series
[nvidia-t4]: /azure/virtual-machines/nct4-v3-series
[nvidia-nc-a100-v4]: /azure/virtual-machines/sizes/gpu-accelerated/nca100v4-series
[nvidia-ndasr-a100-v4]: /azure/virtual-machines/sizes/gpu-accelerated/ndasra100v4-series
[nvidia-ndm-a100-v4]: /azure/virtual-machines/sizes/gpu-accelerated/ndma100v4-series
[nvidia-ncads-h100-v5]: /azure/virtual-machines/sizes/gpu-accelerated/ncadsh100v5-series

[use-nvidia-gpu]: /azure/aks/use-nvidia-gpu
[cis-benchmarks]: /azure/aks/cis-azure-linux

<!-- LINKS - external -->
[cbl-mariner]: https://github.com/microsoft/CBL-Mariner
[azure-linux-packages]: https://packages.microsoft.com/cbl-mariner/2.0/prod/
