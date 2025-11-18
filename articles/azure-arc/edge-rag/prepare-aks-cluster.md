---
title: Prepare AKS Cluster on Azure Local for Edge RAG Preview Enabled by Azure Arc
description: "Learn how to prepare an AKS cluster on Azure Local for Edge RAG deployment, including node pool setup and GPU driver installation."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 11/17/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to prepare an AKS cluster on Azure Local for Edge RAG deployment so that my environment meets the requirements for running the Edge RAG workload.
---

# Prepare AKS cluster on Azure Local for Edge RAG Preview enabled by Azure Arc

For your Edge RAG deployment, prepare an AKS cluster on Azure Local by creating the cluster, configuring node pools, and installing GPU drivers as needed. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prepare your AKS cluster

Create an Azure Kubernetes Service (AKS) cluster on the Azure Local instance with a node pool that meets the minimum requirements.

## Create an AKS Arc cluster

Create an AKS Arc cluster by using one of the following methods:

- Azure portal, see [How to deploy a Kubernetes cluster using the Azure portal](/azure/aks/hybrid/aks-create-clusters-portal)

- Azure CLI, see [Create Kubernetes clusters using Azure CLI](/azure/aks/hybrid/aks-create-clusters-cli)

## Install supported GPU drivers (optional)

AKS Arc only supports Nvidia A2 and A16 GPUs. The following steps are applicable only to these two GPUs.

If you have GPUs available in your Azure Local instance that you want to use for Edge RAG, make sure that the necessary GPU drivers are installed and available in the AKS Arc cluster nodes.

To check if the right drivers are already installed and the GPUs are available to the AKS Arc cluster, run the following command:

```powershell
(Get-MocNode -location MocLocation).properties.statuses.Info`
```

If the output lists all the GPUs available on the Azure Local cluster, you can move to the next step. Otherwise, complete the following steps on any of the Azure Local cluster nodes to enable GPUs.

We recommend using the script in the [Enabling GPU on AKS on Azure ARC - sample](enable-gpu-aks.md) to enable GPUs for use by Edge RAG.

Alternately, you can follow the instructions in [Use GPUs for compute-intensive workloads](/azure/aks/hybrid/deploy-gpu-node-pool) and ensure you meet the [minimum VM hardware requirements](requirements.md#minimum-vm-hardware-requirements) for the GPU mode. If you follow these instructions, you need to run the following command on each Hyper-V host in the Azure Local cluster:

```powershell
Restart-Service wssdagent -Force -Verbose 
Start-sleep 60
(Get-MocNode -location MocLocation).properties.statuses.Info
```

Make sure that all the available GPUs across all nodes are listed in the output of the command.

## Configure machine to manage Azure Arc-enabled Kubernetes clusters (optional)

If you want to manage the Kubernetes clusters from a machine outside the Azure Local instance, set up a driver machine (local management host) with the following tools:

- Azure CLI
- Azure CLI extensions aksarc and Kubernetes-extension
- kubectl
- Helm

This driver machine must be able to connect to the Kubernetes cluster on the network.

To set up a Windows machine to manage your Kubernetes clusters, see the [Script to configure machine to manage Azure Arc-enabled Kubernetes cluster](configure-driver-machine.md).

## Create node pools for AKS Arc cluster

To create a node pool for AKS Arc, complete the following steps from the driver machine. 

1. Sign into Azure by using Azure CLI: `az login`.
1. Create the node pool.

   If GPUs are available:

   - You must create a node pool of at least three CPU virtual machines (VMs), with minimal size of "Standard_D8s_v3". Run the following command:
   
    	```powershell
    	$cpuPoolName = "<CPU Pool Name>"
    	$gpuPoolName = "<GPU Pool Name>"
    	$gpuVmSku = "Standard_NC8_A2" #Can also use Standard_NC8_A16
    	$cpuVmSku = "Standard_D8s_v3"
    	$rg = "<Resource Group name>"
    	$cpuNodeCount = 3
    	$gpuNodeCount = 3
    			
    	az aksarc nodepool add --name $cpuPoolName --cluster-name $k8scluster -g $rg --node-count $cpuNodeCount --node-vm-size $cpuVmSku
    	```
   
   - You must create a node pool of at least three GPU VMs, with minimal size of "Standard_NC8_A2" or "Standard_NC8_A16". Run the following command:
   
    	```powershell
    	az aksarc nodepool add --name $gpuPoolName --cluster-name $k8scluster -g $rg --node-count $gpuNodeCount  --node-vm-size $gpuVmSku
    	```
   
   If only CPUs are available, you must create a node pool of at least six CPU VMs, with minimal size of "Standard_D8s_v3". Run the following command:
   
    ```powershell
    $cpuPoolName = "<CPU Pool Name>"
    $cpuVmSku = "Standard_D8s_v3"
    $rg = "<Resource Group name>"
    $cpuNodeCount = 6
    $k8scluster = "<AKS Arc Cluster>"
    az aksarc nodepool add --name $cpuPoolName --cluster-name $k8scluster -g $rg --node-count $cpuNodeCount --node-vm-size $cpuVmSku
    ```

For more information, see [Create node pools for a cluster in Azure Kubernetes Service (AKS)](/azure/aks/create-node-pools).

## Next step

> [!div class="nextstepaction"]
> [Configure authentication](prepare-authentication.md)
