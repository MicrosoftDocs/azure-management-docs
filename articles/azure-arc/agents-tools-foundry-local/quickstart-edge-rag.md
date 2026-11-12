---
title: "Quickstart: Deploy Agents and Tools with Foundry Local"
description: "Learn how to deploy Agents and Tools with Foundry Local on Azure Kubernetes Service (AKS) without the need for local hardware."
ms.reviewer: assafakiva
ms.service: azure-arc
ms.topic: quickstart
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#customer intent: As a user, I want to deploy Agents and Tools with Foundry Local on Azure Kubernetes Service so that I can assess the solution.
---

# Quickstart: Deploy Agents and Tools with Foundry Local

In this quickstart, you deploy Agents and Tools with Foundry Local on Azure Kubernetes Service (AKS) without local hardware like Azure Local. Create and configure an AKS cluster, add required node pools and extensions, and deploy the Agents and Tools with Foundry Local extension. Configure the deployment to use a Foundry Local model endpoint (`gpt-oss-20b`) for language model inference and then connect to the developer portal to continue setup. This quickstart is intended for evaluation and development scenarios. For a production deployment, see [Deployment overview](deploy-overview.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, ensure you have:

- An active Azure subscription. If you don't have a service subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Azure CLI, Helm, kubectl, and the extensions aksarc and Kubernetes-extension installed locally unless you plan to use [Azure Cloud Shell](/azure/cloud-shell/get-started/ephemeral?tabs=azurecli). If you're not using Azure Cloud Shell, see [Script to configure machine to manage Azure Arc-enabled Kubernetes cluster](configure-driver-machine.md).
- Agents and Tools with Foundry Local registered as an application in Microsoft Entra ID (for example, named `agents-tools-foundry-local`). The app registration must be single-tenant, expose `api://<client-id>` as its Application ID URI, and set `requestedAccessTokenVersion` to `2` in the app manifest. Keep the `EdgeRAGDeveloper` app role for developer portal sign-in, fallback management access, and full agent definitions. After deployment, assign Azure roles as described later in this quickstart. See [Configure authentication](prepare-authentication.md).
- Application (client) ID and the directory (tenant) ID. To get these values after registering Agents and Tools with Foundry Local, see [Get app and tenant IDs](prepare-authentication.md#optional-get-app-and-tenant-ids).
- A Foundry Local endpoint and model deployment for your language model. For setup steps, see [Set up language model endpoint for Agents and Tools with Foundry Local](prepare-model-endpoint.md#foundry-local).
- A Foundry Local application registration client ID (`foundryClientId`) for Entra-based authentication between Agents and Tools with Foundry Local and the Foundry Local endpoint. For setup guidance, see [Register a Foundry Local application](prepare-authentication.md#optional-register-a-foundry-local-application).
- A domain name you can use for `networking.domainName` that matches your app registration redirect URI (for example, `arcrag.contoso.com`).
- Sufficient AKS quota in your target region for the node pool sizes used in this article, including `Standard_NC24ads_A100_v4` for the GPU node pool.

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

Create a resource group to hold the AKS cluster, node pool, and Agents and Tools with Foundry Local resources.

```azurecli
$rg = "edge-rag-aks-rg" 
$location = "eastus2"
az group create `
     --name $rg `
     --location $location
```

## Create and configure an AKS cluster

In this section, you create an AKS cluster and configure it for Agents and Tools with Foundry Local deployment. The steps include setting up the cluster, connecting it to Azure Arc, and preparing it with the necessary extensions and GPU support.

1. Create an AKS cluster:

   ```azurecli
   $k8scluster = "edge-rag-aks"
   az aks create `
      --resource-group $rg `
      --name $k8scluster `
      --node-count 2 `
      --network-policy calico `
      --generate-ssh-keys
   ```

1. Set the rest of the values as needed and then run the command. If you created the application registration for Agents and Tools with Foundry Local in a different tenant from the AKS cluster, set the values for `$entraAppId` and `$entraTenantId` by using the **Application (client) ID** and **Directory (tenant) ID** on the **agents-tools-foundry-local** app registration page in the Azure portal.

   ```azurecli
   # Set your Foundry Local model endpoint
   $apiEndpoint = "https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions"
   $apiModel = "gpt-oss-20b"
   $maxTokensInK = "128"  # Adjust based on your model's context window
   $foundryClientId = "<foundry_app_registration_client_id>"

   # Set Agents and Tools with Foundry Local extension values
   $gpu_enabled = "true" # set to false if no GPU nodes
   $localextname = "edgeragdemo"
   $autoUpgrade = "false"
   $extension = "microsoft.arc.rag" # do not change
   $n = "arc-rag" # do not change

   # Set Entra ID app registration values
   $domainName = "arcrag.contoso.com" # Edit to match the domain used in your registration
   $entraAppId = $(az ad app list --display-name "agents-tools-foundry-local" --query "[].appId" --output tsv)  # Display name is the application name in your registration
   $entraTenantId = $(az account show --query tenantId --output tsv) # Directory or tenant ID
   ```

   If you get a warning when setting `$entraAppId` or `$entraTenantId` by using the queries, set the values by using the **Application (client) ID** and **Directory (tenant) ID** on the **agents-tools-foundry-local** app registration page in the Azure portal.

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

1. Register the `Microsoft.Kubernetes` and `Microsoft.KubernetesConfiguration` providers by running the following commands:

   ```azurecli
   az provider register --namespace Microsoft.Kubernetes
   az provider register --namespace Microsoft.KubernetesConfiguration
   ```

1. Connect the AKS cluster to Azure Arc:

   ```azurecli
   az connectedk8s connect `
      --resource-group $rg ` 
      --location $location ` 
      --name $k8scluster  
   ```

   If prompted, select **y** to install the extension "connectedk8s".

1. Install the required certificate and trust manager as an Azure Arc extension. This installation includes the trust-manager parameters required for Foundry Local certificate handling.

   ```azurecli
   az k8s-extension create `
     --cluster-name $k8scluster `
     --name "azure-cert-manager" `
     --resource-group $rg `
     --cluster-type connectedClusters `
     --extension-type Microsoft.CertManagement `
     --scope cluster `
     --release-train stable `
     --config config.enableGatewayAPI=true `
     --config cert-manager.crds.keep=true `
     --config trust-manager.defaultPackage.enabled=false `
     --config trust-manager.secretTargets.enabled=true `
     --config trust-manager.secretTargets.authorizedSecretsAll=true
   ```

## Create node pools

Add dedicated GPU and CPU node pools to your AKS cluster to support Agents and Tools with Foundry Local.

If you get an error message when you try to create the node pools, you might need to request a quota increase for your Azure subscription, try a different virtual machine size, or create the Azure Kubernetes cluster and node pools in a different [Azure region](/azure/reliability/regions-list). For more information, see [Limits for resources, SKUs, and regions in Azure Kubernetes Service (AKS)](/azure/aks/quotas-skus-regions).

1. Run the following command to create a GPU node pool with nodes:

   ```azurecli
   az aks nodepool add ` 
       --resource-group $rg ` 
       --cluster-name $k8scluster ` 
       --name "gpunodepool" ` 
       --node-count 2 ` 
       --node-vm-size "Standard_NC24ads_A100_v4" `
       --enable-cluster-autoscaler `
       --min-count 2 ` 
       --max-count 2 `
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

1. Before you deploy Agents and Tools with Foundry Local, follow [Create a Foundry Local model endpoint](prepare-model-endpoint.md#foundry-local) to install the `inference-operator` (`Microsoft.Foundry`) extension and deploy the `gpt-oss-20b` `ModelDeployment` on `$k8scluster`. Wait until the `inference-operator` pods and the `gpt-oss-20b` model pod are running in the `foundry-local-operator` namespace.

## Deploy Agents and Tools with Foundry Local on AKS

Complete the following steps to deploy the Agents and Tools with Foundry Local extension onto your AKS cluster.

1. Deploy the Agents and Tools with Foundry Local extension by running the following command:

   ```azurecli
   az k8s-extension create `
       --cluster-type connectedClusters `
       --cluster-name $k8scluster `
       --resource-group $rg `
       --name $localextname `
       --extension-type $extension `
       --debug --release-train preview `
       --auto-upgrade-minor-version $autoUpgrade `
       --configuration-settings isManagedIdentityRequired=true `
       --configuration-settings indexedSourcesEnabled="true" `
       --configuration-settings gpu_enabled=$gpu_enabled `
       --configuration-settings AgentOperationTimeoutInMinutes=30 `
       --configuration-settings auth.tenantId=$entraTenantId `
       --configuration-settings auth.clientId=$entraAppId `
       --configuration-settings networking.mode=envoy `
       --configuration-settings networking.domainName=$domainName `
       --configuration-settings byom.enabled="true" `
       --configuration-settings byom.apiEndpoint=$apiEndpoint `
       --configuration-settings byom.apiModel=$apiModel `
       --configuration-settings byom.maxTokensInK=$maxTokensInK `
       --configuration-settings foundryClientId=$foundryClientId
   ```

   Wait several minutes for the deployment to complete. If your agents also use managed identity to call the Foundry Local model endpoint, include `--configuration-settings 'serviceUrls=["https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/"]'` or update the `mi-destinations` ConfigMap in `model-access-arc-rag` after deployment. See [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity).

1. Get the load balancer VIP by running the following command:

   ```azurecli
   kubectl get service `
     --namespace arc-rag `
     --selector gateway.envoyproxy.io/owning-gateway-name=edge-rag-gateway `
     --output jsonpath="{.items[0].status.loadBalancer.ingress[0].ip}"
   ```

   The command returns the gateway address. Use that value as `<load_balancer_ip>` later in this quickstart.

## Assign Azure roles for API access

Complete this step for all authenticated deployments, whether your model uses managed identity or an API key. An administrator with role-assignment permission must assign these roles.

1. Get the extension resource ID and the extension's system-assigned managed identity principal ID:

   ```azurecli
   $EXTENSION_SCOPE = $(az k8s-extension show `
      --resource-group $rg `
      --cluster-name $k8scluster `
      --cluster-type connectedClusters `
      --name $localextname `
      --query "id" -o tsv)

   $EXTENSION_PRINCIPAL_ID = $(az k8s-extension show `
      --resource-group $rg `
      --cluster-name $k8scluster `
      --cluster-type connectedClusters `
      --name $localextname `
      --query "identity.principalId" -o tsv)
   ```

1. Assign **Reader** to that identity at the extension scope:

   ```azurecli
   az role assignment create `
      --assignee-object-id $EXTENSION_PRINCIPAL_ID `
      --assignee-principal-type ServicePrincipal `
      --role "Reader" `
      --scope $EXTENSION_SCOPE
   ```

   Without this assignment or an inherited Reader assignment, Responses and Conversations can return `503` with `Authorization service unavailable`.

1. Assign **Cognitive Services Contributor** to the developer who manages resources, and assign **Reader** and **Cognitive Services OpenAI User** to the caller who lists and runs agents:

   ```azurecli
   az role assignment create `
      --assignee-object-id "<developer-object-id>" `
      --assignee-principal-type User `
      --role "Cognitive Services Contributor" `
      --scope $EXTENSION_SCOPE

   az role assignment create `
      --assignee-object-id "<caller-object-id>" `
      --assignee-principal-type User `
      --role "Reader" `
      --scope $EXTENSION_SCOPE

   az role assignment create `
      --assignee-object-id "<caller-object-id>" `
      --assignee-principal-type User `
      --role "Cognitive Services OpenAI User" `
      --scope $EXTENSION_SCOPE
   ```

For chat-only users, assign **Reader** to list agents and **Cognitive Services OpenAI User** to run them. App roles alone don't allow Responses or Conversations. Allow time for Azure propagation; the service also caches allows for 300 seconds and denies for 30 seconds. For scope and group guidance, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

## Assign required roles for Foundry Local

If you use `foundryClientId`, also assign the following Azure RBAC roles after deployment. These separate grants allow the extension to call the model. Use connected-cluster scope for Foundry's ARM read permissions and Foundry-extension scope for model inference permission.

Run the following commands for the connected cluster, Foundry operator, and Agents and Tools with Foundry Local managed identities:

```azurecli
# Set resource scopes
$CLUSTER_SCOPE = $(az connectedk8s show -g $rg -n $k8scluster --query "id" -o tsv)
$FOUNDRY_SCOPE = $(az k8s-extension show -g $rg -c $k8scluster `
   -t connectedClusters --name inference-operator --query "id" -o tsv)

# Get managed identity principal IDs
$CLUSTER_PRINCIPAL_ID = $(az connectedk8s show -g $rg -n $k8scluster `
   --query "identity.principalId" -o tsv)
$FOUNDRY_PRINCIPAL_ID = $(az k8s-extension show -g $rg -c $k8scluster `
   -t connectedClusters --name inference-operator --query "identity.principalId" -o tsv)
$EXTENSION_PRINCIPAL_ID = $(az k8s-extension show -g $rg -c $k8scluster `
   -t connectedClusters --name $localextname --query "identity.principalId" -o tsv)

# Reader for the connected cluster identity
az role assignment create `
   --assignee-object-id $CLUSTER_PRINCIPAL_ID `
   --role "Reader" `
   --scope $CLUSTER_SCOPE `
   --assignee-principal-type ServicePrincipal

# Reader for the Foundry operator identity
az role assignment create `
   --assignee-object-id $FOUNDRY_PRINCIPAL_ID `
   --role "Reader" `
   --scope $CLUSTER_SCOPE `
   --assignee-principal-type ServicePrincipal

# Reader for the Agents and Tools with Foundry Local identity
az role assignment create `
   --assignee-object-id $EXTENSION_PRINCIPAL_ID `
   --role "Reader" `
   --scope $CLUSTER_SCOPE `
   --assignee-principal-type ServicePrincipal

# Model inference permission at the Foundry extension scope
az role assignment create `
   --assignee-object-id $EXTENSION_PRINCIPAL_ID `
   --role "Cognitive Services OpenAI User" `
   --scope $FOUNDRY_SCOPE `
   --assignee-principal-type ServicePrincipal
```

For troubleshooting, see [Configure Foundry Local inference authentication](configure-foundry-inference-authentication.md).

## Connect to the developer portal

Before you connect to the portal, verify that the pods in the `arc-rag` namespace are running:

```azurecli
kubectl get pods -n arc-rag
```

Update your host file on your local machine to connect to the developer portal for Agents and Tools with Foundry Local.

1. On your local machine, open Notepad in Administrator mode. 
1. Go to **File** > **Open** > `C:\windows\System32\drivers\etc` > **hosts**. If you can't see the `hosts` file, set the extension type to **All files**.

1. Add the following line at the end of the file where you replace `load_balancer_ip` with the load balancer IP, and edit the domain to match the app registration:

   `<load_balancer_ip> arcrag.contoso.com` 

   For example:

   ```markdown
   # Agents and Tools with Foundry Local developer portal
   172.16.0.0 arcrag.contoso.com
   ```

1. Save the file.
1. Open the domain URL you added to the local `hosts` file, such as `https://arcrag.contoso.com`, and sign in. After sign-in, you land on Chat at `/chat`.
1. To manage data, open `https://arcrag.contoso.com/developer`. The developer portal opens the **Data** page. Follow the next steps to add a data source and set up the data query.

## (Optional) Clean up resources 

If you're done trying out Agents and Tools with Foundry Local, remove the resources created in this quickstart by running the following command:

```azurecli
az group delete `
   --name $rg `
   --yes `
   --no-wait
```

## Next step

> [!div class="nextstepaction"]
> [Add data source](add-data-source.md)
