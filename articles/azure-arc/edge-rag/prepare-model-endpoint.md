---
title: Create "BYOM" Endpoint for Your Model for Edge RAG Preview Enabled by Azure Arc
description: "Learn how to set up an endpoint for the model you plan to use with Edge RAG by using Azure AI Foundry, KAITO, Foundry Local, or Ollama."
author: cwatson-cat
ms.topic: how-to
ms.date: 10/29/2025
ms.author: cwatson
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to create an OpenAI API-compatible endpoint for my own language model so that I can use it with an Edge RAG deployment.
---

# Create a "BYOM" endpoint to use for Edge RAG Preview enabled by Azure Arc

If you plan to bring your own language model (BYOM) instead of one of the models included in Edge RAG, you must set up an OpenAI API compatible endpoint for your Edge RAG deployment. Choose one of the following methods included in this article to create your endpoint.

By bringing your own model, you can enable advanced search types, like hybrid multimodal and deep search, that aren’t available with Edge RAG-provided models. For deep search, we recommend OpenAI [GPT-4o](https://github.com/marketplace/models/azure-openai/gpt-4o), [GPT-4.1-mini](https://github.com/marketplace/models/azure-openai/gpt-4-1-mini) or a later version.

After you create your endpoint, use the endpoint when you [deploy the extension for Edge RAG](deploy.md) and choose to add your own language model.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Azure AI Foundry

To use your own model with Edge RAG, you can deploy a language model and create an endpoint by using Azure AI Foundry.

1. Go to [Azure AI Foundry](https://ai.azure.com/build/overview?wsid=/subscriptions/169db0a5-d678-473b-9020-88d11cc95c49/resourceGroups/edge-rag/providers/Microsoft.MachineLearningServices/workspaces/edgeragprojeastus2&tid=72f988bf-86f1-41af-91ab-2d7cd011db47) and sign in with your Azure account.

1. Create a new Azure AI Foundry resource or go to an existing resource.

1. On the Azure AI Foundry resource, select **Models + endpoints**.

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

   `https://<Azure AI Foundry Resource Name>.openai.azure.com/openai/deployments/<Model Deployment Name>/chat/completions?api-version=<API Version>`

For more information, see the following articles:

- [Deployment types for Azure AI Foundry Models](/azure/ai-foundry/foundry-models/concepts/deployment-types)
- [Quickstart: Create your first AI Foundry resource](/azure/ai-services/multi-service-resource?context=%2Fazure%2Fai-foundry%2Fcontext%2Fcontext&pivots=azportal)

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
1. Use the following endpoint value as you configure the Edge RAG extension deployment:

    `http://ollama-llm.default.svc.cluster.local:11434/v1/chat/completions`

After you deploy the Edge RAG extension, verify that the model can be accessed from another namespace. Run the following curl command from inference flow pod in the arc-rag namespace.

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

## Next step

> [!div class="nextstepaction"]
> [Verify file share access](prepare-file-server.md)