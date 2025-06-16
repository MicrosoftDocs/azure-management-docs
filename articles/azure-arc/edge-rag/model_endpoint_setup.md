---
title: Create Endpoint For Your Model | Edge RAG Enabled By Azure Arc
description: Learn how to set up an endpoint for the model you plan to use with Edge RAG by using Azure AI Foundry, KAITO, Foundry Local, or Ollama.
author: cwatson-cat
ms.topic: how-to
ms.date: 06/16/2025
ms.author: cwatson
ms.reviewer: cwatson
ai-usage: ai-assisted
---

# Create an endpoint to use with Edge RAG enabled by Azure Arc

If you plan to use your own language model instead of one of the models provided by Microsoft, you must set up an endpoint to use with Edge RAG. Choose one of the following methods included in this article to create your endpoint. 

After you create your endpoint, use the endpoint when you [deploy the extension for Edge RAG](deploy.md) and choose to add your own language model.

## Azure AI Foundry

To use your own model with Edge RAG, you can deploy a language model and create an endpoint by using Azure AI Foundry.

1. Go to [Azure AI Foundry](https://ai.azure.com/build/overview?wsid=/subscriptions/169db0a5-d678-473b-9020-88d11cc95c49/resourceGroups/edge-rag/providers/Microsoft.MachineLearningServices/workspaces/edgeragprojeastus2&tid=72f988bf-86f1-41af-91ab-2d7cd011db47) and sign in with your Azure account.

1. Create a new Azure AI Foundry resource or go to an existing resource.

1. On the Azure AI Foundry resource, select **Models + endpoints**.

1. Select **Deploy model** to create a model deployment.

    | Field            | Description   |
    |------------------|--------------|
    | Model            | Choose a chat completion model from the list like `gpt-4o`.  |
    | Deployment name  | Choose deployment name. The default is model name.                   |
    | Deployment type  | Select deployment type. The default is `Global Standard`.      |

1. Select **Deploy**.

1. Wait for the deployment to complete and the **State** is **Succeeded**.

1. Get the endpoint and API Key by selecting on the deployed model. For example, the endpoint looks like the following URL.

   `https://<Azure AI Foundry Resource Name>.openai.azure.com/openai/deployments/<Model Deployment Name>/chat/completions?api-version=<API Version>`


## KAITO

To deploy an AI model by using Kubernetes AI Toolchain Operator (KAITO) on Azure Kubernetes, see [Deploy an AI model on AKS enabled by Azure Arc with the Kubernetes AI toolchain operator](/azure/aks/aksarc/deploy-ai-model?tabs=azurecli).

## Foundry Local

To deploy an AI model using Foundry Local, see [GitHub - microsoft/Foundry-Local](https://github.com/microsoft/Foundry-Local).

## Ollama

You can set up Ollama as a language model endpoint on your Kubernetes cluster. Use either CPU or GPU.

1. If you're using the Ollama with GPU, you must set the following two things on the GPU node. Replace `moc-gpunode` with the name of your GPU node.

   `kubectl taint nodes <moc-gpunode> ollamasku=ollamagpu:NoSchedule –overwrite`

   `kubectl label node <moc-gpunode> hardware=ollamagpu`

1. Create a yaml file by using one of the following snippets depending on whether you're using CPU or GPU for your model.

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

1. Deploy Ollama in the default namespace by using the following yaml snippet. This snippet creates an Ollama deployment and service in default namespace.  

   `kubectl apply -f ollama-deploy.yaml`

1. Prepull a model by using one of the following commands. Get the latest supported models here: [Ollama Search](https://ollama.com/search).

   ```bash
   kubectl exec -n default -it deploy/ollama-deploy -- bash -c "ollama pull <model_name>"
   ```

   Or use kubernetes to connect to the pod and execute the following command inside the ollama pod:

   ```bash
   ollama pull <model_name>
   ```

1. Follow the steps in [Deploy the extension for Edge RAG Preview enabled by Azure Arc](deploy.md) and choose to add your own language model.
1. Use the following endpoint value as you configure the Edge RAG extension deployment:

    `http://ollama-llm.default.svc.cluster.local:11434/v1/chat/completions`

1. *[Optional]* To verify if model can be accessed from another namespace, run the following curl command from inference flow pod in the arc-rag namespace.

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

## Related content

- [Complete deployment prerequisites for Edge RAG Preview, enabled by Azure Arc](complete-prerequisites.md)
- [Deploy the extension for Edge RAG Preview enabled by Azure Arc](deploy.md)
- [Configure "BYOM" endpoint authentication for Edge RAG Preview enabled by Azure Arc](configure-endpoint-authentication.md)