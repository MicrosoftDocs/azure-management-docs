---
title: "Quickstart: Install Edge RAG on Azure Kubernetes Service"
description: "Learn how to install Edge RAG on Azure Kubernetes Service (AKS) without the need for local hardware."
author: cwatson-cat
ms.author: cwatson
ms.service: azure-arc
ms.topic: quickstart
ms.date: 10/28/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#customer intent: As a user, I want to install Edge RAG on Azure Kubernetes Service so that I can assess the solution.
---

# Quickstart: Install Edge RAG Preview enabled by Azure Arc

In this quickstart, you deploy Edge RAG on Azure Kubernetes Service (AKS) without the need for local hardware like Azure Local. This quickstart is intended to get you started with Edge RAG for evaluation or development purposes. To deploy Edge RAG for a production environment, see [Deployment overview](deploy-overview.md).


[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you have:

- An active Azure subscription. If you don't have a service subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Azure CLI, Helm, kubectl, and the extensions aksarc and Kubernetes-extension installed locally unless you plan to use [Azure Cloud Shell](/azure/cloud-shell/get-started/ephemeral?tabs=azurecli). If you're not using Azure Cloud Shell, see [Script to configure machine to manage Azure Arc-enabled Kubernetes cluster](configure-driver-machine.md).
- Edge RAG registered as an application, and app roles and an assigned user created in Microsoft Entra ID. See [Configure authentication for Edge RAG](prepare-authentication.md).
- Application (client) ID and the directory (tenant) ID. To get these values after registering Edge RAG, see [Get app and tenant IDs](prepare-authentication.md#optional-get-app-and-tenant-ids).

## Open Azure Cloud Shell or Azure CLI

Open Azure Cloud Shell or your local Azure CLI to run the commands in this article. In Azure Cloud Shell, you might need to select **Switch to PowerShell**.

1. Sign in to Azure to get started:

   ```azurecli-interactive
   az login
   ```

1. If you have multiple subscriptions, run the following command to get a list of your subscriptions and then set the context of your session to the appropriate subscription name:

   ```azurecli
   az account list --output table
   ```

   Replace the placeholder "subscription name" with your subscription and run the following command:

   ```azurecli
   $sub = "<subscription name>" 
   az account set --subscription  $sub
   ```

## Create resource group

Create a resource group to contain the AKS cluster, node pool, and Edge RAG resources.

```azurecli
$rg = "edge-rag-aks-rg" 
$location = "eastus2"
az group create `
     --name $rg `
     --location $location
```

## Create and configure an AKS cluster

In this section, you create an AKS cluster and configure it for Edge RAG deployment. The steps include setting up the cluster, connecting it to Azure Arc, and preparing it with the necessary extensions and GPU support.

1. Create an AKS cluster:

   ```azurecli
   $k8scluster = "edge-rag-aks"  
   az aks create `
      --resource-group $rg `
      --name $k8scluster `
      --node-count 2 `
      --generate-ssh-keys
   ```

1. Set the rest of the following values as needed and then run the command. If you created the application registration for Edge RAG in a different tenant from the AKS cluster, set the values for `$entraAppId` and `$entraTenantId` by using the **Application (client) ID** and **Directory (tenant) ID** on the **EdgeRAG** app registration page in the Azure portal.

   ```azurecli
    
   # Set Edge RAG extension values
   $modelName = "microsoft/Phi-3.5"    
   $gpu_enabled = "true" # set to false if no GPU nodes 
   $localextname = "edgeragdemo"  
   $autoUpgrade = "false" 
   $extension = "microsoft.arc.rag" # do not change    
   $n = "arc-rag" # do not change

   # Set Entra ID app registration values
   $domainName = "arcrag.contoso.com" # Edit to match the domain used in your registration  
   $entraAppId = $(az ad app list --display-name "EdgeRAG" --query "[].appId" --output tsv)  # Display name is the application name in your registration   
   $entraTenantId = $(az account show --query tenantId --output tsv) # Directory or tenant ID     

   ```

   If you get a warning when setting `$entraAppId` or `$entraTenantId` by using the queries, set the values by using the **Application (client) ID** and **Directory (tenant) ID** on the **EdgeRAG** app registration page in the Azure portal.

1. Connect to Azure and AKS:

   ```azurecli
   az login `
      --scope https://management.core.windows.net//.default `
      --tenant $entraTenantId    
   az aks get-credentials `
      --resource-group $rg `
      --name $k8scluster `
      --overwrite-existing 
   ``` 

   Follow the prompts in the command line to sign in and select the subscription.

1. Install the NVIDIA GPU operator on the cluster:

   ```azurecli
   helm repo add nvidia https://helm.ngc.nvidia.com/nvidia 
   helm repo update   
   helm install --wait --generate-name -n gpu-operator --create-namespace nvidia/gpu-operator --version=v24.9.2 
   ```

1. Register the `Microsoft.Kubernetes` provider by running the following command:

   ```azurecli
   az provider register -n Microsoft.Kubernetes
   ```

1. Connect the AKS cluster to Azure Arc:

   ```azurecli
   az connectedk8s connect `
      --resource-group $rg ` 
      --location $location ` 
      --name $k8scluster  
   ```

   If prompted, select **y** to install the extension "connectedk8s".

1. Install the required certificate and trust manager:

   ```azurecli

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.3/cert-manager.yaml --wait  
    helm repo add jetstack https://charts.jetstack.io --force-update  
    start-sleep -Seconds 20 
    helm upgrade trust-manager jetstack/trust-manager --install --namespace cert-manager --wait 
   ```

## Create node pools

Add dedicated GPU and CPU node pools to your AKS cluster to support Edge RAG.

If you get an error message when you try to create the node pools, you might need to request a quota increase for your Azure subscription, try a different virtual machine size, or create the Azure Kubernetes cluster and node pools in a different [Azure region](/azure/reliability/regions-list). For more information, see [Limits for resources, SKUs, and regions in Azure Kubernetes Service (AKS)](/azure/aks/quotas-skus-regions).

1. Run the following command to create a GPU node pool with nodes:

   ```azurecli
   az aks nodepool add ` 
       --resource-group $rg ` 
       --cluster-name $k8scluster ` 
       --name "gpunodepool" ` 
       --node-count 4 ` 
       --node-vm-size "Standard_NC24ads_A100_v4" `
       --enable-cluster-autoscaler `
       --min-count 4 ` 
       --max-count 4 ` 
       --mode User 
   ```

1. Run the following command to create a CPU node pool with nodes:

   ```azurecli
   az aks nodepool add ` 
       --resource-group $rg ` 
       --cluster-name $k8scluster ` 
       --name "cpunodepool" ` 
       --node-count 4 ` 
       --node-vm-size "Standard_D8s_v3" ` 
       --enable-cluster-autoscaler ` 
       --min-count 4 ` 
       --max-count 4 ` 
       --mode User
    ```

## Deploy Edge RAG on AKS

Complete the following steps to deploy the Edge RAG extension onto your AKS cluster.

1. Deploy the Edge RAG extension by running the following command:

   ```azurecli
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
       --configuration-settings AgentOperationTimeoutInMinutes=60 ` 
       --configuration-settings model=$modelName ` 
       --configuration-settings auth.tenantId=$entraTenantId ` 
       --configuration-settings auth.clientId=$entraAppId ` 
       --configuration-settings ingress.domainname=$domainName ` 
       --configuration-settings ingress-nginx.controller.service.annotations.service\.beta\.kubernetes\.io/azure-load-balancer-health-probe-request-path=/healthz 
   ```

   Wait several minutes for the deployment to complete.

1. Get the load balancer VIP by running the following command:

   ```azurecli
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
1. Go to **File** > **Open** > **C:\windows\System32\drivers\etc** > **hosts**. If you can't see the "hosts" file, set the extension type to **All files**.

1. Add the following line at the end of the file where you replace `load_balancer_ip` with the load balancer IP, and edit the domain to match the app registration:

   `<load_balancer_ip> arcrag.contoso.com` 

   For example:

   ```markdown
   # Edge RAG developer portal
   172.16.0.0 arcrag.contoso.com
   ```

1. Save the file.
1. Go to the developer portal for Edge RAG by using the domain URL you added to the local "hosts" file. For example: `https://arcrag.contoso.com`.
1. Select **Get started**. Then, follow the next steps at the end of this article to add a data source and set up the data query.

## (Optional) Clean up resources 

If you're done trying out Edge RAG, remove the resources created in this quickstart by running the following command:

```azurecli
az group delete `
   --name $rg `
   --yes `
   --no-wait
```

## Next step

> [!div class="nextstepaction"]
> [Add Data Source for Edge RAG](add-data-source.md)
