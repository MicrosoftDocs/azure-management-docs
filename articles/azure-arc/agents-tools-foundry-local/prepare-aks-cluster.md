---
title: Prepare AKS Cluster on Azure Local for Agents and Tools with Foundry Local
description: "Learn how to prepare an AKS cluster on Azure Local for Agents and Tools with Foundry Local deployment, including node pool setup and GPU driver installation."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to prepare an AKS cluster on Azure Local for Agents and Tools with Foundry Local deployment so that my environment meets the requirements for running the Agents and Tools with Foundry Local workload.
---

# Prepare AKS cluster on Azure Local for Agents and Tools with Foundry Local

For your Agents and Tools with Foundry Local deployment, prepare an AKS cluster on Azure Local by creating the cluster, configuring node pools, and installing GPU drivers as needed. This article is part of the deployment prerequisites checklist.

## Prepare your AKS cluster

Create an Azure Kubernetes Service (AKS) cluster on the Azure Local instance with a node pool that meets the minimum requirements.

## Create an AKS Arc cluster

Create an AKS Arc cluster by using one of the following methods:

- Azure portal, see [How to deploy a Kubernetes cluster using the Azure portal](/azure/aks/hybrid/aks-create-clusters-portal)

- Azure CLI, see [Create Kubernetes clusters using Azure CLI](/azure/aks/hybrid/aks-create-clusters-cli)

## Install supported GPU drivers (optional)

AKS on Azure Arc supports only NVIDIA A2 and A16 GPUs. The following steps apply only to these two GPUs.

If you have GPUs available in your Azure Local instance that you want to use for the extension, ensure that the necessary GPU drivers are installed and available in the AKS on Azure Arc cluster nodes.

To check if the right drivers are already installed and the GPUs are available to the AKS on Azure Arc cluster, run the following command:

```powershell
(Get-MocNode -location MocLocation).properties.statuses.Info
```

If the output lists all the GPUs available on the Azure Local cluster, you can move to the next step. Otherwise, complete the following steps on any of the Azure Local cluster nodes to enable GPUs.

Use the script in [Enabling GPU on AKS on Azure Arc - sample](enable-gpu-aks.md) to enable GPUs for use by the extension.

Alternatively, follow the instructions in [Use GPUs for compute-intensive workloads](/azure/aks/hybrid/deploy-gpu-node-pool) and ensure you meet the [minimum VM hardware requirements](requirements.md#minimum-vm-hardware-requirements) for the GPU mode. If you follow these instructions, run the following command on each Hyper-V host in the Azure Local cluster:

```powershell
Restart-Service wssdagent -Force -Verbose 
Start-sleep 60
(Get-MocNode -location MocLocation).properties.statuses.Info
```

Ensure that the output of the command lists all the available GPUs across all nodes.

## Configure machine to manage Azure Arc-enabled Kubernetes clusters (optional)

To manage Kubernetes clusters from a machine outside the Azure Local instance, set up a driver machine (local management host) with the following tools:

- Azure CLI
- Azure CLI extensions aksarc and k8s-extension
- kubectl
- Helm

This driver machine must connect to the Kubernetes cluster on the network.

To set up a Windows machine to manage your Kubernetes clusters, see [Script to configure machine to manage Azure Arc-enabled Kubernetes cluster](configure-driver-machine.md).

## Create node pools for AKS Arc cluster

To create a node pool for AKS Arc, complete the following steps from the driver machine. 

1. Sign in to Azure by using Azure CLI: `az login`.
1. Create the node pool.

   If GPUs are available and `indexedSourcesEnabled=true`:

   - Create a node pool of at least three CPU virtual machines (VMs), with a minimum size of `Standard_D8s_v3`. Run the following command:
   
```powershell
$cpuPoolName = "<CPU Pool Name>"
$gpuPoolName = "<GPU Pool Name>"
$gpuVmSku = "Standard_NC8_A2" #Can also use Standard_NC8_A16
$cpuVmSku = "Standard_D8s_v3"
$rg = "<Resource Group name>"
$k8scluster = "<AKS Arc Cluster>"
$cpuNodeCount = 3
$gpuNodeCount = 2

az aksarc nodepool add --name $cpuPoolName --cluster-name $k8scluster -g $rg --node-count $cpuNodeCount --node-vm-size $cpuVmSku
```
   
   - Create a node pool of at least two GPU virtual machines (VMs), with a minimum size of `Standard_NC8_A2` or `Standard_NC8_A16`. Run the following command:

When `indexedSourcesEnabled=true`, the two GPU VMs handle the local indexed-source stack's text embedding model (BGE-M3) and image embedding model (CLIP ViT-L/14). Docling (document parser) runs on CPU. External LLM endpoints don't use cluster GPUs, whereas running Foundry Local (`gpt-oss-20b`) on the same cluster requires an additional GPU VM.
   
```powershell
az aksarc nodepool add --name $gpuPoolName --cluster-name $k8scluster -g $rg --node-count $gpuNodeCount --node-vm-size $gpuVmSku
```
   
   If `indexedSourcesEnabled=true` and you run embeddings on CPU instead of GPU, create a node pool of at least six CPU VMs, with a minimum size of `Standard_D8s_v3`. Run the following command:
   
```powershell
$cpuPoolName = "<CPU Pool Name>"
$cpuVmSku = "Standard_D8s_v3"
$rg = "<Resource Group name>"
$cpuNodeCount = 6
$k8scluster = "<AKS Arc Cluster>"
az aksarc nodepool add --name $cpuPoolName --cluster-name $k8scluster -g $rg --node-count $cpuNodeCount --node-vm-size $cpuVmSku
```

CPU-only embedding mode applies only when `indexedSourcesEnabled=true`. In CPU-only mode, embedding quality might be reduced and image search isn't available. Docling (document parser) runs on CPU by default and doesn't require GPU.

### Node pool requirements by indexed sources setting

| `indexedSourcesEnabled` | Embedding GPU node pool | CPU node pool | Foundry Local on the same cluster |
|---|---|---|---|
| `true` (default) | 2 GPU VMs for GPU embeddings (or 0 GPUs if using 6 CPU VMs for CPU embeddings) | 3+ CPU VMs with GPU embeddings, or 6+ CPU VMs with CPU embeddings | Requires 1 additional GPU VM for `gpt-oss-20b` |
| `false` | 0 embedding GPUs required (local embedding and parsing aren't deployed) | 3+ CPU VMs required | Requires 1 GPU VM if `gpt-oss-20b` runs on the cluster |

If you set `indexedSourcesEnabled=false`, you don't need the 2 embedding GPU VMs (or the 6-VM CPU embedding pool); 3+ CPU VMs are sufficient unless you also host Foundry Local on the cluster.

For more information, see [Create node pools for a cluster in Azure Kubernetes Service (AKS)](/azure/aks/create-node-pools).

## Next step

> [!div class="nextstepaction"]
> [Configure authentication](prepare-authentication.md)
