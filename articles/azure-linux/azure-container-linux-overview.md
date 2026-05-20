---
title: Azure Container Linux (ACL) for Azure Kubernetes Service (AKS) Overview
description: Learn about Azure Container Linux (ACL), an immutable, container-optimized operating system for Azure Kubernetes Service (AKS) that builds on Flatcar Container Linux while integrating Azure Linux packages and platform features.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: overview
ms.date: 04/28/2026
---

# What is Azure Container Linux (ACL) for Azure Kubernetes Service (AKS)?

In this article, we provide an overview of Azure Container Linux (ACL), an immutable, container-optimized operating system (OS) for Azure Kubernetes Service (AKS). ACL is derived from the Flatcar Container Linux project, building on Flatcar's proven, container-first immutable design while layering in Azure Linux packages, servicing, and platform integration. This allows ACL to stay closely aligned with upstream Flatcar innovation while meeting Azure's production, security, and compliance requirements. To learn more about Flatcar Container Linux, see the [Flatcar documentation](https://www.flatcar.org/).

ACL is generally available (GA) as an OS option on AKS starting AKS v1.34. You can deploy ACL node pools in a new AKS cluster or add ACL node pools to your existing clusters.

> [!NOTE]
> ACL is the GA release of Flatcar Container Linux for AKS, which entered public preview in November 2025. OS Guard (preview) features, such as code integrity with Integrity Policy Enforcement (IPE), will be incorporated into ACL in a future release, after which OS Guard (preview) will be retired. If you need OS Guard features today, we recommend continuing to use OS Guard and migrating to ACL once those features become available.

## Benefits of using ACL on AKS

| Benefit | Description |
| ------- | ----------- |
| **Built-in immutability for stronger security** | Kernel-enforced immutability of the `/usr` directory verifies the integrity of the OS image at boot and runtime. This design helps block unauthorized changes before they can affect your cluster and reduces the risk of OS-level tampering. |
| **Minimal attack surface** | ACL ships only the components required to run containers. By reducing the size and complexity of the OS, ACL minimizes the number of packages, services, and potential entry points available to attackers and simplifies security management. |
| **Automated node image updates** | ACL delivers weekly image-based updates that include the latest security patches and bug fixes. This approach keeps node OS versions consistent and current across the cluster and helps reduce exposure to known vulnerabilities. |
| **Supply-chain trust** | Builds on Azure Linux’s signed packages and supply-chain processes, delivering clear provenance for system components. |
| **Integration with Azure security features** | Native support for [Trusted Launch](/azure/aks/use-trusted-launch) and Secure Boot provides measured boot protections and attestation. |
| **Open-source transparency** | Flatcar as well as many of the underlying technologies (dm-verity and SELinux) are upstream or open source, and Microsoft has tooling and contributions to support these features. |

## Key features of ACL

The following key features distinguish ACL as a hardened, container-optimized OS for AKS:

- **Immutability**: The '/usr' directory is mounted as a read-only volume protected by dm-verity. At runtime, the kernel validates a signed root hash to detect and block tampering
- **Mandatory access control with SELinux**: ACL includes SELinux to enforce mandatory access control policies that restrict which processes can access sensitive system resources. SELinux is operates in enforcing mode by default.
- **Trusted Launch and Secure Boot**: ACL requires [Trusted Launch](/azure/virtual-machines/trusted-launch) with Secure Boot and vTPM, to ensure the integrity of the boot chain before the OS loads. This is achieved using a Unified Kernel Image (UKI), which bundles the kernel, initramfs, and kernel command line into a single signed artifact. During boot, the UKI is measured and recorded in the vTPM, ensuring integrity from the earliest stage.
- **NVIDIA GPU node support**: ACL supports NVIDIA GPU-enabled node pools on AMD64 architectures, allowing you to run high-performance computing (HPC) and AI/ML workloads on AKS with a hardened, container-optimized OS. ACL doesn't support ARM64 architectures for GPU-enabled node pools.
- **AMD64 and ARM64 architecture support**: ACL is available for both AMD64 and ARM64 architectures on AKS.
- **Sovereign Supply Chain Security**: ACL inherits Azure Linux’s secure build pipelines and signed Unified Kernel Images (UKIs).
- **[Node auto-provisioning](/azure/aks/node-auto-provisioning)**: ACL supports node auto-provisioning (NAP).

## Unsupported features

ACL currently doesn't support the following features:

- The `SecurityPatch` and `Unmanaged` [node OS upgrade channels](/azure/aks/auto-upgrade-node-os-image).
- [Generation 1 VMs](/azure/aks/aks-virtual-machine-sizes): You can't use VM sizes that only support Generation 1 with ACL.
- [Pod Sandboxing](/azure/aks/use-pod-sandboxing).
- A non-Trusted Launch variant. ACL requires Trusted Launch.

If your existing cluster uses any of the unsupported features, you might not be able to add an ACL node pool to that cluster.

## Feature roadmap

Azure Linux publishes a [feature roadmap](https://github.com/orgs/microsoft/projects/970/views/2) that contains features that are in development and available for general availability (GA) and public preview.

## OS migrations and upgrades with ACL

AKS supports migrating existing node pools to ACL using in-place OS SKU migration or by creating new ACL node pools. For detailed migration steps, considerations, and rollback instructions, see [Migrate existing nodes to ACL](./tutorial-migrate-azure-container-linux-aks.md).

## ACL for AKS versioning

ACL for AKS releases weekly AKS node images. Versioning follows the AKS date-based format (for example: 202506.13.0). ACL currently only supports full node image updates.

You can check available node images in the release notes and view the `nodeImageVersion` for a running cluster using the [`az aks nodepool list`](/cli/azure/aks/nodepool#az-aks-nodepool-list) command. For example:

```azurecli-interactive
az aks nodepool list --resource-group <resource-group-name> --cluster-name <aks-cluster-name> --query '[].{name: name, nodeImageVersion: nodeImageVersion}'
```

Example output:

```output
[
{
    "name": "nodes",
    "nodeImageVersion": "AKSAzureContainerLinux-202606.01.0"
}
]
```

## Related content

To get started using ACL for AKS, see the following resources:

- [Deploy an ACL cluster using the Azure CLI](./quick-deploy-azure-container-linux-aks-cli.md)
- [Deploy an ACL cluster using an ARM template](quick-deploy-azure-container-linux-aks-arm.md)
- [Tutorial: Create an ACL cluster](./tutorial-create-cluster-azure-container-linux-aks.md)
- [Tutorial: Add an ACL node pool to an existing cluster](./tutorial-add-azure-container-linux-node-pool.md)
