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

The Azure Linux Container Host is an operating system image that's optimized for running container workloads on [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes). Microsoft maintains the Azure Linux Container Host, which is based on [CBL-Mariner][cbl-mariner], an open-source Linux distribution created by Microsoft.

The Azure Linux Container Host is lightweight, containing only the packages needed to run container workloads. Microsoft has hardened the container host through extensive validation tests and internal usage. The container host is compatible with Azure agents and provides reliability and consistency from cloud to edge across AKS, AKS for Azure Stack HCI (Hyper Converged Infrastructure), and Azure Arc. 

You have several deployment options: deploy Azure Linux node pools in a new cluster, add Azure Linux node pools to your existing clusters, or migrate your existing nodes to Azure Linux nodes.

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
  - The Azure Linux Container Host only includes the necessary packages to run container workloads. This minimal footprint consumes limited disk and memory resources, resulting in faster cluster operations on AKS. These operations include creating, upgrading, deleting, and scaling clusters, as well as creating nodes and pods.
  - Azure Linux contains only 500 packages and uses up to *5 GB* less disk space on AKS compared to other distributions.
- **Secure supply chain**
  - The Linux and AKS teams at Microsoft build, sign, and validate the [Azure Linux Container Host packages][azure-linux-packages] from source. Microsoft hosts both packages and sources on Microsoft-owned and secured platforms.
  - Before Microsoft releases a package, each package undergoes a full set of unit tests and end-to-end testing on the existing image to prevent regressions. This extensive testing, combined with the smaller package count, reduces the likelihood of disruptive application updates.
  - Azure Linux focuses on stability by backporting fixes to core components like the kernel and OpenSSL. The distribution limits substantial changes or significant version bumps to major release boundaries (for example, Azure Linux 2.0 to 3.0), which helps prevent customer outages.
- **Secure by default**
  - The Azure Linux Container Host emphasizes security and follows secure-by-default principles. The distribution uses a hardened Linux kernel with Azure cloud optimizations and flags tuned for Azure. The minimal package set provides a reduced attack surface and eliminates the need to patch and maintain unnecessary packages.
  - Microsoft monitors the Common Vulnerabilities and Exposures (CVE) database and releases security patches monthly. Critical updates are released within days when necessary.
  - Azure Linux passes all the [CIS Level 1 benchmarks][cis-benchmarks], making it the only Linux distribution on AKS to achieve this certification.
  - For more information about Azure Linux Container Host security principles, see [AKS security concepts](/azure/aks/concepts-security).
- **Maintains compatibility with existing workloads**
  - All existing and future AKS extensions, add-ons, and open-source projects support Azure Linux. This includes runtime components like Dapr, infrastructure-as-code (IaC) tools like Terraform, and monitoring solutions like Dynatrace.
  - Azure Linux ships with `containerd` as its container runtime and uses the upstream Linux kernel. This enables existing containers based on Linux images (such as Alpine) to work seamlessly on Azure Linux.

## What's new with Azure Linux 3.0?

Azure Linux 3.0 is generally available for use on AKS. Every three years, Azure Linux releases a new version of its operating system with upgrades to major components. 

The following table shows the version upgrades for major components in this release: 

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

To get started with NVIDIA GPU workloads on AKS with Azure Linux, see [Use GPUs for compute-intensive workloads on AKS][use-nvidia-gpu].

> [!NOTE]
> If there are specific GPU series you'd like to see supported on AKS with Azure Linux, or if you have other prioritization requests, file an issue in the [AKS GitHub repository](https://github.com/Azure/AKS/issues).

## Next steps

- Learn more about [Azure Linux Container Host core concepts](./concepts-core.md).
- Follow the tutorial to [deploy, manage, and update applications](./tutorial-azure-linux-create-cluster.md).
- Get started by [creating an Azure Linux Container Host for AKS cluster using the Azure CLI](./quickstart-azure-cli.md).

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
