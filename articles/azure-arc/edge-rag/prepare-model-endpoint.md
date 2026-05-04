---
title: Create Your Language Model Endpoint for Agentic RAG
description: "Learn how to set up an OpenAI API-compatible endpoint for your language model to use with Agentic RAG by using Microsoft Foundry, KAITO, Foundry Local, or Ollama."
author: cwatson-cat
ms.topic: how-to
ms.date: 04/30/2026
ms.author: cwatson
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to create an OpenAI API-compatible endpoint for my own language model so that I can use it with an Agentic RAG deployment.
---

# Create your language model endpoint for Agentic RAG

Agentic RAG requires you to provide your own language model endpoint (BYOM). Set up an OpenAI API-compatible endpoint using one of the methods below.

All search types (hybrid, vector, text, hybrid multimodal, and deep search) are available with your BYOM endpoint.

After you create your endpoint, use it when you [deploy the Agentic RAG extension](deploy.md). The endpoint URL, model name, and max tokens are required deployment parameters.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Microsoft Foundry

To use your own model with Agentic RAG, you can deploy a language model and create an endpoint by using Foundry.

1. Go to [Foundry](https://ai.azure.com/build/overview?wsid=/subscriptions/169db0a5-d678-473b-9020-88d11cc95c49/resourceGroups/edge-rag/providers/Microsoft.MachineLearningServices/workspaces/edgeragprojeastus2&tid=72f988bf-86f1-41af-91ab-2d7cd011db47) and sign in with your Azure account.

1. Create a new Foundry resource or go to an existing resource.

1. On the Foundry resource, select **Models + endpoints**.

1. Select **Deploy model** > **Deploy base model**.
1. Choose a chat completion model from the list like `gpt-4o`.
1. Select **Confirm**.
1. Edit the following fields as appropriate for your scenario:

    | Field            | Description   |
    |------------------|--------------|
    | Deployment name  | Choose deployment name. The default is name of the model you selected.                   |
    | Deployment type  | Select deployment type. The default is **Global Standard**.      |

1. Select **Deploy to selected resource**.

1. Wait for the deployment to complete and the **State** is **Succeeded**.

1. Get the endpoint and API Key by selecting on the deployed model. For example, the endpoint looks like the following URL.

   `https://<Foundry Resource Name>.openai.azure.com/openai/deployments/<Model Deployment Name>/chat/completions?api-version=<API Version>`

For more information, see the following articles:

- [Deployment types for Foundry Models](/azure/ai-foundry/foundry-models/concepts/deployment-types)
- [Quickstart: Create your first Foundry resource](/azure/ai-services/multi-service-resource?context=%2Fazure%2Fai-foundry%2Fcontext%2Fcontext&pivots=azportal)

## KAITO

To deploy an AI model by using Kubernetes AI Toolchain Operator (KAITO) on Azure Kubernetes, see [Deploy an AI model on AKS enabled by Azure Arc with the Kubernetes AI toolchain operator](/azure/aks/aksarc/deploy-ai-model?tabs=azurecli).

## Foundry Local

To deploy an AI model using Foundry Local, see [GitHub - microsoft/Foundry-Local](https://github.com/microsoft/Foundry-Local).

## Ollama

You can set up Ollama as a language model endpoint on your Kubernetes cluster. Use either CPU or GPU.

1. If you're using the Ollama with GPU, you must set the following two things on the GPU node. Replace `moc-gpunode` with the name of your GPU node.

   ```bash
   kubectl taint nodes <moc-gpunode> ollamasku=ollamagpu:NoSchedule –overwrite

   kubectl label node <moc-gpunode> hardware=ollamagpu`
   ```

1. Create a yaml file by using one of the following snippets depending on whether you're using GPU or CPU for your model.

    - GPU yaml:

      ```yaml
      # ollama-deploy.yaml
      
      apiVersion: apps/v1
      kind: Deployment
      metadata:
          name: ollama-deploy
          namespace: default
      spec:
          replicas: 1
          selector:
              matchLabels:
                  app: ollama-deploy
          template:
              metadata:
                  labels:
                      app: ollama-deploy
              spec:
                  affinity:
                      nodeAffinity:
                          requiredDuringSchedulingIgnoredDuringExecution:
                              nodeSelectorTerms:
                              - matchExpressions:
                                  - key: hardware
                                      operator: In
                                      values:
                                      - ollamagpu
                  containers:
                      - name: ollama
                          image: ollama/ollama
                          args: ["serve"]
                          ports:
                              - containerPort: 11434
                          volumeMounts:
                              - name: ollama-data
                                  mountPath: /root/.ollama
                          resources:
                              limits:
                                  nvidia.com/gpu: "1"
                  volumes:
                      - name: ollama-data
                          emptyDir: {}
                  tolerations:
                      - effect: NoSchedule
                          key: ollamasku
                          operator: Equal
                          value: ollamagpu
      
      ---
      apiVersion: v1
      kind: Service
      metadata:
          name: ollama-llm
          namespace: default
      spec:
          selector:
              app: ollama-deploy
          ports:
              - port: 11434
                  targetPort: 11434
                  protocol: TCP
      ```
 
    - CPU yaml:

      ```yaml
      # ollama-deploy.yaml
      
      apiVersion: apps/v1
      kind: Deployment
      metadata:
          name: ollama-deploy
          namespace: default
      spec:
          replicas: 1
          selector:
              matchLabels:
                  app: ollama-deploy
          template:
              metadata:
                  labels:
                      app: ollama-deploy
              spec:
                  containers:
                      - name: ollama
                          image: ollama/ollama
                          args: ["serve"]
                          ports:
                              - containerPort: 11434
                          volumeMounts:
                              - name: ollama-data
                                  mountPath: /root/.ollama
                  volumes:
                      - name: ollama-data
                          emptyDir: {}
      
      ---
      apiVersion: v1
      kind: Service
      metadata:
          name: ollama-llm
          namespace: default
      spec:
          selector:
              app: ollama-deploy
          ports:
              - port: 11434
                  targetPort: 11434
                  protocol: TCP
      ```

1. Deploy Ollama in the default namespace by using the following yaml snippet. This snippet creates an Ollama deployment and service in default namespace.  

   ```bash
   kubectl apply -f ollama-deploy.yaml
   ```

1. Download a model by using one of the following commands. Get the latest supported models here: [Ollama Search](https://ollama.com/search).

   ```bash
   kubectl exec -n default -it deploy/ollama-deploy -- bash -c "ollama pull <model_name>"
   ```

   Or use k9s to connect to the pod and execute the following command inside the ollama pod:

   ```bash
   ollama pull <model_name>
   ```
1. Use the following endpoint value as you configure the Agentic RAG extension deployment:

    `http://ollama-llm.default.svc.cluster.local:11434/v1/chat/completions`

After you deploy the Agentic RAG extension, verify that the model can be accessed from another namespace. Run the following curl command from inference flow pod in the arc-rag namespace.

```bash
curl http://ollama-llm.default.svc.cluster.local:11434/v1/chat/completions \
    -H "Content-Type: application/json" \
    -d '{
        "model": "llama3:8b",
        "messages": [
            { "role": "system", "content": "You are a helpful assistant." },
            { "role": "user", "content": "What is the capital of Japan?" }
        ]
    }'
```

## Validate your endpoint

Before deploying Agentic RAG, verify your endpoint works by sending a test request:

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

You should receive a JSON response with a `choices` array containing the model's answer. If this works, your endpoint is ready for Agentic RAG.

## Next step

> [!div class="nextstepaction"]
> [Verify file share access](prepare-file-server.md)