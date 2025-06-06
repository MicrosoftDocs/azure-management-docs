---
title: Complete Deployment Prerequisites for Edge RAG
description: "Learn how to complete deployment prerequisites for Edge RAG to ensure a successful setup for AI-powered applications."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 06/06/2025
ai-usage: ai-assisted

#CustomerIntent: As a cloud administrator or AI application developer, I want to complete the deployment prerequisites for Edge RAG so that I can ensure a successful setup and configuration of the environment for AI-powered applications.
ms.custom:
  - build-2025
---

# Complete deployment prerequisites for Edge RAG Preview, enabled by Azure Arc

Complete the following steps to prepare for your Edge RAG deployment.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, review [What you need for Edge RAG](requirements.md).


## Step 1: Ensure you have "contributor" permission at the subscription level

Verify that you have [Contributor](/azure/role-based-access-control/built-in-roles/privileged#contributor) permissions for the subscription you want to use. See [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).

Contributor role is required at the subscription level for the following commands:

```powershell
az provider register --namespace Microsoft.KubernetesConfiguration
az feature register --namespace Microsoft.KubernetesConfiguration --name extensions
```

## Step 2: Choose the right language model

Decide which language model your organization wants to deploy. You can use your own language model or use the Microsoft provided language models: 

- [Microsoft Phi 3.5 Mini](https://huggingface.co/microsoft/Phi-3.5-mini-instruct) and
- [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2). 

After Edge RAG extension is deployed, you can't change the language model. Therefore, work with your application development team to decide which is the right model for your organization's use case. 

You can refer to some of these resources from Microsoft to choose the right model for your use case:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Azure AI Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.

## Step 3: Verify NFS server is configured and reachable

Make sure all your documents and images are available on a supported network file system (NFS) server. That NFS server must be configured correctly and reachable from the Edge RAG deployment on Azure Local. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources) 
- [Configure the network file system server for Edge RAG](configure-nfs-server.md).

Use the NFS server you configured to host the documents and images you want to use for testing with Edge RAG.

## Step 4: Prepare AKS cluster on Azure Local

Create an Azure Kubernetes Service (AKS) cluster on the Azure Local instance with a node pool that meets the minimum requirements.

### Create an AKS Arc cluster

Create an AKS Arc cluster by using one of the following methods:

- Azure portal, see [How to deploy a Kubernetes cluster using the Azure portal](/azure/aks/hybrid/aks-create-clusters-portal)

- Azure CLI, see [Create Kubernetes clusters using Azure CLI](/azure/aks/hybrid/aks-create-clusters-cli)

### Install supported GPU drivers (optional)

AKS Arc only supports Nvidia A2 and A16 GPUs. The following steps are applicable only to these two GPUs.

If you have GPUs available in your Azure Local instance you want to use for Edge RAG, make sure that the necessary GPU drivers are installed and available in the AKS Arc cluster nodes.

To check if the right drivers are already installed and the GPUs are available to the AKS Arc cluster, run the following command:

```powershell
(Get-MocNode -location MocLocation).properties.statuses.Info`
```

If the output lists all the GPUs available on the Azure Local cluster, you can move to the next step. Otherwise, complete the following steps on any of the Azure Local cluster nodes to enable GPUs.

We recommend using the script in the [Enabling GPU on AKS on Azure ARC - sample](enable-gpu-aks.md) to enable GPUs for use by Edge RAG.

Alternately, you can follow the instructions in [Use GPUs for compute-intensive workloads](/azure/aks/hybrid/deploy-gpu-node-pool) and ensure you meet the [minimum hardware requirements](requirements.md#minimum-hardware-requirements) for the GPU mode. If you follow these instructions, you need to run the following command on each Hyper-V host in the Azure Local cluster:

```powershell
Restart-Service wssdagent -Force -Verbose 
Start-sleep 60
(Get-MocNode -location MocLocation).properties.statuses.Info
```

Make sure that all the available GPUs across all nodes are listed in the output of the command.

### Configure machine to manage Azure Arc-enabled Kubernetes clusters (optional)

If you want to manage the Kubernetes clusters from a machine outside of the Azure Local host, set up a machine with the following tools:

- Azure CLI
- Azure CLI extensions aksarc and Kubernetes-extension
- kubectl
- Helm

This driver machine must be able to connect to the Kubernetes cluster on the network.

To set up a Windows machine to manage your Kubernetes clusters, see the [Script to configure machine to manage Azure Arc-enabled Kubernetes cluster](configure-driver-machine.md).

### Create node pools for AKS Arc cluster

To create a node pool for AKS Arc, complete the following steps from the driver machine. 

1. Sign into Azure by using Azure CLI: `az login`.
1. Create the node pool.

   If GPUs are available:

   - You must create a node pool of at least three CPU VMs, with minimal size of "Standard_D8s_v3". Run the following command:
   
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

## Step 5: Configure authentication

Configure and control authentication to the Edge RAG for AI application developers and for end users of the chat endpoint. In this section, you create an application registration, create app roles, and assign users or groups to those roles. 

You might need to work with your Microsoft Entra or cloud administrator to configure authentication.

1. In the Azure portal, go to **Microsoft Entra ID**.
1. Go to the appropriate tenant and select **Manage** > **App registrations**.
1. Select **New registration** to create an application registration.

	:::image type="content" source="media/complete-prerequisites/new-app-registration.png" alt-text="Screenshot that shows the new registration option on the top of the application registration page.":::

1. Enter *EdgeRAG* for **Name**.

1. Select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant**).

1. Select **Register**.

	:::image type="content" source="media/complete-prerequisites/register-app.png" alt-text="Screenshot that shows the fields on the register an application page where you add an application name and select supported account types.":::

1. After the application is registered, go to the registration and select **Manage** > **Authentication**.

1. Select **Add a platform** > **Single-page application**.

1. Specify your domain name appended with */authorizing* (for example, `arcrag.contoso.com/authorizing`)  as the **Redirect URIs**.

	:::image type="content" source="media/complete-prerequisites/configure-application.png" alt-text="Screenshot that shows the single-page application page where you configure redirect URLs and more." lightbox="media/complete-prerequisites/configure-application.png":::

1. Select **Configure**.
1. For **Supported account types**, select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**.

	:::image type="content" source="media/complete-prerequisites/supported-account-types.png" alt-text="Screenshot that shows the options for the supported account types with the last option selected.":::

1. Select **+ Add a platform** > **Mobile and desktop applications**.
1. For **Redirect URIs**, select `https://login.microsoftonline.com/common/oauth2/nativeclient`.

1. Select **Configure**.
1. On the left-hand side menu, under **Manage**, select **App roles**.
1. Create two app roles. One for *EdgeRAGDeveloper* and another for *EdgeRAGEndUser*. Use the appropriate values listed in the table that follows the image.

	:::image type="content" source="media/complete-prerequisites/app-roles.png" alt-text="Screenshot that shows the two app roles created for the developer and user.":::


   |Field  |Value|
   |---------|---------|
   |Display name     |   *EdgeRAGDeveloper* or *EdgeRAGEndUser*      |
   |Allowed member types   |  User/Groups       |
   |Value    |    *EdgeRAGDeveloper* or *EdgeRAGEndUser*        |
   |Description    |  *EdgeRAGDeveloper* or *EdgeRAGEndUser*          |
   |Do you want to enable this app role?|Checked|

1. When complete, close the **App roles** page.
1. To assign users or groups to the role you created, on the tenant's left-hand side menu, under **Manage**, select **Enterprise applications**.
1. Search for and select the *EdgeRag* application you created.
1. Go to **Manage** > **Properties**.
1. Disable **Assignment Required**.
1. On the left-hand side menu, select **Users and groups** > **Add user/group**.
1. Select users and/or groups and assign **EdgeRAGDeveloper** or **EdgeRAGEndUser** role as appropriate.

## Step 6: Install networking and observability components

From the driver machine, install and configure MetalLB for the AKS Arc cluster and the observability dependency modules.

1. **Install MetalLB**

   Skip this step if MetalLB is installed and configured in the current AKS Arc cluster.

   To install and [configure](/azure/aks/hybrid/deploy-load-balancer-cli) MetalLB, you can run the following commands on any of the cluster nodes in the Azure Local instance:


	```powershell
	$lbName = "metallb"
	$ipRange = ""   # <------ Provide the static IP address range that will be assigned to metalLB (format: CIDR format E.g. <IP address>-<IP address> or <IP address>/32)
	$sub = "<Subscription GUID>"
	$rg = "<Resource Group name>"
	$k8scluster = "<AKS Arc cluster name>"
	az extension add -n k8s-runtime --upgrade 
	$resourceuri = "subscriptions/$sub/resourceGroups/$rg/providers/Microsoft.Kubernetes/connectedClusters/$k8scluster"
	az k8s-runtime load-balancer enable --resource-uri $resourceuri
	az k8s-runtime load-balancer create --load-balancer-name $lbName --resource-uri $resourceuri --addresses $ipRange --advertise-mode "ARP"
	```

1. **Install observability dependency modules**

   `Microsoft.iotoperations.platform` is a simple extension that installs certificate manager and trust manager modules. Run the following command to install the extension.


	```powershell
	$sub = "<Subscription GUID>"
	$rg = "<Resource Group name>"
	$k8scluster = "<AKS Arc cluster name>"
	az k8s-extension create -g $rg -c $k8scluster -t connectedClusters --scope cluster --name "cert-manager" --release-namespace "cert-manager" --release-train preview --extension-type "Microsoft.iotoperations.platform" --debug
	```

    if the `Microsoft.iotoperations.platform` extension isn't available in your region, use following steps to install the required certificate and trust manager.

	```powershell
    # Install Cert-Manager and Trust-Manager 

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.3/cert-manager.yaml --wait  
    helm repo add jetstack https://charts.jetstack.io --force-update  
    start-sleep -Seconds 20 
    helm upgrade trust-manager jetstack/trust-manager --install --namespace cert-manager --wait 

## Step 7: Configure DNS

Make sure you configure your DNS to map the IP address assigned to MetalLB to the domain name of the local portal like `arcrag.contoso.com`.

Alternately, update your local machine (laptop or DVM) to reach the local portal domain by using the following steps:

1. Open a text editor like Notepad with **Administrator** privileges.

2. Then open file "**C:\Windows\System32\drivers\etc\hosts**"

3. At the end of the file, add a line: "100.XX.XX.XX arcrag.contoso.com" where you replace the IP address and url for your local portal.

4. Save the file

5. In the browser, go to your local portal like `https://arcrag.contoso.com`.

## Related content

[Deploy the Edge RAG extension](deploy.md)