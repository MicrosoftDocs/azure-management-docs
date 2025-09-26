---
title: "Quickstart: Install Edge RAG on Azure Kubernetes Service"
description: "Walkthrough for installing Edge RAG on AKS."
author: cwatson-cat
ms.author: cwatson
ms.service: azure-arc
ms.topic: quickstart
ms.date: 09/26/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#customer intent: As a user, I want to install Edge RAG on Azure Kubernetes Service so that I can assess the solution.

---

# Quickstart: Install Edge RAG Preview enabled by Azure Arc

In this quickstart, you deploy Edge RAG on Azure Kubernetes Service (AKS), and verify the installation. This quickstart is intended to get you started with Edge RAG for evaluation or development purposes without the need for local hardware. To deploy Edge RAG for a production environment, see [Deployment overview](deploy-overview.md).

If you don't have a service subscription, create a [free Azure account](https://azure.microsoft.com/free/).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you have:

- An active Azure subscription
- Permissions to create and manage Azure Kubernetes Service (AKS) clusters and install extensions.
- Azure CLI installed ([Install Azure CLI](/cli/azure/install-azure-cli)) locally unless you plan to use Azure Cloud Shell

[!INCLUDE [cloud-shell-try-it.md](~/reusable-content/ce-skilling/azure/includes/cloud-shell-try-it.md)]

## Open Azure Cloud Shell or Azure CLI

You can use the Azure Cloud Shell or your local Azure CLI to run the following commands. Make sure you are logged in:

```azurepowershell-interactive
az login
```

## Create resource group

Create a resource group to contain the AKS cluster, node pool, and Edge RAG resources.

```azurepowershell-interactive
$rg = "edge-rag-aks-rg" `
$location = "eastus2" `
az group create --name $rg --location $location
```

## Create and configure an AKS cluster

In this section, you create an AKS cluster and configure it for Edge RAG deployment. The steps include setting up the cluster, connecting it to Azure Arc, and preparing it with the necessary extensions and GPU support.

1. Create an AKS cluster:

   ```azurepowershell-interactive
   $k8scluster =  "edge-rag-aks"  
   az aks create --resource-group $rg --name $k8scluster --node-count 3 --generate-ssh-keys
   ```

1. Set the rest of the following values as needed and then run the command:

   ```azurepowershell-interactive
   # Azure variables
   $sub = "your_subscription"   
   $tenantid = "your_tenantID" 
    
   # Edge RAG extension variables   
   $modelName = "microsoft/Phi-3.5"    
   $gpu_enabled = "true" # set to false if no GPU nodes 
   $localextname = "edgeragdemo"  
   $autoUpgrade = "false" 
   $domainName = "arcrag.contoso.com"  # tied to your EntraID app registration    
   $entraTenantId = "your_entra_tenant_id" #  tied to your EntraID app registration    
   $entraClientId = "your_entra_app_id"  # tied to your EntraID app registration 
   $extension = "microsoft.arc.rag" # do not change    
   $n = "arc-rag" # do not change
   ```

1. Connect to Azure and AKS. 

   ```azurepowershell-interactive
   az login --scope https://management.core.windows.net//.default --tenant $tenantId `   
   az aks get-credentials --resource-group $rg --name $cluster --overwrite-existing 
   ``` 

1. Install the NVIDIA GPU operator on the cluster.

   ```azurepowershell-interactive
   helm repo add nvidia https://helm.ngc.nvidia.com/nvidia 
   helm repo update   
   helm install --wait --generate-name -n gpu-operator --create-namespace nvidia/gpu-operator --version=v24.9.2 
   ```
 
1. Connect the AKS cluster to Azure Arc.

   ```azurepowershell-interactive
   az connectedk8s connect --resource-group $rg --location $location --name $k8scluster  
   ```

1. Install the Kubernetes-native certificate management controller by running the following command.
 
   ```azurepowershell-interactive
   az k8s-extension create `   
      -g $rg `    
      -c $k8scluster `   
      -t connectedClusters `    
      --scope cluster `
      --name cert-manager `    
      --release-namespace cert-manager    
      --release-train preview `   
      --extension-type Microsoft.iotoperations.platform `    
      --debug 
   ```

## Create node pools

Add dedicated GPU and CPU node pools to your AKS cluster to support Edge RAG. Run the following command:

```azurepowershell-interactive
# GPU nodepool 
az aks nodepool add ` 
    --resource-group $rg ` 
    --cluster-name $k8scluster ` 
    --name "gpunodepool" ` 
    --node-count 3 ` 
    --node-vm-size "Standard_NC8as_T4_v3" `
    --enable-cluster-autoscaler `
    --min-count 3 ` 
    --max-count 3 ` 
    --mode User 

# CPU nodepool 

az aks nodepool add ` 
    --resource-group $rg ` 
    --cluster-name $k8scluster ` 
    --name "cpunodepool" ` 
    --node-count 3 ` 
    --node-vm-size "Standard_D4s_v3" ` 
    --enable-cluster-autoscaler ` 
    --min-count 3 ` 
    --max-count 3 ` 
    --mode User
 ```

If the nodepool creation fails with a **DenyVMsWithoutTrustedLaunchEnabled** policy error, add the following tags to the command and rerun:  

```azurepowershell-interactive
SkipDenyVMsWithoutTrustedLaunchEnabled : true 
azsecpack : nonprod 
```

## Deploy Edge RAG on AKS

Complete the following steps to deploy the Edge RAG extension onto your AKS cluster.

1. Deploy the Edge RAG extension by running the following command.

   ```azurepowershell-interactive
   az k8s-extension create `    
       --cluster-type connectedClusters `   
       --cluster-name $k8scluster `    
       --resource-group $rg `    
       --name $localextname `   
       --extension-type $extension `    
       --debug --release-train preview ` 
       --auto-upgrade $autoUpgrade ` 
       --configuration-settings isManagedIdentityRequired=true ` 
       --configuration-settings gpu_enabled=$gpu_enabled ` 
       --configuration-settings AgentOperationTimeoutInMinutes=30 ` 
       --configuration-settings model=$modelName ` 
       --configuration-settings auth.tenantId=$entraTenantId ` 
       --configuration-settings auth.clientId=$entraClientId ` 
       --configuration-settings ingress.domainname=$domainName ` 
       --configuration-settings ingress-nginx.controller.service.annotations.service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path=/healthz ` 
   ```

1. Get the load balancer VIP by running the following command:

   ```azurepowershell-interactive
   kubectl get service ingress-nginx-controller -n arc-rag -o yaml 
   ```

   Look for:
   ```markdown
   status:    
   loadBalancer:   
      ingress:    
      - ip: <load_balancer_ip>    
      ipMode: VIP 
   ```
## Connect to the developer portal

Update your host file on your local machine to connect to the developer portal for Edge RAG.

1. On your local machine, open Notepad in Administrator mode. 
1. Go to **File** > **Open** > **C:\windows\System32\drivers\etc** > **hosts**. If you can't see the hosts file, set extension type to **All files**.

   Add this line at the end of the file where you replace `load_balancer_ip` with the load balancer IP and save. 

   `<load_balancer_ip> arcrag.contoso.com` 

1. Go to the developer portal for Edge RAG at `https://arcrag.contoso.com`.

## Clean up resources

To remove the resources created in this quickstart, run:

```azurepowershell-interactive
az group delete --name $rg--yes --no-wait
```

## Next step

> [!div class="nextstepaction"]
> [Add Data Source for Edge RAG](add-data-source.md)

---
