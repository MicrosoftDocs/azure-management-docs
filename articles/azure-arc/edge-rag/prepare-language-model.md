---
title: Prepare Your Language Model for Agents and Tools with Foundry Local
description: "Learn how to choose a language model and understand endpoint requirements for your Agents and Tools with Foundry Local deployment using the Bring Your Own Model (BYOM) approach."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/04/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to choose a language model for use with Agents and Tools with Foundry Local so that I can deploy and manage an AI chat solution for my edge environment.

---

# Prepare your language model for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local requires an external language model (Bring Your Own Model). This article helps you choose the right model and understand the endpoint options for your deployment.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Select a language model

Agents and Tools with Foundry Local doesn't include any language models. You must provide your own LLM endpoint (BYOM) that exposes an OpenAI-compatible chat completions API. Both the Agentic Layer (for agent runs) and the Knowledge Layer (for RAG inference) use this endpoint.

Work with your application development team to choose the right model for your use case. For the best results with deep search, use a model like GPT-4o, GPT-4.1-mini, or later.

To choose the right model for your use case, refer to these resources from Microsoft:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Microsoft Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.

## Set up your endpoint

After you choose a model, you need an OpenAI-compatible chat completions endpoint. Several methods are available, including Foundry Local on Azure Local (recommended for production) and other options like Microsoft Foundry, KAITO, and Ollama.

For supported methods and step-by-step setup instructions, see [Create your language model endpoint](prepare-model-endpoint.md).

## Next steps

- [Create your language model endpoint](prepare-model-endpoint.md) — set up your LLM endpoint.
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md) — use the endpoint during deployment.
- [Configure BYOM endpoint authentication](configure-endpoint-authentication.md) — set up authentication after deployment.