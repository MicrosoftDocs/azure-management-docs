---
title: Create a language model endpoint
description: "Learn how to create an OpenAI-compatible model endpoint for Agents and Tools with Foundry Local by using Foundry Local or Microsoft Foundry."
ms.topic: how-to
ms.date: 11/05/2026
ms.reviewer: assafakiva
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to create an OpenAI API-compatible endpoint for my own language model so that I can use it with an Agents and Tools with Foundry Local deployment.
---

# Create a language model endpoint

When you enable indexed sources, the extension needs an OpenAI API-compatible language model endpoint for local retrieval-augmented generation. If you set `indexedSourcesEnabled=false`, don't pass an endpoint during deployment. Agents still need a model endpoint, which you register as a model connection after deployment. If that endpoint is Foundry Local on the same cluster, set `foundryClientId` during deployment.

All search types (hybrid, vector, text, and hybrid multimodal) are available with your language model endpoint.

After you create your endpoint, use it when you [deploy the Agents and Tools with Foundry Local extension](deploy.md) with indexed sources enabled. The endpoint URL, model name, and max tokens are then required deployment parameters.

## Choose a setup method

Choose a method based on your environment, connectivity, and production requirements.

| Method | Description | Best for |
|---|---|---|
| **[Foundry Local on Azure Local](#foundry-local)** | Deploy models on your Arc-connected cluster by using the Foundry Local extension. | Production deployments with Azure-managed models. |
| **[Microsoft Foundry](#microsoft-foundry)** | Deploy cloud-hosted models through the Foundry portal. | Cloud-connected deployments with managed models. |

## Foundry Local

Deploy an AI model on your Arc-connected Kubernetes cluster by using the Foundry Local extension. Foundry Local is currently a CLI-based experience.

The following table summarizes the key properties of the Foundry Local extension:

| Property | Value |
|---|---|
| Extension type | `Microsoft.Foundry` |
| Default namespace | `foundry-local-operator` |
| Inference port | `5000` (TLS enabled by default via nginx sidecar) |
| API format | OpenAI-compatible (`/v1/chat/completions`) |

You must install and run Foundry Local on your cluster *before* you install the Agents and Tools with Foundry Local extension. Pass `foundryClientId` during deployment whenever indexed sources or agents call Foundry Local on the cluster. The `byom.*` endpoint parameters (`byom.enabled=true`, `byom.apiEndpoint`, `byom.apiModel`, and `byom.maxTokensInK`) are required during deployment only when `indexedSourcesEnabled=true` for the local indexed-source stack and deprecated classic chat in the developer portal. Agent Service doesn't use `byom.*`; agents use model connections (`PUT /connections/{name}`) that you register after deployment. If you don't set up Foundry Local correctly, model calls fail at runtime with connection errors.

For setup instructions, see [What is Foundry Local on Azure Local?](/azure/azure-sovereign-clouds/private/foundry-local/what-is-foundry-local-on-azure-local) and [Foundry Local on GitHub](https://github.com/microsoft/Foundry-Local).

This section shows how to deploy the recommended model (**gpt-oss-20b**) and configure its endpoint for use with Agents and Tools with Foundry Local.

### Prerequisites

Before you start, confirm that your cluster, tools, and access settings meet the minimum requirements for Foundry Local.

- Preview deployment access for Foundry Local on Azure Local
- Azure Arc-enabled Kubernetes cluster (Kubernetes 1.29 or later)
- `kubectl` configured for your cluster
- An app registration for authentication (Microsoft Entra ID)
- GPU nodes with sufficient memory for large language models (for example, 40 GB+ VRAM or multi-GPU setups recommended for gpt-oss-20b)

### Step 1 - Install required extensions

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
     --release-train stable \
     --config config.enableGatewayAPI=true \
     --config cert-manager.crds.keep=true \
     --config trust-manager.defaultPackage.enabled=false \
     --config trust-manager.secretTargets.enabled=true \
     --config trust-manager.secretTargets.authorizedSecretsAll=true
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

   > [!IMPORTANT]
   > Microsoft Entra ID authentication is required for Foundry Local. You must provide a valid `entraAuth.tenantId` and `entraAuth.clientId` from an [app registration](/entra/identity-platform/quickstart-register-app). The Agents and Tools with Foundry Local extension uses this identity for secure communication with the Foundry Local endpoint.

1. Verify installation:

   ```bash
   kubectl get pods -n foundry-local-operator
   kubectl get crd | grep foundry
   ```

   Expected output: five pods in `Running` or `Completed` status, and four Foundry Local custom resource definitions (CRDs) registered:

   ```output
   inferenceservices.foundrylocal.azure.com
   modeldeployments.foundrylocal.azure.com
   models.foundrylocal.azure.com
   storemodels.foundrylocal.azure.com
   ```

   > [!WARNING]
   > Don't proceed to Step 2 until all four CRDs are present. The `ModelDeployment` CRD installs through a Helm pre-install hook that can fail silently. If `modeldeployments` is missing, extract and apply it manually:
   >
   > ```bash
   > helm get hooks inference-operator -n foundry-local-operator > /tmp/hooks.yaml
   > # Extract the ModelDeployment CRD YAML from the hooks output, save to a file, then:
   > kubectl apply -f /tmp/modeldeployment-crd.yaml
   > ```

### Step 2 - Deploy the recommended model (gpt-oss-20b)

Deploy the recommended gpt-oss-20b model to create a local inference endpoint for your language model configuration.

1. Create a ModelDeployment resource:

   ```yaml
   apiVersion: foundrylocal.azure.com/v1
   kind: ModelDeployment
   metadata:
     name: gpt-oss-20b
     namespace: foundry-local-operator
   spec:
     workloadType: generative
     compute: gpu
     runtime: vllm
     model:
       catalog:
         name: gpt-oss-20b
         version: "latest"
     replicas: 1
     port: 5000
     resources:
       requests: { cpu: "4", memory: "32Gi" }
       limits: { cpu: "8", memory: "64Gi", gpu: 1 }
     nodeSelector:
       # For Azure Local (HaaS) GPU nodes, use:
       nvidia.com/gpu.present: "true"
       # For AKS-managed GPU nodes, use:
       #   kubernetes.azure.com/accelerator: nvidia
       #   agentpool: <your_gpu_node_pool>
     tolerations:
       - { key: "sku", operator: "Equal", value: "gpu", effect: "NoSchedule" }
       - { key: "nvidia.com/gpu", operator: "Exists", effect: "NoSchedule" }
     endpoint:
       enabled: false
     vllm:
       modelCacheStorageGi: 100
       preferences:
         gpu_memory_utilization: 0.92
         max_model_len: 16384
         dtype: "bfloat16"
         max_num_seqs: 128
         max_num_batched_tokens: 4096
         enforce_eager: false
   ```

   Deploy the model with `endpoint.enabled: false` to access it through the internal Kubernetes service DNS instead of an external ingress. Set the `nodeSelector` based on your cluster type:

   - **AKS-managed GPU nodes**: Use `kubernetes.azure.com/accelerator: nvidia` and `agentpool: <your_gpu_node_pool>`.
   - **Azure Local (HaaS) GPU nodes**: Use `nvidia.com/gpu.present: "true"`.

   **KV cache quantization**: If your GPU supports SM 9.0 or later (for example, NVIDIA H100), enable FP8 KV cache quantization by adding `kv_cache_dtype: "fp8_e4m3"` under `vllm.preferences`. Don't use this setting on L40S, A100, or earlier GPUs.

   > [!IMPORTANT]
   > **Runtime selection**: Use the `vllm` runtime for Agent Service requests. Agent execution can send OpenAI-compatible tool calls and message content parts that the current `onnx-genai` request handler doesn't accept. Use `onnx-genai` only for direct model or knowledge-layer calls that your deployed model runtime supports.

1. Apply the deployment.

   ```bash
   kubectl apply -f model-deployment.yaml
   ```

1. Verify it's running:

   ```bash
   kubectl get modeldeployment gpt-oss-20b -n foundry-local-operator
   ```

   Wait until the status is **Running**. GPU model deployment typically takes 10-30 minutes depending on model size and whether the image is cached.

### Step 3 - Verify the model endpoint

Test the deployed endpoint to confirm that it accepts chat completion requests and returns a valid response.

1. Discover the model service endpoint dynamically:

   ```bash
   kubectl get svc <model-name> -n foundry-local-operator -o jsonpath='https://{.metadata.name}.{.metadata.namespace}.svc.cluster.local:{.spec.ports[0].port}'
   ```

   Service names vary by model deployment. Use this command to confirm the correct service name before proceeding.

1. Port-forward the model service:

   ```bash
   kubectl port-forward svc/gpt-oss-20b -n foundry-local-operator 5000:5000
   ```

1. Note the internal Kubernetes service URLs for your configuration:

   - **Agent Service model connections (`PUT /connections/{name}` and `serviceUrls`)** use the `/v1/` base URL:

     ```text
     https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/
     ```

   - **Indexed-source deployment (`byom.apiEndpoint`, required only when `indexedSourcesEnabled=true`)** uses the chat completions URL:

     ```text
     https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions
     ```

   Use these URLs when Foundry Local runs on the same cluster as Agents and Tools with Foundry Local.

   > [!IMPORTANT]
   > The endpoint **must** use `https://`, not `http://`. Foundry Local enables TLS on port 5000 via an nginx sidecar with a self-signed certificate. Using `http://` results in `400 Bad Request: The plain HTTP request was sent to HTTPS port`.
   >
   > The Agents and Tools with Foundry Local extension trusts the Foundry CA certificate automatically when you set `foundryClientId`. The Helm chart mounts the `foundry-local-operator-ca-bundle` ConfigMap and sets `REQUESTS_CA_BUNDLE` / `SSL_CERT_FILE` environment variables. Without `foundryClientId`, HTTPS calls fail with `SSL: CERTIFICATE_VERIFY_FAILED`.

   If you have an external ingress configured, you can also use the external URL:

   ```text
   https://<foundry_ingress_host>/v1/chat/completions
   ```

1. Test the endpoint:

   ```bash
   curl -X POST http://localhost:5000/v1/chat/completions \
     -H "Content-Type: application/json" \
     -d '{
       "model": "gpt-oss-20b",
       "messages": [
         {"role": "user", "content": "Hello, what can you do?"}
       ],
       "max_tokens": 256
     }'
   ```

   Port-forwarding connects directly to the model container, bypassing the TLS and authentication sidecars. This setup is appropriate for local verification only. In production, the Agents and Tools with Foundry Local extension connects to the model via the internal HTTPS service URL with Entra ID authentication handled automatically through the `foundryClientId` configuration.

   You should receive a JSON response with a `choices` array.

### Step 4 - Configure the extension

When you [deploy the Agents and Tools with Foundry Local extension](deploy.md), pass `foundryClientId` whenever indexed sources or agents call Foundry Local on the cluster. Pass the `byom.*` settings only when `indexedSourcesEnabled=true` (the local indexed-source stack and deprecated classic chat in the developer portal use these settings):

```
foundryClientId=<foundry_app_registration_client_id>
byom.enabled=true
byom.apiEndpoint=https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions
byom.apiModel=gpt-oss-20b
byom.maxTokensInK=16
```

Agent Service doesn't use `byom.*`. Instead, agents use model connections (`PUT /connections/{name}`) with the `/v1/` base URL (`https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/`) and `serviceUrls` approval for managed identity. See [Create and manage agents](create-manage-agents.md).

The `foundryClientId` parameter enables Microsoft Entra ID-based authentication between the Agents and Tools with Foundry Local extension and the Foundry Local endpoint. You don't need an API key secret when you use Foundry Local.

After deployment, the Agents and Tools with Foundry Local extension can use the local gpt-oss-20b deployment for language model requests.

For optional operator parameters and namespace configuration, see [Deployment parameter reference](deploy-reference.md#foundry-local-operator-parameters).

## Microsoft Foundry

To use your own model with the extension, deploy a language model and create an endpoint by using Foundry.

1. Go to [Foundry](https://ai.azure.com) and sign in with your Azure account.

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

Before deploying the extension, verify that your endpoint works by sending a test request.

> [!NOTE]
> The authentication header format depends on your provider. Foundry Local managed identity and OpenAI-compatible endpoints pass a bearer token in the `Authorization` header. Microsoft Foundry and Azure OpenAI endpoints accept an API key in the `api-key` header (`api-key: $API_KEY`) or a Microsoft Entra bearer token in the `Authorization` header.

```bash
curl -X POST <your-endpoint-url> \
  -H "Content-Type: application/json" \
  -H "api-key: $API_KEY" \
  -d '{
    "model": "<your-model-name>",
    "messages": [
      {"role": "system", "content": "You are a helpful assistant."},
      {"role": "user", "content": "What is the capital of France?"}
    ]
  }'
```

You should receive a JSON response with a `choices` array containing the model's answer. If this test succeeds, your endpoint is ready for the extension.

## Related content

- [Choose your language model](prepare-language-model.md)
- [Deployment parameter reference](deploy-reference.md)

## Next step

> [!div class="nextstepaction"]
> [Verify file share access](prepare-file-server.md)
