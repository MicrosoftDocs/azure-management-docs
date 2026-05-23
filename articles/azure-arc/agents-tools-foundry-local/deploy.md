---
title: Deploy the Agents and Tools with Foundry Local Extension
description: "Learn how to deploy the Agents and Tools with Foundry Local extension by using either Azure CLI or the Azure portal."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/23/2026
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
1. Select **Settings** > **Extensions** > **+ Add**, and **Edge RAG** from the list.

   :::image type="content" source="media/deploy/add-cluster-extension.png" alt-text="Screenshot of the extensions you can add from the cluster with Agentic RAG highlighted." lightbox="media/deploy/add-cluster-extension.png":::
1. On the **Basics** tab, provide the following information:

   | Field      | Value                                                        |
   |-----------------|--------------------------------------------------------------|
   | Subscription    | Select the subscription that contains your Azure Kubernetes Service (AKS) cluster on Azure Local. |
   | Resource group  | Select the resource group that contains your AKS Arc cluster. |
   | Deployment name | Provide a name for the deployment.                           |
   | Region          | Select the region to deploy Agents and Tools with Foundry Local.                        |
   | Cluster         | Select the cluster that you want to deploy Agents and Tools with Foundry Local to.      |


   :::image type="content" source="media/deploy/install-extension.png" alt-text="Screenshot of the basic tab with fields to enter the project and instance details." lightbox="media/deploy/install-extension.png":::

1. Select **Next**.
1. On the **Configurations** tab, provide the following information:

   | Field | Value |
   |---|---|
   | **RAG settings** | |
   | RAG components | Select the components to install: **Select all** (default), **Agentic Retrieval Engine**, or **Knowledge Sources Layer**. |
   | **Deployment mode** | |
   | Deployment mode | Select **GPU** or **CPU** based on your available hardware. This setting applies to the Knowledge Sources layer. |
   | **NFS connection (data source)** | |
   | Kerberos | Optional. Select this option if you want to connect to an NFS server by using Kerberos authentication. |
   | Kerberos SPN | Required only when **Kerberos** is selected. Enter the SPN in the format `service/host@REALM` (for example, `nfs/edgerag-svc@CONTOSO.COM`). |
   | **SharePoint connection (data source)** | |
   | SharePoint ingestion | Optional. Select this option if you want to connect to SharePoint by using Key Vault authentication. |
   | Key Vault name | Required only when **SharePoint ingestion** is selected. Enter the Azure Key Vault name. |
   | KV cert secret name | Required only when **SharePoint ingestion** is selected. Enter the Key Vault secret name that stores the certificate. |
   | KV cert password secret name | Required only when **SharePoint ingestion** is selected. Enter the Key Vault secret name that stores the certificate password. |
   | Workload identity client ID | Required only when **SharePoint ingestion** is selected. Enter the workload identity client ID (GUID). |
   | **Inference model** | |
   | Language model source | Select **Foundry Local** or **Bring your own**. |
   | Application ID | Required only when **Foundry Local** is selected.  |
   | Language model name | Required. Enter your deployed language model name. |
   | LLM endpoint | Required. Enter your OpenAI-compatible endpoint URL. For example: `https://<Foundry_Resource_Name>.openai.azure.com/openai/deployments/<model_name>/chat/completions?api-version=<API_VERSION>`. For Foundry Local on Azure Local, use your cluster-internal endpoint. |
   | Max token (K) | Required. Enter a value from 4K to 2048K. |

   :::image type="content" source="media/deploy/install-extension-configurations.png" alt-text="Screenshot of the configuration tab where you select the model type and other configurations." Lightbox="media/deploy/install-extension-configurations.png":::

1. Select **Next**.
1. On the **Access** tab, provide the following information:

   | Field | Value |
   |---|---|
   |**SSL settings**||
   |SSL CNAME|Enter the domain name for your system. The domain name should match the redirect URI used during app registration and must not include the `https://` prefix. For example, `arcrag.contoso.com`.|
   |Kubernetes SSL secret name|Enter the name of the Kubernetes secret to store the SSL certificate. By default, Agents and Tools with Foundry Local uses a self-signed SSL certificate in this secret. After installation, you can replace it with a signed certificate.|
   |**Access**||
   | Entra application ID|Enter the application ID from the enterprise application you registered for authentication.|
   | Entra tenant ID|Enter the tenant ID from the enterprise application you registered for authentication.|

   :::image type="content" source="media/deploy/install-extension-access.png" alt-text="Screenshot of the access tab with SSL settings and Entra application fields." lightbox="media/deploy/install-extension-access.png":::

1. Select **Review + create**.
1. Review and validate the parameters you provided.
1. Select **Create** to complete the Agents and Tools with Foundry Local deployment.
1. When the deployment is complete, under **Extensions**, validate that the extension types **microsoft.arc.rag** and **microsoft.extensiondiagnostics** are listed.

#### [Azure CLI](#tab/azure-cli)

1. Use one base deployment script and set optional parameters only for the connection types you enable.

   Before you run the script, choose one deployment path:

   - Kerberos only: set `enableKerberos=true` and `enableSharePoint=false`.
   - SharePoint ingestion only: set `enableKerberos=false` and `enableSharePoint=true`.
   - Kerberos and SharePoint ingestion: set both values to `true`.

   If you enable Kerberos, complete [Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local](connect-file-share-kerberos-setup.md) first. If you enable SharePoint ingestion, complete [Set up SharePoint server-to-server authentication for Agents and Tools with Foundry Local](connect-sharepoint-setup.md) first.

