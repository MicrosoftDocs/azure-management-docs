---
title: Prepare Your Language Model Endpoint for Agentic RAG
description: "Learn how to set up an OpenAI-compatible LLM endpoint for your Agentic RAG deployment using the Bring Your Own Model (BYOM) approach."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 04/30/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to choose a language model for use with Agentic RAG so that I can deploy and manage an AI chat solution for my edge environment.

---

# Prepare your language model endpoint for Agentic RAG

Agentic RAG requires an external language model (Bring Your Own Model). This article helps you set up an OpenAI-compatible LLM endpoint for your deployment.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Select a language model

Agentic RAG does not bundle any language models. You must provide your own LLM endpoint (BYOM) that exposes an OpenAI-compatible chat completions API. This endpoint is used by both the Agentic Layer (for agent runs) and the Knowledge Layer (for RAG inference).

Work with your application development team to choose the right model for your use case. For the best results with deep search, use a model like GPT-4o, GPT-4.1-mini, or later.

You can refer to some of these resources from Microsoft to choose the right model for your use case:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.

## Set up your BYOM endpoint

Agentic RAG works with any language model that exposes an OpenAI-compatible chat completions API endpoint. You can deploy your model locally using one of the following methods, or use a cloud-hosted endpoint if your environment allows network access:

| Method | Description | Best for |
|---|---|---|
| **FoundryOnArc** | Deploy models on your Arc-connected cluster using Azure AI Foundry. | Production deployments with Azure-managed models. |
| **KAITO** | Kubernetes AI Toolchain Operator for model hosting on AKS. | On-premises model hosting with GPU support. |
| **Azure OpenAI** | Cloud-hosted models via Azure OpenAI Service. | When cloud connectivity is available and acceptable. |
| **Ollama** | Lightweight model server running on your cluster. | Development, testing, and CPU-only scenarios. |
| **Foundry Local** | Run models locally via Microsoft Foundry Local. | Local development and evaluation. |
| **Any OpenAI-compatible endpoint** | Any service exposing `/v1/chat/completions`. | Custom or third-party model servers. |

For step-by-step setup instructions for each method, see [Create a BYOM endpoint](prepare-model-endpoint.md).

## Next steps

- [Create a BYOM endpoint](prepare-model-endpoint.md) — set up your LLM endpoint.
- [Deploy the Agentic RAG extension](deploy.md) — use the endpoint during deployment.
- [Configure BYOM endpoint authentication](configure-endpoint-authentication.md) — set up authentication after deployment.