---
title: Configuring the Chat Solution for Edge RAG
description: "Learn about configuring the chat solution in Edge RAG, including ingesting data, prompt engineering, and integrating an endpoint."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 06/05/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a developer or IT professional, I want to understand how to configure data ingestion, optimize chunk and model settings, and integrate the chat endpoint in Edge RAG so that I can enable effective, secure, and contextually relevant chat experiences for end users in my organization.
ms.custom:
  - build-2025
---

# Configuring the chat solution for Edge RAG Preview enabled by Azure Arc

This article provides a high-level overview of the key configuration concepts for the Edge RAG chat solution. Use this guidance to help plan your approach before configuring your chat solution.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Configuring the solution in the developer portal

As part of the Edge RAG solution, a local developer portal is deployed on the Azure Kubernetes Service (AKS) cluster. Developers can access this portal to do the following tasks:

- **Data ingestion**: Provide the on-premises data source and customize settings of the RAG pipeline.
- **Data query**: Provide a custom system prompt, modify model parameters, and evaluate the efficacy of the chat solution by using the chat playground.

Access the portal via the redirect URI (for example, [https://](#)arcrag.contoso.com) that was provided at the time of extension deployment (or the redirect URI provided during app registration.

To authenticate and authorize your access to the portal, make sure you have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Data ingestion

Data ingestion refers to a set of user activities around providing on-premises data and configuring the related settings to make the data searchable. So, when a query comes in, the right context can be retrieved and provided as context to the language model.

### Planning data ingestion

Before you start configuring your chat solution, complete the following steps:

- **Prepare the data**. Review [supported data sources](requirements.md#supported-data-sources). Make sure all your private data is in a network file system (NFS) share that's' accessible from Edge RAG. For data ingestion, you need the NFS share path, NFS user ID, and NFS group ID.

  Make sure that the files aren't password protected or otherwise encrypted for the Edge RAG application to be able to access the data.

- **Choose the right settings for data ingestion**. Before you add a data source in Edge RAG, we recommend you choose the appropriate chunk settings and sync frequency.

### Chunk settings

Before you add a data source in Edge RAG, choose the appropriate chunk size, chunk overlap, and sync frequency. Here's some high-level guidance to select the right chunk settings for your data, as provided by Azure:

- **Chunk size**: Define a fixed size that's sufficient for semantically meaningful paragraphs (for example, 200 words) and allows for some overlap (for example, 10-15% of the content) can produce good chunks as input for embedding vector generators.

  | **Processor** | **Recommended Chunk Size** | **Max Supported Size** |
  |---|---|---|
  | **GPU** | 2000 | 4000 |
  | **CPU-only** | 2000 | 2000 |

- **Chunk overlap**: When you chunk data, overlapping a small amount of text between chunks can help preserve context. We recommend starting with an overlap of approximately 10%. For example, given a fixed chunk size of 256 tokens, you would begin testing with an overlap of 25 tokens. The actual amount of overlap varies depending on the type of data and the specific use case, but we find that 10-15% works for many scenarios.

  |**Processor** | **Recommended Chunk Overlap** | **Max Supported Overlap** |
  | ---|---|---|
  | **GPU** | 200 | 1000 |
  | **CPU-only** | 200 | 200 |

When it comes to chunking data, think about these factors:

- **Shape and density of your documents**: If you need intact text or passages, larger chunks and variable chunking that preserves sentence structure can produce better results.

- **User queries**: Larger chunks and overlapping strategies help preserve context and semantic richness for queries that target specific information.

- **Large Language Models (LLM) have performance guidelines for chunk size**. you need to set a chunk size that works best for all of the models you're using. For instance, if you use models for summarization and embeddings, choose an optimal chunk size that works for both.

### Data ingestion by using REST APIs

You can also perform data ingestion programmatically by using the  REST APIs.

- Data ingestion can take a long time depending on the size of the data, the compute resources available to the embedding model, and other factors.
- Create as many data ingestions as you'd like. However, all the data is vectorized and stored in a single index.

## Data query

In the context of Edge RAG, setting up the data query refers to a set of user activities that include:
- Providing a system prompt. 
- Configuring the model settings to customize the solution to specific user requirements.
- Evaluating the solution to ensure requirements are being met.

### Choosing the right prompt and model parameters

A critical part of prompt engineering is providing the right system prompt and model parameters according to your data and use case. To choose the right system prompt, see [AI Services - safety system messages](/azure/ai-services/openai/concepts/system-message) for high-level guidance.

To choose the model parameters, here's directional guidance:

| **Name** | **Description** |
|---|---|
| **Temperature** | Controls randomness. Lowering the temperature means that the model produces more repetitive and deterministic responses. Increasing the temperature results in more unexpected or creative responses. Try adjusting temperature or Top P but not both. |
| **Top-N** | Refers to number of most relevant chunks given to the language model. Choosing *N* depends on the application's needs:<br> <br>- For broad coverage, use a larger *N*. For example, 10-20. <br>- For precision-critical tasks, use a smaller *N*. For example, 3-5.<br><br> Experiment with *N* values to balance retrieval accuracy and downstream model performance. Bigger the *value of N*, the longer the inference time. |
| **Top-P** | Similar to temperature, this controls randomness but uses a different method. Lowering Top P narrows the model's token selection to likelier tokens. Increasing Top P lets the model choose from tokens with both high and low likelihood. Try adjusting temperature or Top P but not both. |
| **Past Messages Included** | Determines how many messages from the conversation history (previous user and assistant messages) are included in the current inference request. |
| **Text strictness** | Controls how strictly the RAG system filters and ranks retrieved text documents before passing them to the generation model.<br><br>- **Low strictness**: More documents are considered relevant, even if they're only loosely related to the query.<br>- **High strictness**: Only documents that closely match the query are used. |
| **Image strictness** | Similar to text strictness but applies to retrieved image data if the RAG system supports multimodal retrieval.<br><br>- **Low strictness**: The system might retrieve a broader range of images.<br>- **High strictness**: Only images that are very closely aligned with the query are retrieved. |

### Data querying by using REST APIs

In addition to the developer portal, you can use the REST APIs to configure the chat solution such as providing the system message and model parameters.

### Consuming the chat endpoint

After you set up data ingestion and you, as the prompt engineer, are satisfied with the chat solution, you can integrate the chat endpoint in downstream line-of-business applications. Alternately, end users can use the chat application provided out-of-the-box to get started quickly. For more information, see [Test the chat solution for Edge RAG](test-end-user-app.md).

If you want to integrate the chat endpoint in one of your line-of-business applications, use the  REST APIs.

## Related content

- [Supported data sources](requirements.md#supported-data-sources)
- [Add data source for the chat solution in Edge RAG](add-data-source.md)
- [Set up the data query for Edge RAG chat solution](set-up-data-query.md)