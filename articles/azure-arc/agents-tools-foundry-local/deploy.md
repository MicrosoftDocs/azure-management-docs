---
title: Deploy the Agents and Tools with Foundry Local Extension
description: "Learn how to deploy the Agents and Tools with Foundry Local extension by using either Azure CLI or the Azure portal."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ms.custom:
  - build-2025
# Customer Intent: As a cloud administrator or developer, I want to deploy the Agents and Tools with Foundry Local extension using Azure CLI or the Azure portal so that I can enable advanced language model capabilities on my Azure Kubernetes Service (AKS) Arc cluster to provide an intelligent chat solutions.
---
# Deploy the extension for Agents and Tools with Foundry Local

After you complete the prerequisite steps, follow the steps in this article to deploy the Agents and Tools with Foundry Local extension. If your Azure Local environment uses disconnected operations, see [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md) instead.

To try Agents and Tools with Foundry Local without the need for local hardware, see [Quickstart: Deploy Agents and Tools with Foundry Local](quickstart-edge-rag.md).

## Prerequisites

Before you begin, [complete the deployment prerequisites for Agents and Tools with Foundry Local](complete-prerequisites.md).

## Deploy the extension

Deploy the extension by using either the Azure portal or Azure CLI.

#### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com/), go to the Azure Kubernetes cluster on Azure Local.
1. Select **Settings** > **Extensions** > **+ Add**, and **Agents and Tools with Foundry Local** from the list.
1. Select **Create**.
1. On the **Basics** tab, provide the following information:

   | Field      | Value                                                        |
   |-----------------|--------------------------------------------------------------|
   | Subscription    | Select the subscription that contains your Azure Kubernetes Service (AKS) cluster on Azure Local. |
   | Resource group  | Select the resource group that contains your AKS Arc cluster. |
   | Deployment name | Provide a name for the deployment.                           |
   | Region          | Select the region to deploy Agents and Tools with Foundry Local.                        |
   | Cluster         | Select the cluster that you want to deploy Agents and Tools with Foundry Local to.      |

1. Select **Next**.
1. On the **Components** tab, review the included components and choose a value in the required **Enable indexed sources** dropdown (**Enabled** or **Disabled**; the portal template defaults `indexedSourcesEnabled` to `false`).
1. To deploy local ingestion, collections, and Indexed Sources MCP, set **Enable indexed sources** to **Enabled**.
1. Select **Configure** for **Indexed sources** and choose **CPU (recommended)** or **GPU** processing mode.
1. To use NFS with Kerberos, select **Use Kerberos authentication for NFS** and enter the **Kerberos service principal name (SPN)** in the format `service/host@REALM`, such as `nfs/edgerag-svc@CONTOSO.COM`.
1. Select **Save** to save the indexed-sources configuration.

   SharePoint indexed ingestion is enabled by default when indexed sources are enabled. You create the signing Secret and select it later when you add each SharePoint source. The deployment form doesn't collect SharePoint farm identities or signing credentials.

   The portal doesn't configure `foundryClientId`, `byom.*` settings, or Microsoft 365 Local cache storage. If your deployment needs `foundryClientId` (for Foundry Local), `byom.*` (for classic chat in the developer portal), or Microsoft 365 Local cache claims, set them with Azure CLI during installation or in a later extension update. See [Shared file-cache requirements](prepare-microsoft-365-local.md#shared-file-cache-requirements).

1. Select **Next**.
1. On the **Connection + Access** tab, each section (**Extension address and HTTPS**, **User sign-in with Microsoft Entra ID**, and **Trusted services**) opens when you select **Configure** and is saved when you select **Save**:

   | Section | Field | Value |
   |---|---|---|
   | **Extension address and HTTPS** | **Hostname** | Enter the extension hostname without `https://` or a path, such as `agents.contoso.com`. It must match the redirect URI used during app registration. |
   | **Extension address and HTTPS** | **Kubernetes TLS secret name** | Enter the Secret that holds the extension's HTTPS certificate. |
   | **User sign-in with Microsoft Entra ID** | **Application (client) ID** | Enter the application client ID from the Microsoft Entra app registration. |
   | **User sign-in with Microsoft Entra ID** | **Directory (tenant) ID** | Enter the directory tenant ID from the Microsoft Entra app registration. |
   | **Trusted services** | **Service URL** | Optional. Select **Configure**, add each **Service URL** that models or tools use with the extension's managed identity, and select **Save**. The list maps to `serviceUrls`. Use HTTPS URLs without query strings. Wildcards aren't supported. |

