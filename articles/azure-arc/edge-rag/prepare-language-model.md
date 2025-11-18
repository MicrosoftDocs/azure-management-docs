---
title: Choose the Right Language Model for Edge RAG Preview Enabled by Azure Arc
description: "Learn how to choose the right language model for Edge RAG deployment, including Microsoft and custom options, to optimize your AI solution."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 06/20/2025
ms.subservice: edge-rag
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to choose a language model for use with Edge RAG so that I can deploy and manage an AI chat solution for my edge environment.

---

# Choose the right language model for Edge RAG Preview enabled by Azure Arc

Review available model options and understand model requirements to choose the right language model for your Edge RAG deployment. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Select a language model

Decide which language model your organization wants to deploy. You can use your own language model or use one of the Edge RAG-provided language models.  

After Edge RAG extension is deployed, you can't change the language model. Therefore, work with your application development team to decide which is the right model for your organization's use case.

You can refer to some of these resources from Microsoft to choose the right model for your use case:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Azure AI Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.


## Edge RAG-provided language models

If you don't have your own language model to use with Edge RAG, select one of the following provided language models when you deploy the Edge RAG extension:

- [Microsoft Phi 3.5 Mini](https://huggingface.co/microsoft/Phi-3.5-mini-instruct)
- [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2)

## Bring your own language model

Edge RAG works with small language models (SLM) or large language models (LLM) that expose endpoints that support the OpenAI inference API. Set up these models locally using Kubernetes AI toolchain operator (KAITO) or similar mechanisms. Edge RAG can also work with OpenAI models in Azure that need API Key-based authentication.

When you bring your own language model (BYOM), you can enable advanced search options like deep search. For the best results with deep search, use a language model like OpenAI [GPT-4o](https://github.com/marketplace/models/azure-openai/gpt-4o), [GPT-4.1-mini](https://github.com/marketplace/models/azure-openai/gpt-4-1-mini) or a later version. These models provide the advanced reasoning and quality needed for deep search scenarios.

If you plan to use your own language model with Edge RAG, you must complete the steps in the following articles:

- Before you deploy Edge RAG, [create an endpoint to use for Edge RAG deployment](prepare-model-endpoint.md).
- After you deploy the Edge RAG extension, [configure "BYOM" endpoint authentication for Edge RAG](configure-endpoint-authentication.md).


## Next step

If you choose to:
- Use an Edge RAG-provided language model, see [Verify file share access for Edge RAG](prepare-file-server.md).
- Bring use your own language model, see [Create an endpoint to use for Edge RAG](prepare-model-endpoint.md).