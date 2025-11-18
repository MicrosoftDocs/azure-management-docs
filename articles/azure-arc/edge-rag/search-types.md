---
title: Types of search in Edge RAG
description: Learn about search types, their options, and how they're used in Edge RAG deployments.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 10/29/2025
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As a cloud administrator or developer, I want to understand search types in Edge RAG so I can choose and configure the right search type for my deployment.
---

# Search types in Edge RAG Preview enabled by Azure Arc

A search method or type, like full text, vector, or hybrid, controls how Edge RAG retrieves and ranks results from your indexed data. It shapes the way users interact with your solution and the quality of the answers they get.

You select a search type when you configure your data query settings in the Edge RAG developer portal, after you deploy your solution with either an Edge RAG-provided model or your own model (BYOM).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Why your search type matters

Your choice of search type affects:

- The kinds of questions users can ask.<br>
  For example: Text search works best for simple keyword queries, while vector or hybrid search lets users ask more natural, conversational questions.
- The relevance and accuracy of results.
  For example: Hybrid search can return more relevant answers by combining exact matches with semantic understanding, while text search might miss contextually similar results.
- Access to advanced features like multimodal or deep search. 
  For example: Deep search enables complex reasoning across multiple documents, and hybrid multimodal search lets users find information in both text and images, which are capabilities not available with other search types.

## Available search types

The following definitions describe each search type available in Edge RAG, so you can understand how they work and choose the best option for your scenario.

- **Deep search**: Implements [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0), a method that builds a dynamic graph of information at query time to find the most relevant and connected content. Instead of relying on precomputed summaries or embeddings, LazyGraph RAG incrementally explores and connects data across sources, gathering only the information needed to answer the question. This approach improves answer quality and reduces cost by focusing retrieval on the most useful content for each query.

- **Full text search**: A search method that scans and matches the entire body of text in documents, by using keywords, phrases, or boolean queries to find relevant chunks in the supplied documents.

- **Hybrid search**: Combines both full-text search (keyword-based) and vector search (semantic similarity) to retrieve the most relevant documents. It uses the precision of keyword matching and the depth of semantic understanding for improved retrieval accuracy.

- **Hybrid multimodal search**: Combines full-text and vector search across both text and image data, allowing Edge RAG to retrieve and rank results from multiple data types in a single query. Unlike hybrid search, which works only with text, by using both keyword and semantic similarity, hybrid multimodal search extends this approach to include images. This approach enables richer, more comprehensive answers that draw from both textual, and visual content.

- **Vector search**: A search method that finds relevant documents by comparing the semantic similarity between vector embeddings of the user's query and precomputed embeddings of documents, typically using cosine similarity or other distance metrics in a vector space.

## Compare search types

Edge RAG supports several search types. The following table summarizes each search type, best use cases, and performance considerations. The options you see depend on the language model you use and your scenario:

| Search type               | What it does      | Best use cases       | Performance notes                                                                                  |
|--------------------------|------------------|-----------------|---------------------|
| **Text search**          | Finds exact words or phrases in your documents using keywords, phrases, or boolean queries.                                                                                                                              | Standard question and answer (Q&A), document lookup, compliance scenarios           | Fastest; highly efficient for simple lookups.                                                     |
| **Vector search**        | Uses semantic similarity to find contextually relevant results by comparing vector embeddings.                                                                                                                           | Conversational Q&A, finding related content                   | Fast; slightly more compute than text search.                                                      |
| **Hybrid search**        | Combines text (keyword) and vector (semantic) search for better coverage and accuracy.                                                                                            | Most scenarios; default for general use                       | Fast; combines strengths of text and vector search.                                                |
| **Deep search**          | Implements [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0), dynamically exploring and connecting data at query time for advanced retrieval. | Complex reasoning, summarization, multi-document Q&A          | Slower; higher compute, but higher quality and context. Requires BYOM and advanced language model. |
| **Hybrid multimodal**    | Combines text, vector, and image search to retrieve and rank results from multiple data types in a single query.                                                                  | Scenarios involving both text and images                      | Slower than text/vector; performance depends on data size/type. Requires BYOM.                     |
| **Full text search**     | Scans and matches the entire body of text in documents.            | Legal, compliance, or when exact matches are required         | Fastest for keyword lookups; might miss semantic matches.                                            |

## Search type availability by deployment

The search types available when you configure your chat solution depend on how you deployed Edge RAG, whether you use an Edge RAG-provided language model or bring your own (BYOM). 

| Deployment type                | Available search types                                      |
|-------------------------------|------------------------------------------------------------|
| Edge RAG-provided language model       | Hybrid search, text search, vector search                  |
| Bring your own model (BYOM)    | Hybrid search, text search, vector search, deep search, hybrid multimodal search |