1. Select **Review + create**.
1. Review and validate the parameters you provided.
1. Select **Create** to complete the Agents and Tools with Foundry Local deployment.
1. When the deployment is complete, under **Extensions**, validate that the extension types **microsoft.arc.rag** and **microsoft.extensiondiagnostics** are listed.
1. Complete [Allow the extension to check Azure roles](#allow-the-extension-to-check-azure-roles). The portal doesn't create this role assignment.

#### [Azure CLI](#tab/azure-cli)

1. Use one base deployment script and set optional parameters only for the connection types you enable.

   Set `useFoundryLocal=true` when indexed sources or agents use Foundry Local on this cluster. This setting passes `foundryClientId`, which agents use to trust the Foundry Local certificate and as the default managed identity audience. Otherwise, set `useFoundryLocal=false`. The other deployment-level model settings apply only when indexed sources are enabled.

   If you enable Kerberos, complete [Set up NFS with Kerberos authentication for Agents and Tools with Foundry Local](connect-file-share-kerberos-setup.md) first. SharePoint indexed ingestion is part of indexed-sources deployments; create the source-specific certificate Secret and SharePoint trust before you add a SharePoint source.

1. Set common deployment values:

   ```powershell
   $gpu_enabled = "true"  # Mark it true if you have GPUs available for Agents and Tools with Foundry Local
   $localextname = "edgeragdemo" # Once used do not change
   $autoUpgrade = "false"
   $tenantId = "<App Tenant ID>" # App registrations -> Your app -> Overview on Azure portal
   $appId = "<App ID>" # App registrations -> Your app -> Overview on Azure portal
   $domainName = "arcrag.contoso.com" # App redirect URI and this domain name should be the same
   $tlsSecretName = "" # Optional: Kubernetes TLS Secret name for HTTPS
   $sub = "<Subscription GUID>"
   $rg = "<Resource Group name>"
   $k8scluster = "<Azure Kubernetes Service (AKS) Arc cluster name>"
   $extension = "microsoft.arc.rag" # do not change
   $n = "arc-rag" # do not change
   ```

1. Set values for the language model, indexed sources, and optional connection types:

   ```powershell
   $indexedSourcesEnabled = "true" # Set to false to omit local ingestion, collections, and Indexed Sources MCP

   # Required only when indexedSourcesEnabled=true
   $apiEndpoint = "<Your OpenAI-compatible LLM endpoint URI>"
   $apiModel = "<Model Name>"
   $maxTokensInK = "<Max Tokens In K (e.g. 10, 20 etc.)>"

   # Foundry Local setting
   $useFoundryLocal = "false" # Set to true when indexed sources or agents use Foundry Local on this cluster
   $foundryClientId = "<foundry_app_registration_client_id>" # Required only when useFoundryLocal=true

   # Optional approved HTTPS URLs for managed identity calls (JSON array string, e.g. '["https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/"]')
   $serviceUrls = ""

   # Optional Kerberos settings
   $enableKerberos = "false"
   $kerberosSpn = "nfs/edgerag-svc@CONTOSO.COM"

   # Optional Microsoft 365 Local cache claims. Leave empty if you don't use Microsoft 365 Local.
   $sharePointCacheClaim = "" # Existing RWX PVC for SharePoint, such as m365-sharepoint-cache
   $exchangeCacheClaim = ""   # Existing RWX PVC for Exchange, such as m365-exchange-cache

   # SharePoint indexed ingestion is enabled by default. Set false to opt out explicitly.
   $sharePointIngestionEnabled = "true"
   ```

1. Create the BYOM API key secret only when indexed sources are enabled and the language model source is **Bring your own** (`useFoundryLocal=false`):

   ```powershell
   # Create the BYOM API key secret only for indexed sources that use API-key authentication
   if ($indexedSourcesEnabled -eq "true" -and $useFoundryLocal -eq "false") {
      kubectl create namespace arc-rag --dry-run=client -o yaml | kubectl apply -f -
      kubectl delete secret byom-api-key -n arc-rag --ignore-not-found
      kubectl create secret generic byom-api-key --from-literal=BYOM_API_KEY="<your_api_key>" -n arc-rag
   }
   ```

   The deployment stores the BYOM API key as a Kubernetes secret instead of passing it as a configuration parameter. Create this secret only when indexed sources are enabled and `useFoundryLocal=false`.

   The `byom.*` settings and the `byom-api-key` Secret are used only by classic chat in the developer portal. Classic chat is deprecated, and its APIs accept requests only from the developer portal. Agents don't use this key. They use [model connections](create-manage-agents.md).

1. After you populate the values, build configuration settings and deploy the Azure Arc extension:

   ```powershell
   az account set --subscription $sub
   az provider register --namespace Microsoft.KubernetesConfiguration
   az feature register --namespace Microsoft.KubernetesConfiguration --name extensions

   if ($indexedSourcesEnabled -eq "false") {
      $gpu_enabled = "false"
   }

   $config = @(
      "isManagedIdentityRequired=true",
      "gpu_enabled=$gpu_enabled",
      "AgentOperationTimeoutInMinutes=30",
      "auth.tenantId=$tenantId",
      "auth.clientId=$appId",
      "networking.mode=envoy",
      "networking.domainName=$domainName",
      "indexedSourcesEnabled=$indexedSourcesEnabled",
      "sharepoint.sharepointIngestionEnabled=$sharePointIngestionEnabled"
   )

   if ($indexedSourcesEnabled -eq "false" -or $gpu_enabled -eq "false") {
      $config += "skipGPUValidations=true"
   }

   if ($tlsSecretName) {
      $config += "networking.tlsSecretName=$tlsSecretName"
   }

   if ($serviceUrls) {
      $config += "serviceUrls=$serviceUrls"
   }

   if ($sharePointCacheClaim) {
      $config += "sharepointMcp.cache.claimName=$sharePointCacheClaim"
   }

   if ($exchangeCacheClaim) {
      $config += "exchangeMcp.cache.claimName=$exchangeCacheClaim"
   }

   if ($indexedSourcesEnabled -eq "true") {
      $config += @(
         "byom.enabled=true",
         "byom.apiEndpoint=$apiEndpoint",
         "byom.apiModel=$apiModel",
         "byom.maxTokensInK=$maxTokensInK"
      )
   }

   if ($useFoundryLocal -eq "true") {
      $config += @(
         "foundryClientId=$foundryClientId"
      )
   }

   if ($indexedSourcesEnabled -eq "true" -and $enableKerberos -eq "true") {
      $config += @(
         "kerberos.enabled=true",
         "kerberos.spn=$kerberosSpn"
      )
   }

   az k8s-extension create `
      --cluster-type connectedClusters `
      --cluster-name $k8scluster `
      --resource-group $rg `
      --name $localextname `
      --extension-type $extension `
      --release-namespace $n `
      --release-train preview `
      --auto-upgrade-minor-version $autoUpgrade `
      --configuration-settings $config
   ```

----

The Agents and Tools with Foundry Local extension deployment typically takes about 30 minutes but can take longer depending on your connectivity.

## Allow the extension to check Azure roles

Complete this step after either a portal or CLI deployment. This step applies to all model types, including models that use an API key.

The extension checks callers' Azure role assignments through Azure Resource Manager (ARM). Its system-assigned managed identity needs **Reader** at the extension resource scope to read role definitions, role assignments, and deny assignments. An administrator with role-assignment permission needs to run the following commands in Bash:

```azurecli
EXTENSION_ID=$(az k8s-extension show \
  --resource-group "<resource-group>" \
  --cluster-name "<cluster-name>" \
  --cluster-type connectedClusters \
  --name "<extension-name>" \
  --query id -o tsv)

EXTENSION_PRINCIPAL_ID=$(az k8s-extension show \
  --resource-group "<resource-group>" \
  --cluster-name "<cluster-name>" \
  --cluster-type connectedClusters \
  --name "<extension-name>" \
  --query identity.principalId -o tsv)

az role assignment create \
  --assignee-object-id "$EXTENSION_PRINCIPAL_ID" \
  --assignee-principal-type ServicePrincipal \
  --role "Reader" \
  --scope "$EXTENSION_ID"
```

> [!IMPORTANT]
> Without this assignment or an inherited Reader assignment, the Azure role check is unavailable. Responses and Conversations return `503` with `Authorization service unavailable`. Other APIs work only if their app-role fallback allows the caller.

Next, [assign Azure roles to callers](prepare-authentication.md#assign-azure-roles). For a connected deployment, chat users need **Reader** to list agents and **Cognitive Services OpenAI User** to run them. Assign **Cognitive Services Contributor** to people who manage resources.

Allow time for Azure role propagation. The service caches allowed decisions for 300 seconds and denied decisions for 30 seconds.

## Verify the deployment

After deployment, verify the components running in the `arc-rag` namespace. The core Agent Service workloads are always deployed. The `indexedSourcesEnabled` setting controls whether the local indexed-source stack is also deployed.

| `indexedSourcesEnabled` | Expected pods |
|---|---|
| `true` | Core Agent Service pods (`agents-management`, `agents-execution`, `kb-execution`, and `knowledge-sources`) plus the local indexed-source stack (`ingestionapi`, `inferencingflow`, `vectordb-api-server`, embedding models, Docling, Indexed Sources MCP, Milvus, Redis, and PostgreSQL). |
| `false` | Core Agent Service pods and PostgreSQL. The local ingestion, collection, and Indexed Sources MCP workloads aren't deployed. |

Run the following command to check:

```bash
kubectl get pods -n arc-rag
```

## Configure post-deployment authentication

First, complete [Allow the extension to check Azure roles](#allow-the-extension-to-check-azure-roles) and [assign caller roles](prepare-authentication.md#assign-azure-roles). Then complete the separate model authentication task that matches your language model source:

- If you installed the Foundry Local on Azure Local Azure Arc extension and deployed the extension by using `foundryClientId` (Foundry Local), complete [Configure Foundry Local inference authentication for Agents and Tools with Foundry Local](configure-foundry-inference-authentication.md).
- If you deployed the extension for BYOM (`useFoundryLocal=false`), complete [Configure BYOM endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md).

## Verify end-to-end connectivity

After you configure authentication, verify that the extension and Foundry Local are ready:

1. Check Foundry model availability (use port forwarding since `endpoint.enabled=false`). Foundry Local enables TLS on port `5000` by default, and requires a Microsoft Entra bearer token when `foundryClientId` is set:

   ```bash
   kubectl port-forward svc/gpt-oss-20b 5000:5000 -n foundry-local-operator
   # In another terminal (include -H "Authorization: Bearer $TOKEN" when Entra authentication is enabled):
   curl -k https://localhost:5000/v1/models
   ```

1. Test a chat completion by using port forwarding:

   ```bash
   curl -k -X POST https://localhost:5000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $TOKEN" \
     -d '{
       "model": "gpt-oss-20b",
       "messages": [
         {"role": "user", "content": "Hello, what is 2+2?"}
       ],
       "temperature": 0.7,
       "max_tokens": 100
     }'
   ```

1. Verify that all pods in the `arc-rag` namespace are ready, and then test an agent through the `/openai/v1/responses` endpoint:

   ```bash
   kubectl wait --for=condition=Ready pod --all -n arc-rag --timeout=300s
   ```

   To create a model connection and send a test request to `https://<cluster-domain>/openai/v1/responses`, see [Quickstart: Create an agent and chat with it](quickstart-create-agent.md).

## Related content

- [Deployment parameter reference and troubleshooting](deploy-reference.md)
- [Configure endpoint authentication for Agents and Tools with Foundry Local](configure-endpoint-authentication.md)
- [Custom certificate authority in Azure Kubernetes Service (AKS)](/azure/aks/custom-certificate-authority)
- [Agentic Retrieval overview](knowledge-layer-overview.md)
- [Add data source for the chat solution in Agents and Tools with Foundry Local](add-data-source.md)
- [Compare Azure Government and global Azure](/azure/azure-government/compare-azure-government-global-azure#agents-and-tools-with-foundry-local)
