---
title: Create Your Language Model Endpoint for Agents and Tools with Foundry Local
description: "Learn how to set up an OpenAI API-compatible endpoint for your language model to use with Agents and Tools with Foundry Local by using Foundry Local on Azure Local or Microsoft Foundry."
author: cwatson-cat
ms.topic: how-to
ms.date: 05/12/2026
ms.author: cwatson
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to create an OpenAI API-compatible endpoint for my own language model so that I can use it with an Agents and Tools with Foundry Local deployment.
---

# Create your language model endpoint for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local requires you to provide your own language model endpoint (BYOM). Set up an OpenAI API-compatible endpoint by using one of the following methods.

All search types (hybrid, vector, text, and hybrid multimodal) are available with your BYOM endpoint.

After you create your endpoint, use it when you [deploy the Agents and Tools with Foundry Local extension](deploy.md). The endpoint URL, model name, and max tokens are required deployment parameters.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Choose a setup method

Choose a method based on your environment, connectivity, and production requirements.

| Method | Description | Best for |
|---|---|---|
| **[Foundry Local on Azure Local](#foundry-local)** | Deploy models on your Arc-connected cluster by using the Foundry Local extension. | Production deployments with Azure-managed models. |
| **[Microsoft Foundry](#microsoft-foundry)** | Deploy cloud-hosted models through the Foundry portal. | Cloud-connected deployments with managed models. |

## Foundry Local

Deploy an AI model on your Arc-connected Kubernetes cluster by using the Foundry Local extension. Foundry Local is currently a CLI-based experience.

If you use Foundry Local as your model endpoint, you must install the **Foundry Local extension** on your cluster *before* you install the Agents and Tools with Foundry Local extension. The model endpoint URL from Foundry Local is a required parameter during Agents and Tools deployment.

For setup instructions, see [What is Foundry Local on Azure Local?](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local) and [Foundry Local on GitHub](https://github.com/microsoft/Foundry-Local).

This section shows how to deploy the recommended model (**gpt-oss-20b**) and configure its endpoint for use with Agents and Tools.

### Prerequisites

Before you start, confirm that your cluster, tools, and access settings meet the minimum requirements for Foundry Local.

- Preview deployment access for Foundry Local on Azure Local
- Azure Arc-enabled Kubernetes cluster (Kubernetes 1.29 or later)
- `kubectl` configured for your cluster
- An app registration for authentication (Microsoft Entra ID)
- GPU nodes with sufficient memory for large language models (for example, 40 GB+ VRAM or multi-GPU setups recommended for gpt-oss-20b)

---

### Step 1 — Install required extensions

Install the required Kubernetes extensions so your cluster can host and run Foundry Local model workloads.

1. Install cert-manager and trust-manager:

   ```azurecli
   az k8s-extension create \
     --cluster-name <your_arc_cluster_name> \
     --name "azure-cert-manager" \
     --resource-group <resource_group> \
     --cluster-type connectedClusters \
     --extension-type Microsoft.CertManagement \
     --scope cluster \
     --release-train stable
   ```

1. Install the Foundry inference operator:

   ```azurecli
   az k8s-extension create \
     --resource-group <resource_group> \
     --cluster-name <cluster_name> \
     --name "inference-operator" \
     --extension-type Microsoft.Foundry \
     --scope cluster \
     --release-namespace "foundry-local-operator" \
     --cluster-type connectedClusters \
     --auto-upgrade-minor-version true \
     --release-train stable \
     --config entraAuth.tenantId="<tenant_id>" \
     --config entraAuth.clientId="<client_id>"
   ```

1. Verify installation:

   ```bash
   kubectl get pods -n foundry-local-operator
   ```

### Step 2 — Deploy the recommended model (gpt-oss-20b)

Deploy the recommended gpt-oss-20b model to create a local inference endpoint for your BYOM configuration.

1. Create a ModelDeployment resource:

   ```yaml
   apiVersion: foundrylocal.azure.com/v1
   kind: ModelDeployment
   metadata:
     name: gpt-oss-20b
     namespace: foundry-local-operator
   spec:
     model:
       catalog:
         name: gpt-oss-20b
         version: "latest"
     workloadType: generative
     compute: gpu
     runtime: vllm
     replicas: 1
   ```

1. Apply the deployment:

   ```bash
   kubectl apply -f model-deployment.yaml
   ```

1. Verify it's running:

   ```bash
   kubectl get modeldeployment gpt-oss-20b -n foundry-local-operator
   ```

   Wait until the status is **Running**.

### Step 3 — Verify the model endpoint

Test the deployed endpoint to confirm that it accepts chat completion requests and returns a valid response.

1. Port-forward the model service:

   ```bash
   kubectl port-forward svc/gpt-oss-20b -n foundry-local-operator 5000:5000
   ```

1. Retrieve the API key:

   ```bash
   kubectl get secret gpt-oss-20b-api-keys -n foundry-local-operator \
     -o jsonpath="{.data.primary-key}" | base64 -d
   ```

1. Test the endpoint:

   ```bash
   curl -k -X POST https://localhost:5000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "api-key: <your-api-key>" \
     -d '{
       "model": "gpt-oss-20b",
       "messages": [
         {"role": "user", "content": "Hello, what can you do?"}
       ],
       "max_tokens": 256
     }'
   ```

   You should receive a JSON response with a `choices` array.

### Step 4 — Configure Agents and Tools with Foundry Local

Use the model endpoint as your BYOM configuration.

```yaml
byom:
  enabled: true
  apiEndpoint: "http://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1"
  apiModel: "gpt-oss-20b"
  maxTokensInK: "16"
```

Store the API key in a Kubernetes secret (for example, `byom-api-key`) in the namespace used by Agents and Tools with Foundry Local, following your deployment requirements.

After configuration, Agents and Tools with Foundry Local use the local gpt-oss-20b deployment for all language model interactions.

## Microsoft Foundry

To use your own model with Agents and Tools with Foundry Local, deploy a language model and create an endpoint by using Foundry.

1. Go to [Foundry](https://ai.azure.com/build/overview?wsid=/subscriptions/169db0a5-d678-473b-9020-88d11cc95c49/resourceGroups/edge-rag/providers/Microsoft.MachineLearningServices/workspaces/edgeragprojeastus2&tid=72f988bf-86f1-41af-91ab-2d7cd011db47) and sign in with your Azure account.

1. Create a new Foundry resource or go to an existing resource.

1. On the Foundry resource, select **Models + endpoints**.

1. Select **Deploy model** > **Deploy base model**.
1. Choose a chat completion model from the list like `gpt-4o`.
1. Select **Confirm**.
1. Edit the following fields as appropriate for your scenario:

    | Field            | Description   |
    |------------------|--------------|
    | Deployment name  | Choose a deployment name. The default is the name of the model you selected.                   |
    | Deployment type  | Select a deployment type. The default is **Global Standard**.      |

1. Select **Deploy to selected resource**.

1. Wait for the deployment to complete and the **State** is **Succeeded**.

1. Get the endpoint and API key by selecting the deployed model. For example, the endpoint looks like the following URL.

   `https://<Foundry Resource Name>.openai.azure.com/openai/deployments/<Model Deployment Name>/chat/completions?api-version=<API Version>`

For more information, see the following articles:

- [Deployment types for Foundry Models](/azure/ai-foundry/foundry-models/concepts/deployment-types)
- [Quickstart: Create your first Foundry resource](/azure/ai-services/multi-service-resource?context=%2Fazure%2Fai-foundry%2Fcontext%2Fcontext&pivots=azportal)

## Validate your endpoint

Before deploying Agents and Tools with Foundry Local, verify your endpoint works by sending a test request:

```bash
curl -X POST <your-endpoint-url> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer <your-api-key-if-needed>" \
  -d '{
    "model": "<your-model-name>",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```

You should receive a JSON response with a `choices` array containing the model's answer. If this works, your endpoint is ready for Agents and Tools with Foundry Local.

## Next step

> [!div class="nextstepaction"]
> [Verify file share access](prepare-file-server.md)