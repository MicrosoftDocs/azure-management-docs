---
title: Prepare Linux for Edge Volumes
description: Learn how to prepare Linux in Azure Container Storage enabled by Azure Arc Edge Volumes using Azure Kubernetes Service enabled by Azure Arc, Edge Essentials, or Ubuntu.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.custom:
  - linux-related-content
  - references_regions
  - build-2025
ms.date: 02/05/2026

# Customer intent: As a cloud developer, I want to prepare my Linux environment for Edge Volumes in Azure Container Storage, so that I can effectively manage and deploy workloads using Azure Kubernetes Service (AKS) enabled by Azure Arc.
---

# Prepare Linux for Edge Volumes

The article describes how to prepare Linux for Edge Volumes using Azure Kubernetes Service (AKS) enabled by Azure Arc, Edge Essentials, or Ubuntu.

These instructions assume that you already have an Arc-enabled Kubernetes cluster. To connect an existing Kubernetes cluster to Azure Arc, [see these instructions](/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli).

If you want to use Azure Container Storage enabled by Azure Arc with Azure IoT Operations, follow the [instructions to create a cluster for Azure IoT Operations](/azure/iot-operations/deploy-iot-ops/howto-prepare-cluster?tabs=ubuntu).

You must also install a cert-manager as described in [Install Edge Volumes](howto-install-edge-volumes.md#install-certificate-and-trust-managers).

## Prerequisites

The following table lists the prerequisites for Azure Container Storage enabled by Azure Arc:

| Requirement | Single-node/Two-node cluster | Multi-node cluster |
|-------------|------------------------------|-------------------|
| **Operating System** | | |
| Kernel version | 5.15 and above (minimum supported) | 5.15 and above (minimum supported) |
| NFSv4.2 support | Enabled in the kernel | Enabled in the kernel |
| **Hardware Requirements** | | |
| CPU | 4 CPUs minimum | 8 CPUs minimum |
| RAM | 16 GB minimum | 32 GB recommended (16 GB minimum) |
| VM recommendation | Standard_D8ds_v5 or equivalent | Standard_D8as_v5 or equivalent |
| **Storage Requirements** | | |
| Storage provisioner | Local-path storage class required | N/A |
| Reserved system volume | 1 GB per Edge Volume | N/A |
| **System Configuration** | | |
| sysctl configuration | `fs.inotify.max_user_instances >= 1024` | `fs.inotify.max_user_instances >= 1024` |
| NVME over TCP kernel module | Not required | Required |
| Hugepages configuration | Not required | Set to 512 |

## Additional considerations

- **Kernel compatibility**: Known issues exist with kernel versions 6.4 and 6.2.
- **Regional availability**: Azure Container Storage enabled by Azure Arc is only available in: East US, East US 2, West US, West US 2, West US 3, North Europe, West Europe
- Ensure that the required disk storage is available and properly mounted
- For multi-node clusters, 32 GB RAM serves as a buffer; however, 16 GB RAM should suffice. Edge Essentials configurations require 8 CPUs with 10 GB RAM per node, making 16 GB RAM the minimum requirement.

## Single-node clusters

A single-node cluster is commonly used for development or testing purposes due to its simplicity in setup and minimal resource requirements. These clusters offer a lightweight and straightforward environment for developers to experiment with Kubernetes without the complexity of a multi-node setup. Additionally, in situations where resources such as CPU, memory, and storage are limited, a single-node cluster is more practical. Its ease of setup and minimal resource requirements make it a suitable choice in resource-constrained environments.

However, single-node clusters come with limitations, including their lack of high availability, fault tolerance, scalability, and decreased performance.

## Multi-node clusters

A multi-node Kubernetes configuration is typically used for production, staging, or large-scale scenarios because of features such as high availability, fault tolerance, scalability, and performance. A multi-node cluster also introduces challenges and trade-offs, including complexity, overhead, cost, and efficiency considerations. For example, setting up and maintaining a multi-node cluster requires extra knowledge, skills, tools, and resources (network, storage, compute). The cluster must handle coordination and communication among nodes, leading to potential latency and errors. Additionally, running a multi-node cluster is more resource-intensive and is costlier than a single-node cluster. Optimization of resource usage among nodes is crucial for maintaining cluster and application efficiency and performance.

In summary, a single-node Kubernetes cluster might be suitable for development, testing, and resource-constrained environments. A multi-node cluster is more appropriate for production deployments, high availability, scalability, and scenarios in which distributed applications are a requirement. This choice ultimately depends on your specific needs and goals for your deployment.

Azure Container Storage enabled by Azure Arc doesn't natively support multi-node deployments. If you need a multi-node Kubernetes cluster, consider using the [Azure Local](/azure/azure-local) service.

### Minimum storage requirements

When you provision an Edge Volume, a static system volume of 1 GB is reserved for the system itself to run; this is in addition to the provisioned size chosen by the user in the **Custom Resource Definition** option.

When you use Azure Local for fault tolerance, Azure Local offers a *ReadWriteOnce* file system, so you can allocate resources to it in the same way you would for a single node cluster; no other constraints apply.

## Prepare Linux for Edge Volumes using a single-node or two-node cluster

This section describes how to prepare Linux using a single-node or two-node cluster, and assumes you [fulfilled the prerequisites](#prerequisites). Select the appropriate tab for your Linux distribution and Kubernetes platform.

# [AKS enabled by Azure Arc](#tab/aks)

### Prepare Linux with AKS enabled by Azure Arc

If you run a single-node or two-node cluster on Linux with AKS enabled by Azure Arc, you don't need to perform any additional steps.

# [AKS Edge Essentials](#tab/aks-ee)
[!INCLUDE [single-node-edge-essentials](includes/single-node-edge-essentials.md)]

# [Ubuntu](#tab/ubuntu)
[!INCLUDE [single-node-ubuntu](includes/single-node-ubuntu.md)]

# [Other](#tab/other)

### Prepare Linux with other platforms

The available platform options are production-like environments that Microsoft validated. These platforms aren't necessarily the only environments on which Azure Container Storage enabled by Azure Arc can run. Azure Container Storage can run on any Arc-enabled Kubernetes cluster that meets the Azure Arc-enabled Kubernetes system requirements. If you're running on an environment not listed, here are a few suggestions to increase the likelihood of a successful installation:

1. Run the following commands to increase the user watch and instance limits:

   ```bash
   echo fs.inotify.max_user_instances=8192 | sudo tee -a /etc/sysctl.conf
   echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. Run the following commands to increase the file descriptor limit for better performance:

   ```bash
   echo fs.file-max = 100000 | sudo tee -a /etc/sysctl.conf
   sudo sysctl -p
   ```

1. Run the following command to install the local path provisioner:

   ```bash
   kubectl apply -f https://raw.githubusercontent.com/Azure/AKS-Edge/main/samples/storage/local-path-provisioner/local-path-storage.yaml
   ```

---

## Next steps

- [Prepare Linux for Edge Volumes using a single-node or two-node cluster](howto-single-node-cluster-edge-volumes.md)
- [Prepare Linux for Edge Volumes using a multi-node cluster](howto-multi-node-cluster-edge-volumes.md)
