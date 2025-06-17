---
title: Choose the Right Language Model for Edge RAG Deployment
description: "Decide which language model to deploy for Edge RAG."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/21/2025
ai-usage: ai-assisted
---

# Choose the right language model for Edge RAG Deployment

In this article, choose the right language model for your Edge RAG deployment by reviewing available options and understanding model requirements. This article is part of the deployment prerequisites checklist.

## Select a language model

Decide which language model your organization wants to deploy. You can use your own language model or use the Microsoft provided language models: 

- [Microsoft Phi 3.5 Mini](https://huggingface.co/microsoft/Phi-3.5-mini-instruct) and
- [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-Instruct-v0.2). 

After Edge RAG extension is deployed, you can't change the language model. Therefore, work with your application development team to decide which is the right model for your organization's use case. 

You can refer to some of these resources from Microsoft to choose the right model for your use case:

- Blog: [How to Choose the Right Models for Your Apps | Azure AI](https://techcommunity.microsoft.com/blog/microsoftmechanicsblog/how-to-choose-the-right-models-for-your-apps--azure-ai/4271216)
- Video: [How to Choose the Right Models for Your Apps | Azure AI - YouTube](https://www.youtube.com/watch?app=desktop&v=sx_uGylH8eg&t=53s)
- [Azure AI Foundry](/azure/ai-studio/concepts/model-benchmarks) also provides tooling such as model benchmarks to choose the right model.


## Microsoft provided language model

## Bring your own language model

Edge RAG can work with small language models (SLM) or large language models (LLM) that expose endpoints which support the OpenAI Inference API. Set up these models locally using Kubernetes AI toolchain operator (KAITO)  or similar mechanisms. Edge RAG can also work with OpenAI models in Azure that need API Key-based authentication.

## Next step

> [!div class="nextstepaction"]
> [Verify NFS server is accessible for Edge RAG deployment](prepare-file-server.md)