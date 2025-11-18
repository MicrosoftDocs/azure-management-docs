---
title: Deploy the Edge RAG Extension
description: "Learn how to deploy the Edge RAG extension by using either Azure CLI or the Azure portal."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 06/10/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer Intent: As a cloud administrator or developer, I want to deploy the Edge RAG extension using Azure CLI or the Azure portal so that I can enable advanced language model capabilities on my Azure Kubernetes Service (AKS) Arc cluster to provide an intelligent chat solutions.
---
# Deploy the extension for Edge RAG Preview enabled by Azure Arc

After you complete the prerequisites steps, complete the steps in this article to deploy Edge RAG extension.

To try Edge RAG without the need for local hardware, see [Quickstart: Install Edge RAG Preview enabled by Azure Arc](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, [complete the deployment prerequisites for Edge RAG Preview](complete-prerequisites.md).

## Deploy the extension

Deploy Edge RAG by using either the Azure portal or Azure CLI with an Edge RAG supplied language model or use your own language model.

#### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com/), go to the Azure Kubernetes cluster on Azure Local. 
1. Select **Settings** > **Extensions** > **+ Add**, and **Edge RAG** from the list.

   :::image type="content" source="media/deploy/add-cluster-extension.png" alt-text="Screenshot of the extensions you can add from the cluster with Edge RAG highlighted." lightbox="media/deploy/add-cluster-extension.png":::
1. On the **Basics** tab, provide the following information:

   | Field      | Value                                                        |
   |-----------------|--------------------------------------------------------------|
   | Subscription    | Select the subscription that contains your Azure Kubernetes Service (AKS) cluster on Azure Local. |
   | Resource group  | Select the resource group that contains your AKS Arc cluster. |
   | Deployment name | Provide a name for the deployment.                           |
   | Region          | Select the region to deploy Edge RAG.                        |
   | Cluster         | Select the cluster that you want to deploy Edge RAG to.      |

   :::image type="content" source="media/deploy/install-extension.png" alt-text="Screenshot of the basic tab with fields to enter the project and instance details.":::

1. Select **Next: Configuration**.
1. On the Configuration tab, provide the following information:

   | Field      | Value                                                                                           |
   |-----------------|-------------------------------------------------------------------------------------------------|
   | Deployment mode | Select GPU mode or CPU mode depending on your available hardware.                               |
   |**Model**| The information you enter in this section  depend on the language model you select.|
   |Language model source         | Select the language model that you want to deploy. Choose either an Edge RAG-provided language model or bring own language model (BYOM).                                              |
   |Language model name|If you chose to use a provided model, select one of the Edge RAG-provided language models.|
   |**Add your own language model**|If you chose to bring your own language model, enter the following information.|
   |Model name|Enter the name of your language model.|
   |LLM endpoint|Enter the name of your large language model (LLM) endpoint in the format `http://some-endpoint` or `https://some-endpoint`. For example, `https://<Endpoint_Name>.openai.azure.com/openai/deployments/<model_name> /chat/completions?api-version=<API_VERSION>`. |
   |Max token (k)|Enter a number range between 4K to 2048 K for your language model.|
   |**SSL settings**||
   |SSL CNAME           | Provide the domain name for your system. This domain name is the same as redirect URI provided during app registration.|
   |Kubernetes SSL secret name     | Provide a friendly name for the SSL secret to be used by the application. By default, Edge RAG uses a self-signed SSL certificate to store under this name in the kubernetes secret store. After installation, you can update the certificate with an official signed certificate.               |
   |**Access**||
   | Entra app ID    | Provide the application ID from the app you registered as part of configuring authentication (App Registrations > Your app > Overview). |
   | Entra tenant ID | Provide tenant ID from the app you registered as part of configuring authentication (App Registrations > Your app > Overview). |

    :::image type="content" source="media/deploy/install-extension-configurations.png" alt-text="Screenshot of the configuration tab where you select the model type and other configurations.":::

1. Select **Review + create**.
1. Review and validate the parameters you provided.
1. Select **Create** to complete the Edge RAG deployment.
1. When the deployment is complete, under **Extensions**, validate that the extension types **microsoft.arc.rag** and **microsoft.extensiondiagnostics** are listed.

#### [Azure CLI](#tab/azure-cli)

1. Set the values for the parameters in the following command and then run the command.

   ```powershell
   $gpu_enabled = "true"  # Mark it true if you have GPUs available for Edge RAG
   $localextname = "edgeragdemo" # Once used do not change
   $autoUpgrade = "false"
   $tenantId = "<App Tenant ID>" # App registrations -> Your app -> Overview on Azure portal
   $appId = "<App ID>" # App registrations -> Your app -> Overview on Azure portal
   $domainName = "arcrag.contoso.com" # App redirect URI and this domain name should be the same
   $sub = "<Subscription GUID>"
   $rg = "<Resource Group name>"
   $k8scluster = "<Azure Kubernetes Service (ASK) Arc cluster name>"
   $extension = "microsoft.arc.rag" # do not change
   $n = "arc-rag" # do not change
   ```

1. Set the values needed for either the Edge RAG-provided language model or your own language model.
   - Edge-RAG supplied language model option: Edit the following command as appropriate and run the command.

     ```powershell
     $modelName = "microsoft/Phi-3.5" # If you want to switch to Mistral 7B, change this variable to "mistralAI/Mistral-7B" 
     ```

   - Bring your own language model option:  Edit the following command as appropriate and run the command.

      ```powershell
      $apiEndpoint = <Endpoint URI> 
      $apiModel = <Model Name> 
      $maxTokensInK = <Max Tokens In K (e.g. 10, 20 etc.)> 
     ```

1. After you populate the parameter values, deploy the Azure Arc extension by running the command for either the  supplied language model or your own language model:

   - Edge RAG supplied language model option: Run the following command.

     ```powershell
     az provider register --namespace Microsoft.KubernetesConfiguration
     az feature register --namespace Microsoft.KubernetesConfiguration --name extensions 

     az k8s-extension create --cluster-type connectedClusters --cluster-name $k8scluster --resource-group $rg --name $localextname --extension-type $extension --debug --release-train preview --auto-upgrade $autoUpgrade `
        --configuration-settings isManagedIdentityRequired=true --configuration-settings gpu_enabled=$gpu_enabled --configuration-settings AgentOperationTimeoutInMinutes=30  `
        --configuration-settings model=$modelName --configuration-settings auth.tenantId=$tenantId --configuration-settings auth.clientId=$appId --configuration-settings ingress.domainname=$domainName
     ```

   - Add your own language model option: Run the following command.

     ```powershell
     az k8s-extension create --cluster-type connectedClusters --cluster-name $k8scluster --resource-group $rg --name $localextname --extension-type $extension --debug --release-train preview --auto-upgrade $autoUpgrade ` 
         --configuration-settings isManagedIdentityRequired=true --configuration-settings gpu_enabled=$gpu_enabled --configuration-settings AgentOperationTimeoutInMinutes=30  `     
         --configuration-settings auth.tenantId=$tenantId --configuration-settings auth.clientId=$appId --configuration-settings ingress.domainname=$domainName `   
         --configuration-settings byom.enabled="true" --configuration-settings byom.apiEndpoint=$apiEndpoint --configuration-settings byom.apiModel=$apiModel --configuration-settings byom.maxTokensInK=$maxTokensInK 
     ```

----

The Edge RAG extension deployment typically takes about 30 minutes but can take longer depending on your connectivity.

## Bring your own language model

If you added your own language model when you deployed the Edge RAG extension, complete the steps in [Configure "BYOM" endpoint authentication for Edge RAG](configure-endpoint-authentication.md).

## Related content

- [Configure "BYOM" endpoint authentication for Edge RAG](configure-endpoint-authentication.md)
- [Custom certificate authority in Azure Kubernetes Service (AKS)](/azure/aks/custom-certificate-authority)
- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc)