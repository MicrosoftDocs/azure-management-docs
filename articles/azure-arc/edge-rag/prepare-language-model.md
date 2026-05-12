---
title: Prepare Your Language Model Endpoint for Agents and Tools with Foundry Local
description: "Learn how to set up an OpenAI-compatible LLM endpoint for your Agents and Tools with Foundry Local deployment using the Bring Your Own Model (BYOM) approach."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/04/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to choose a language model for use with Agents and Tools with Foundry Local so that I can deploy and manage an AI chat solution for my edge environment.

---

# Prepare your language model endpoint for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local requires an external language model (Bring Your Own Model). This article helps you set up an OpenAI-compatible LLM endpoint for your deployment.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Select a language model

Agents and Tools with Foundry Local doesn't include any language models. You must provide your own LLM endpoint (BYOM) that exposes an OpenAI-compatible chat completions API. Both the Agentic Layer (for agent runs) and the Knowledge Layer (for RAG inference) use this endpoint.

Work with your application development team to choose the right model for your use case. For the best results with deep search, use a model like GPT-4o, GPT-4.1-mini, or later.

To choose the right model for your use case, refer to these resources from Microsoft:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.

## Set up your BYOM endpoint

Agents and Tools with Foundry Local works with any language model that exposes an OpenAI-compatible chat completions API endpoint. You can deploy your model locally by using one of the following methods, or use a cloud-hosted endpoint if your environment allows network access.

### Recommended methods

Start with one of these Microsoft-supported options when you want the most reliable setup experience.

| Method | Description | Best for |
|---|---|---|
| **Foundry Local on Azure Local** | Deploy models on your Arc-connected cluster by using Azure AI Foundry. | Production deployments with Azure-managed models. |
| **Microsoft Foundry** | Run models locally via Foundry Local. | Local development and evaluation. |

### Other methods

Use one of these options when your environment needs a different hosting model or a custom OpenAI-compatible endpoint.

| Method | Description | Best for |
|---|---|---|
| **KAITO** | Kubernetes AI Toolchain Operator for model hosting on AKS. | On-premises model hosting with GPU support. |
| **Azure OpenAI** | Cloud-hosted models via Azure OpenAI Service. | When cloud connectivity is available and acceptable. |
| **Ollama** | Lightweight model server running on your cluster. | Development, testing, and CPU-only scenarios. |
| **Any OpenAI-compatible endpoint** | Any service exposing `/v1/chat/completions`. | Custom or third-party model servers. |

For step-by-step setup instructions for each method, see [Create a BYOM endpoint](prepare-model-endpoint.md).

## Next steps

- [Create a BYOM endpoint](prepare-model-endpoint.md) — set up your LLM endpoint.
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md) — use the endpoint during deployment.
- [Configure BYOM endpoint authentication](configure-endpoint-authentication.md) — set up authentication after deployment.