1. Set common deployment values:

   ```powershell
   $gpu_enabled = "true"  # Mark it true if you have GPUs available for Agents and Tools with Foundry Local
   $localextname = "edgeragdemo" # Once used do not change
   $autoUpgrade = "false"
   $tenantId = "<App Tenant ID>" # App registrations -> Your app -> Overview on Azure portal
   $appId = "<App ID>" # App registrations -> Your app -> Overview on Azure portal
   $domainName = "arcrag.contoso.com" # App redirect URI and this domain name should be the same
   $sub = "<Subscription GUID>"
   $rg = "<Resource Group name>"
   $k8scluster = "<Azure Kubernetes Service (AKS) Arc cluster name>"
   $extension = "microsoft.arc.rag" # do not change
   $n = "arc-rag" # do not change
   $foundryClientId = "<foundry_app_registration_client_id>"
   ```

1. Set values for the language model, deployment mode, and optional connection types:

   ```powershell
   # BYOM is mandatory - set your model endpoint
   $apiEndpoint = "<Your OpenAI-compatible LLM endpoint URI>"
   $apiModel = "<Model Name>"
   $maxTokensInK = "<Max Tokens In K (e.g. 10, 20 etc.)>"
   $layerSelection = "combined" # Options: combined, agentic, knowledge

   # Optional connection switches
   $enableKerberos = "false"
   $enableSharePoint = "false"

   # Kerberos setting (required only when enableKerberos=true)
   $kerberosSpn = "nfs/edgerag-svc@CONTOSO.COM"

   # SharePoint settings (required only when enableSharePoint=true)
   $keyVaultName = "<Key Vault name>"
   $kvCertSecretName = "edgerag-sp-s2s-cert"
   $kvCertPasswordSecretName = "sp-cert-password"
   $workloadIdentityClientId = "<Managed identity client ID>"
   ```

1. Create the BYOM API key secret:

   ```powershell
   # Create the BYOM API key secret before extension installation
   kubectl create namespace arc-rag --dry-run=client -o yaml | kubectl apply -f -
   kubectl delete secret byom-api-key -n arc-rag --ignore-not-found
   kubectl create secret generic byom-api-key --from-literal=BYOM_API_KEY="<your_foundry_api_key>" -n arc-rag
   ```

   The BYOM API key is stored as a Kubernetes secret rather than passed as a configuration parameter. You must create this secret before you install the extension. For Foundry Local, retrieve the key by running `kubectl get secret gpt-oss-20b-api-keys -n foundry-local-operator -o jsonpath="{.data.primary-key}" | base64 -d`.

1. After you populate the values, build configuration settings and deploy the Azure Arc extension:

   ```powershell
   az provider register --namespace Microsoft.KubernetesConfiguration
    az feature register --namespace Microsoft.KubernetesConfiguration --name extensions

    $config = @(
       "isManagedIdentityRequired=true",
       "gpu_enabled=$gpu_enabled",
       "AgentOperationTimeoutInMinutes=30",
       "auth.tenantId=$tenantId",
       "auth.clientId=$appId",
       "ingress.domainname=$domainName",
       "layerSelection=$layerSelection",
       "byom.enabled=true",
       "byom.apiEndpoint=$apiEndpoint",
       "byom.apiModel=$apiModel",
       "byom.maxTokensInK=$maxTokensInK",
       "foundryClientId=$foundryClientId"
    )

    if ($enableKerberos -eq "true") {
       $config += @(
          "kerberos.enabled=true",
          "kerberos.spn=$kerberosSpn"
       )
    }

    if ($enableSharePoint -eq "true") {
       $config += @(
          "sharepoint.sharepointIngestionEnabled=true",
          "sharepoint.s2s.keyvaultName=$keyVaultName",
          "sharepoint.s2s.kvCertSecretName=$kvCertSecretName",
          "sharepoint.s2s.kvCertPasswordSecretName=$kvCertPasswordSecretName",
          "sharepoint.s2s.workloadIdentityClientId=$workloadIdentityClientId"
       )
    }

    az k8s-extension create `
       --cluster-type connectedClusters `
       --cluster-name $k8scluster `
       --resource-group $rg `
       --name $localextname `
       --extension-type $extension `
       --release-train preview `
       --auto-upgrade $autoUpgrade `
       --configuration-settings $config
   ```

----

The Agents and Tools with Foundry Local extension deployment typically takes about 30 minutes but can take longer depending on your connectivity.

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

## Bring your own language model

After deploying the Agents and Tools with Foundry Local extension, complete the steps in [Configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md).

## Related content

- [Deployment parameter reference and troubleshooting](deploy-reference.md)
- [Configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md)
- [Custom certificate authority in Azure Kubernetes Service (AKS)](/azure/aks/custom-certificate-authority)
- [Configuring the chat solution for Agents and Tools with Foundry Local](build-chat-solution-overview.md)
- [Add data source for the chat solution in Agents and Tools with Foundry Local](add-data-source.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#edge-rag-preview-enabled-by-azure-arc)
