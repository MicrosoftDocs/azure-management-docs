---
title: Deploy the Agents and Tools with Foundry Local Extension
description: "Learn how to deploy the Agents and Tools with Foundry Local extension by using either Azure CLI or the Azure portal."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer Intent: As a cloud administrator or developer, I want to deploy the Agents and Tools with Foundry Local extension using Azure CLI or the Azure portal so that I can enable advanced language model capabilities on my Azure Kubernetes Service (AKS) Arc cluster to provide an intelligent chat solutions.
---
# Deploy the extension for Agents and Tools with Foundry Local

After you complete the prerequisite steps, complete the steps in this article to deploy the Agents and Tools with Foundry Local extension.

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, [complete the deployment prerequisites for Agents and Tools with Foundry Local](complete-prerequisites.md).

## Deploy the extension

Deploy Agents and Tools with Foundry Local by using either the Azure portal or Azure CLI.

#### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com/), go to the Azure Kubernetes cluster on Azure Local. 
1. Select **Settings** > **Extensions** > **+ Add**, and **Agentic RAG** from the list.

   :::image type="content" source="media/deploy/add-cluster-extension.png" alt-text="Screenshot of the extensions you can add from the cluster with Agentic RAG highlighted." lightbox="media/deploy/add-cluster-extension.png":::
1. On the **Basics** tab, provide the following information:

   | Field      | Value                                                        |
   |-----------------|--------------------------------------------------------------|
   | Subscription    | Select the subscription that contains your Azure Kubernetes Service (AKS) cluster on Azure Local. |
   | Resource group  | Select the resource group that contains your AKS Arc cluster. |
   | Deployment name | Provide a name for the deployment.                           |
   | Region          | Select the region to deploy Agents and Tools with Foundry Local.                        |
   | Cluster         | Select the cluster that you want to deploy Agents and Tools with Foundry Local to.      |

   :::image type="content" source="media/deploy/install-extension.png" alt-text="Screenshot of the basic tab with fields to enter the project and instance details.":::

1. Select **Next: Configuration**.
1. On the Configuration tab, provide the following information:

   | Field      | Value                                                                                           |
   |-----------------|-------------------------------------------------------------------------------------------------|
   | Deployment mode | Select GPU mode or CPU mode depending on your available hardware.                               |
   | Deployment mode | Select the deployment mode: **Agentic and Knowledge** (default), **Agentic Only**, or **Knowledge Only**. |
   |**Language model (BYOM ΓÇö required)**| |
   |Model name|Enter the name of your language model.|
   |LLM endpoint|Enter the URL of your OpenAI-compatible LLM endpoint. For example, for Microsoft Foundry: `https://<Foundry_Resource_Name>.openai.azure.com/openai/deployments/<model_name>/chat/completions?api-version=<API_VERSION>`. For Foundry Local on Azure Local, use the cluster-internal endpoint from your model deployment. |
   |Max token (k)|Enter a number range between 4K to 2048 K for your language model.|
   |**SSL settings**||
   |SSL CNAME           | Provide the domain name for your system. This domain name is the same as redirect URI provided during app registration.|
   |Kubernetes SSL secret name     | Provide a friendly name for the SSL secret to be used by the application. By default, Agents and Tools with Foundry Local uses a self-signed SSL certificate to store under this name in the kubernetes secret store. After installation, you can update the certificate with an official signed certificate.               |
   |**Access**||
   | Entra app ID    | Provide the application ID from the app you registered as part of configuring authentication (App Registrations > Your app > Overview). |
   | Entra tenant ID | Provide tenant ID from the app you registered as part of configuring authentication (App Registrations > Your app > Overview). |

    :::image type="content" source="media/deploy/install-extension-configurations.png" alt-text="Screenshot of the configuration tab where you select the model type and other configurations.":::

1. Select **Review + create**.
1. Review and validate the parameters you provided.
1. Select **Create** to complete the Agents and Tools with Foundry Local deployment.
1. When the deployment is complete, under **Extensions**, validate that the extension types **microsoft.arc.rag** and **microsoft.extensiondiagnostics** are listed.

#### [Azure CLI](#tab/azure-cli)

