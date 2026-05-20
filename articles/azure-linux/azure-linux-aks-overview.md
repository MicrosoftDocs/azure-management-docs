---
title: Overview of the Azure Linux Container Host for Azure Kubernetes Service (AKS)
description: Learn about the Azure Linux Container Host, a lightweight and secure operating system image optimized for running container workloads on Azure Kubernetes Service (AKS), including its key benefits and supported GPU VM sizes.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: overview
ms.date: 04/28/2026
---

# What is the Azure Linux Container Host for Azure Kubernetes Service (AKS)?

The Azure Linux Container Host is an operating system (OS) image optimized for running container workloads on [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes).

The Azure Linux Container Host is lightweight, containing only the packages needed to run container workloads. The container host is hardened based on significant validation tests and internal usage and is compatible with Azure agents. It provides reliability and consistency from cloud to edge across AKS, Azure Local, and Azure Arc. You can deploy Azure Linux node pools in a new cluster, add Azure Linux node pools to your existing clusters, or migrate your existing nodes to Azure Linux nodes.

This article provides an overview of the Azure Linux Container Host, its key benefits, and the GPU VM sizes it supports when running container workloads on AKS.

## Key benefits

The Azure Linux Container Host offers the following key benefits:

| Benefit | Description |
| ------- | ----------- |
| Small and lightweight | • Includes only the necessary set of packages needed to run container workloads, consuming limited disk and memory resources and producing faster cluster operations on AKS. <br> • Has roughly 400 packages, taking up the least disk space by up to _five GB_ on AKS. |
| Secure supply chain | • The Linux and AKS teams at Microsoft build, sign, and validate the [Azure Linux Container Host packages][azure-linux-packages] from source, and host packages and sources in Microsoft-owned and secured platforms. <br> • Each package runs through a full set of unit tests and end-to-end testing on the existing image to prevent regressions before release. <br> • Azure Linux focuses on stability, backporting fixes in core components like the kernel or openssl and limiting substantial changes or version bumps to major release boundaries to prevent customer outages. |
| Secure by default | • Follows secure-by-default principles, including using a hardened Linux kernel with Azure cloud optimizations and flags tuned for Azure. <br> • Provides a reduced attack surface and eliminates patching and maintenance of unnecessary packages. <br> • Microsoft monitors the Common Vulnerabilities and Exposures (CVE) database and releases security patches monthly and critical updates within days if necessary. <br> • Azure Linux passes all the [CIS Level 1 benchmarks][cis-benchmarks], making it the only Linux distribution on AKS that does so. |
| Maintains compatibility with existing workloads | • All existing and future AKS extensions, add-ons, and open-source projects on AKS support Azure Linux. Extension support includes support for runtime components like Dapr, IaC tools like Terraform, and monitoring solutions like Dynatrace. <br> • Azure Linux ships with `containerd` as its container runtime and the upstream Linux kernel, which enables existing containers based on Linux images (like Alpine) to work seamlessly on Azure Linux. |

## Supported GPU virtual machine (VM) sizes

Azure Linux supports all the NVIDIA GPU VM sizes used on AKS. To get started deploying NVIDIA GPU workloads on AKS with Azure Linux, see [Use GPUs for compute-intensive workloads on Azure Kubernetes Service (AKS)][use-nvidia-gpu].

## Related content

- [Create a cluster with the Azure Linux Container Host for AKS](./tutorial-create-cluster-azure-linux-aks.md)
- [Deploy an Azure Linux Container Host for AKS cluster using the Azure CLI](./deploy-azure-linux-aks-cli.md)

<!-- LINKS - internal -->
[use-nvidia-gpu]: /azure/aks/use-nvidia-gpu
[cis-benchmarks]: /azure/aks/cis-azure-linux-v3

<!-- LINKS - external -->
[azure-linux-packages]: https://packages.microsoft.com/azurelinux/3.0/prod/
