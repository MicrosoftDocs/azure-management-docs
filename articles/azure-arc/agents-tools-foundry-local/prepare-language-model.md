---
title: Choose Your Language Model for Agents and Tools with Foundry Local
description: "Learn how to choose a language model and understand endpoint requirements for your Agents and Tools with Foundry Local deployment."
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/05/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to choose a language model for use with Agents and Tools with Foundry Local so that I can deploy and manage an AI chat solution for my edge environment.

---

# Choose your language model for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local requires a language model for inference. This article helps you choose the right model for your use case and understand the available deployment options.

## Select a language model

Agents and Tools with Foundry Local doesn't include any language models. You must provide your own OpenAI-compatible language model endpoint. Agent Service connects to models through model connections (`PUT /connections/{name}`) that you register after deployment, while the deployment-level `byom.*` chat completions endpoint is used only by the local indexed-source stack and deprecated classic chat in the developer portal when `indexedSourcesEnabled=true`.

Work with your application development team to choose the right model for your use case.

To choose the right model for your use case, refer to these resources from Microsoft:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.

## Available models with Foundry Local

If you use [Foundry Local](prepare-model-endpoint.md#foundry-local) as your model endpoint, you can deploy the following models.

The recommended model for most use cases is **gpt-oss-20b**. For step-by-step deployment instructions, see [Create your language model endpoint](prepare-model-endpoint.md).

**CPU-optimized models** (no GPU required):

| Model | Parameters | Notes |
|---|---|---|
| `phi-3-mini-4k-instruct-generic-cpu:2` | 3.8B | Microsoft Phi-3 Mini |
| `phi-3.5-mini-instruct-generic-cpu:1` | 3.8B | Microsoft Phi-3.5 Mini |
| `qwen2.5-0.5b-instruct-generic-cpu:3` | 0.5B | Small, fast |
| `qwen2.5-1.5b-instruct-generic-cpu:3` | 1.5B | Larger Qwen |
| `llama3.2:1b` | 1B | Meta Llama 3.2 |
| `llama3.2:3b` | 3B | Meta Llama 3.2 |

**GPU-optimized models** (CUDA required):

| Model | Parameters |
|---|---|
| `gpt-oss-20b` | 20B |
| `qwen2.5-1.5b-instruct-cuda-gpu:3` | 1.5B |
| `llama3.1:8b` | 8B |

### Recommended models

| Use case | Recommended model | Runtime |
|---|---|---|
| Agent execution | `gpt-oss-20b` | `vllm` (required for Agent Service tool calling and multi-part message content) |
| Indexed inference | `gpt-oss-20b` | `vllm` or `onnx-genai` |

Agents that call tools need a model and runtime that support OpenAI-compatible tool calling.

## Set up your endpoint

After you choose a model, you need an OpenAI-compatible endpoint. Use Foundry Local on Azure Local because it runs on the same Arc-connected cluster as the extension. You can also use Microsoft Foundry for cloud-hosted models.

For supported methods and step-by-step setup instructions, see [Create your language model endpoint](prepare-model-endpoint.md).

When you create an Agent Service model connection, use the HTTPS API base ending in `/v1/`, not the complete `/chat/completions` request URL.

## Next steps

- [Create your language model endpoint](prepare-model-endpoint.md) — set up your LLM endpoint.
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md) — deploy the extension.
- [Create and manage agents](create-manage-agents.md) — register model connections and create agents after deployment.
- [Configure Foundry Local inference authentication](configure-foundry-inference-authentication.md) — authorize managed identity access to Foundry Local after deployment.
- [Configure BYOM endpoint authentication](configure-endpoint-authentication.md) — set up authentication after deployment.