1. Set the values for the parameters in the following command and then run the command.

   ```powershell
   $gpu_enabled = "true"  # Mark it true if you have GPUs available for Agents and Tools with Foundry Local
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

1. Set the values for your language model and deployment mode:

   ```powershell
   # BYOM is mandatory ΓÇö set your model endpoint
   $apiEndpoint = "<Your OpenAI-compatible LLM endpoint URI>"
   $apiModel = "<Model Name>"
   $maxTokensInK = "<Max Tokens In K (e.g. 10, 20 etc.)>"
   $layerSelection = "combined" # Options: combined, agentic, knowledge
   ```

1. After you populate the parameter values, create the BYOM API key secret and deploy the Azure Arc extension:

   ```powershell
   # Create the BYOM API key secret before extension installation
   kubectl create namespace arc-rag --dry-run=client -o yaml | kubectl apply -f -
   kubectl delete secret byom-api-key -n arc-rag --ignore-not-found
   kubectl create secret generic byom-api-key --from-literal=BYOM_API_KEY="<your_foundry_api_key>" -n arc-rag
   ```

   > [!NOTE]
   > The BYOM API key is stored as a Kubernetes secret rather than passed as a configuration parameter. You must create this secret before you install the extension. For Foundry Local, retrieve the key by running `kubectl get secret gpt-oss-20b-api-keys -n foundry-local-operator -o jsonpath="{.data.primary-key}" | base64 -d`.

   ```powershell
   az provider register --namespace Microsoft.KubernetesConfiguration
   az feature register --namespace Microsoft.KubernetesConfiguration --name extensions 

   az k8s-extension create --cluster-type connectedClusters --cluster-name $k8scluster --resource-group $rg --name $localextname --extension-type $extension --debug --release-train preview --auto-upgrade $autoUpgrade `
       --configuration-settings isManagedIdentityRequired=true --configuration-settings gpu_enabled=$gpu_enabled --configuration-settings AgentOperationTimeoutInMinutes=30 `
       --configuration-settings auth.tenantId=$tenantId --configuration-settings auth.clientId=$appId --configuration-settings ingress.domainname=$domainName `
       --configuration-settings layerSelection=$layerSelection `
       --configuration-settings byom.enabled="true" --configuration-settings byom.apiEndpoint=$apiEndpoint --configuration-settings byom.apiModel=$apiModel --configuration-settings byom.maxTokensInK=$maxTokensInK `
       --configuration-settings foundryClientId="<foundry_app_registration_client_id>"
   ```

----

The Agents and Tools with Foundry Local extension deployment typically takes about 30 minutes but can take longer depending on your connectivity.

## Bring your own language model

After deploying the Agents and Tools with Foundry Local extension, complete the steps in [Configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md).

## Verify deployment by mode

After deployment, verify the components running in the `arc-rag` namespace match your selected deployment mode:

| Mode | Expected pods |
|---|---|
| **Combined** | All Knowledge Layer pods (ingestionapi, inferencingflow, vectordb-api-server, embedding models, docling, milvus, postgres) + all Agentic Layer pods (agent-manager, agents-runtime, knowledge-sources, indexed-sources-mcp-server) |
| **Agentic** | Agentic Layer pods only (agent-manager, agents-runtime, knowledge-sources, indexed-sources-mcp-server, postgres) |
| **Knowledge** | Knowledge Layer pods only (ingestionapi, inferencingflow, vectordb-api-server, embedding models, docling, milvus, postgres) |

Run the following command to check:

```bash
kubectl get pods -n arc-rag
```

## Verify end-to-end connectivity

After deployment, verify that the extension can communicate with Foundry Local:

1. Check Foundry model availability (port-forward since `endpoint.enabled=false`):

   ```bash
   kubectl port-forward svc/gpt-oss-20b 5000:5000 -n foundry-local-operator
   # In another terminal:
   curl http://localhost:5000/v1/models
   ```

1. Test a chat completion via port-forward:

   ```bash
   curl -X POST http://localhost:5000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "gpt-oss-20b",
       "messages": [
         {"role": "user", "content": "Hello, what is 2+2?"}
       ],
       "temperature": 0.7,
       "max_tokens": 100
     }'
   ```

1. Test the extension's inference endpoint:

   ```bash
   curl -X POST http://localhost:3001/edgeai/chat/completions?api-version=2024-10-01-preview \
     -H "Content-Type: application/json" \
     -H "x-user-role: dev" \
     -d '{
       "messages": [{"role": "user", "content": "Test question"}],
       "data_sources": [{"type": "milvus", "parameters": {"endpoint": "", "index_name": "edgeragapp"}}]
     }'
   ```

## Related content

- [Deployment parameter reference and troubleshooting](deploy-reference.md)

- [Configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md)
- [Custom certificate authority in Azure Kubernetes Service (AKS)](/azure/aks/custom-certificate-authority)
- [Configuring the chat solution for Agents and Tools with Foundry Local](build-chat-solution-overview.md)
- [Add data source for the chat solution in Agents and Tools with Foundry Local](add-data-source.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc)