If you plan to bring your own model (BYOM), we recommend OpenAI [GPT-4o](https://github.com/marketplace/models/azure-openai/gpt-4o), [GPT-4.1-mini](https://github.com/marketplace/models/azure-openai/gpt-4-1-mini) or a later version. For more information about BYOM, see: 

- [Choose the right language model for Edge RAG](prepare-language-model.md)
- [Create a "BYOM" endpoint to use for Edge RAG](prepare-model-endpoint.md)

For more information about deploying with a language model, see:

- [Deployment overview for Edge RAG](deploy-overview.md) 
- [Deploy the extension for Edge RAG](deploy.md)


## How to select a search type

You select the search type in the Edge RAG developer portal when you configure your data query settings. The available options depend on your deployment. Some parameters might differ by search type and model.

## Performance considerations

Each search type offers different performance characteristics. Consider these points when choosing the best option for your scenario:

- **Text, vector, and hybrid search:**
  These search types are optimized for speed and efficiency. They return results quickly, making them a good choice for most standard Q&A and document lookup scenarios.

- **Hybrid multimodal search:**
  This search type processes both text and images, which can increase processing time compared to text-only searches. Performance depends on the size and type of your data.

- **Deep search:**
  Deep search uses advanced large language models and is based on the LazyRAG approach. While it can deliver higher-quality and more contextually rich answers, it might require more compute resources and take longer to return results than other search types.
  To learn more about the technology behind deep search, see [LazyGraphRAG: Setting a new standard for quality and cost](https://www.microsoft.com/en-us/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0).

## Tips for choosing a search type

Use these tips to help you select the best search type for your Edge RAG deployment and scenario:

- Pick the search type that matches your data and user needs.
- Hybrid search is a good default for most scenarios.
- For advanced or multimodal needs, deploy Edge RAG with your own language model to use deep or hybrid multimodal search.
- Not all search types are available for every model. For more information, see [Search type availability by deployment](#search-type-availability-by-deployment).

## Search type parameters

The following tables list and describe all parameters you can configure for each search type in Edge RAG. Not all parameters are available for every search type or deployment. The Edge RAG developer portal shows only the options that apply to your selected search type and model.

Model and search parameters apply to text, vector, hybrid, and hybrid multimodal search types. These parameters aren't available for deep search.

Deep search parameters are only available when you select deep search as your search type. These settings don't apply to other search types.

### Model parameters

The following model parameters are available for text, vector, hybrid, and hybrid multimodal search types. These settings control how the language model generates answers, such as the creativity or focus of responses, rather than how documents are retrieved or ranked. Model parameters aren't available for deep search.

| Model parameter | Description |
|---|---|
| **Temperature** | Controls randomness. Lowering the temperature means that the model produces more repetitive and deterministic responses. Increasing the temperature results in more unexpected or creative responses. Try adjusting temperature or Top P but not both.  |
| **Top-P (nucleus sampling)** | Similar to temperature, this controls randomness but uses a different method. Lowering Top P narrows the model's token selection to likelier tokens. Increasing Top P lets the model choose from tokens with both high and low likelihood. Try adjusting temperature or Top P but not both. |

### Search parameters

The following search parameters are available for text, vector, hybrid, and hybrid multimodal search types. These parameters aren't available for deep search. While model parameters control how the language model generates answers, these search parameters let you fine-tune how Edge RAG retrieves, filters, and ranks documents from your indexed data. These parameters help you optimize the relevance and precision of the information sent to the model for each query.

These parameters only apply to the **Knowledge-based** chat experience.

| Search parameter| Description |
|---|---|
| **Top-N documents** | Refers to number of most relevant chunks given to the language model. Choosing *N* depends on the application's needs:<br> <br>- For broad coverage, use a larger *N*. For example, 10-20. <br>- For precision-critical tasks, use a smaller *N*. For example, 3-5.<br><br> Experiment with *N* values to balance retrieval accuracy and downstream model performance. Bigger the *value of N*, the longer the inference time.  |
| **Text strictness** | Controls how strictly the RAG system filters and ranks retrieved text documents before passing them to the generation model.<br><br>- **Low strictness**: More documents are considered relevant, even if they're only loosely related to the query.<br>- **High strictness**: Only documents that closely match the query are used. |
| **Image strictness** (Hybrid multimodal search only) | Similar to text strictness but applies to retrieved image data.<br><br>- **Low strictness**: The system might retrieve a broader range of images.<br>- **High strictness**: Only images that are very closely aligned with the query are retrieved. |

### Deep search parameters

The following parameters let you customize how deep search retrieves, expands, and reasons over information to deliver high-quality answers for complex queries. Deep search parameters are only available when you select deep search as your search type. These settings don't apply to other search types.

| Name | Description |
|---|---|
| **Query expansion** | Automatically expands user queries with related terms and synonyms to improve search coverage. |
| **Number of sub-queries** | Sets how many sub-queries are generated from the original query to broaden the search and improve recall. |
| **Hypothetical answer generation** | Generates hypothetical answers to improve retrieval relevance, helping the system find better supporting content. |
| **Sub-query expansion** | Applies query expansion to each sub-query individually for even broader coverage. |
| **Relevance test budget** | Sets the maximum number of candidate results to test for relevance, balancing quality and performance. |
| **Text unit sample size** | Controls the number of text units (such as document chunks) sampled for evaluation during search. |
| **Maximum context tokens** | Sets the maximum number of tokens (words or pieces of words) included in the context window for the model to consider when generating an answer. |
| **Response format** | Lets you specify instructions or preferences for how the answer should be formatted (for example, concise and precise). |

## Related content

- [Deployment overview for Edge RAG](deploy-overview.md)
- [Deploy Edge RAG](deploy.md)
- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
- [Set up data query](set-up-data-query.md)

