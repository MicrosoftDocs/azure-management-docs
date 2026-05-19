---
title: Types of Search in Agents and Tools with Foundry Local
description: Learn about search types, their options, and how they're used in Agents and Tools with Foundry Local deployments.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/04/2026
ms.subservice: edge-rag
ai-usage: ai-generated
#CustomerIntent: As a cloud administrator or developer, I want to understand search types in Agents and Tools with Foundry Local so I can choose and configure the right search type for my deployment.
---

# Search types in Agents and Tools with Foundry Local

A search method or type, like full text, vector, or hybrid, controls how Agents and Tools with Foundry Local retrieves and ranks results from your indexed data. It shapes the way users interact with your solution and the quality of the answers they get.

You select a search type when you configure your data query settings in the Agents and Tools with Foundry Local developer portal. All search types are available with your BYOM language model endpoint.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Why your search type matters

Your choice of search type affects:

- The kinds of questions users can ask.<br>
  For example: Text search works best for simple keyword queries, while vector or hybrid search lets users ask more natural, conversational questions.
- The relevance and accuracy of results.
  For example: Hybrid search can return more relevant answers by combining exact matches with semantic understanding, while text search might miss contextually similar results.
- Access to advanced features like multimodal search. 
  For example: Hybrid multimodal search lets users find information in both text and images, which is a capability not available with other search types.

## Available search types

The following definitions describe each search type available in Agents and Tools with Foundry Local, so you can understand how they work and choose the best option for your scenario.

- **Full text search**: A search method that scans and matches the entire body of text in documents, by using keywords, phrases, or boolean queries to find relevant chunks in the supplied documents.

- **Hybrid search**: Combines both full-text search (keyword-based) and vector search (semantic similarity) to retrieve the most relevant documents. It uses the precision of keyword matching and the depth of semantic understanding for improved retrieval accuracy.

- **Hybrid multimodal search**: Combines full-text and vector search across both text and image data, allowing Agents and Tools with Foundry Local to retrieve and rank results from multiple data types in a single query. Unlike hybrid search, which works only with text, by using both keyword and semantic similarity, hybrid multimodal search extends this approach to include images. This approach enables richer, more comprehensive answers that draw from both textual, and visual content.

- **Vector search**: A search method that finds relevant documents by comparing the semantic similarity between vector embeddings of the user's query and precomputed embeddings of documents, typically using cosine similarity or other distance metrics in a vector space.

## Compare search types

Agents and Tools with Foundry Local supports several search types. The following table summarizes each search type, best use cases, and performance considerations:

| Search type               | What it does      | Best use cases       | Performance notes                                                                                  |
|--------------------------|------------------|-----------------|---------------------|
| **Text search**          | Finds exact words or phrases in your documents using keywords, phrases, or boolean queries.                                                                                                                              | Standard question and answer (Q&A), document lookup, compliance scenarios           | Fastest; highly efficient for simple lookups.                                                     |
| **Vector search**        | Uses semantic similarity to find contextually relevant results by comparing vector embeddings.                                                                                                                           | Conversational Q&A, finding related content                   | Fast; slightly more compute than text search.                                                      |
| **Hybrid search**        | Combines text (keyword) and vector (semantic) search for better coverage and accuracy.                                                                                            | Most scenarios; default for general use                       | Fast; combines strengths of text and vector search.                                                |
| **Hybrid multimodal**    | Combines text, vector, and image search to retrieve and rank results from multiple data types in a single query.                                                                  | Scenarios involving both text and images                      | Slower than text/vector; performance depends on data size/type. |
| **Full text search**     | Scans and matches the entire body of text in documents.            | Legal, compliance, or when exact matches are required         | Fastest for keyword lookups; might miss semantic matches.                                            |

## Search type availability by deployment

All search types are available with Agents and Tools with Foundry Local, since all deployments use BYOM:

| Search type | Available |
|---|---|
| Hybrid search | Yes |
| Text search | Yes |
| Vector search | Yes |
| Hybrid multimodal search | Yes |

For more information about deploying with a language model, see:

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Deploy the extension for Agents and Tools with Foundry Local](deploy.md)

## How to select a search type

Select the search type in the Agents and Tools with Foundry Local developer portal when you configure your data query settings. Some parameters might differ by search type and model.

## Collection-scoped search

You can scope all search types to specific collections. When you query through the developer portal or API, specify which collections to search:

- **Developer portal**: Select the target collection in the data query settings.
- **Inference API**: Set `data_sources[0].parameters.index_name` to the collection name.
- **MCP Server**: Set `collection_names` in the tool arguments. You can query multiple collections in a single request.

For more information, see [Collections overview](collections-overview.md) and [MCP Server overview](mcp-server-overview.md).

## Performance considerations

Each search type offers different performance characteristics. Consider these points when choosing the best option for your scenario:

- **Text, vector, and hybrid search:**
  These search types are optimized for speed and efficiency. They return results quickly, making them a good choice for most standard Q&A and document lookup scenarios.

- **Hybrid multimodal search:**
  This search type processes both text and images, which can increase processing time compared to text-only searches. Performance depends on the size and type of your data.

## Tips for choosing a search type

Use these tips to help you select the best search type for your Agents and Tools with Foundry Local deployment and scenario:

- Pick the search type that matches your data and user needs.
- Hybrid search is a good default for most scenarios.
- All search types are available with your BYOM endpoint.
- When using agents, the built-in MCP server provides search tools. See [MCP Server overview](mcp-server-overview.md).

## Search type parameters

The following tables list and describe all parameters you can configure for each search type in Agents and Tools with Foundry Local. Not all parameters are available for every search type or deployment. The Agents and Tools with Foundry Local developer portal shows only the options that apply to your selected search type and model.

### Model parameters

The following model parameters are available for text, vector, hybrid, and hybrid multimodal search types. These settings control how the language model generates answers, such as the creativity or focus of responses, rather than how documents are retrieved or ranked.

| Model parameter | Description |
|---|---|
| **Temperature** | Controls randomness. Lowering the temperature means that the model produces more repetitive and deterministic responses. Increasing the temperature results in more unexpected or creative responses. Try adjusting temperature or Top P but not both.  |
| **Top-P (nucleus sampling)** | Similar to temperature, this parameter controls randomness but uses a different method. Lowering Top P narrows the model's token selection to likelier tokens. Increasing Top P lets the model choose from tokens with both high and low likelihood. Try adjusting temperature or Top P but not both. |

### Search parameters

The following search parameters are available for text, vector, hybrid, and hybrid multimodal search types. While model parameters control how the language model generates answers, these search parameters let you fine-tune how Agents and Tools with Foundry Local retrieves, filters, and ranks documents from your indexed data. These parameters help you optimize the relevance and precision of the information sent to the model for each query.

These parameters only apply to the **Knowledge-based** chat experience.

| Search parameter| Description |
|---|---|
| **Top-N documents** | Refers to number of most relevant chunks given to the language model. Choosing *N* depends on the application's needs:<br> <br>- For broad coverage, use a larger *N*. For example, 10-20. <br>- For precision-critical tasks, use a smaller *N*. For example, 3-5.<br><br> Experiment with *N* values to balance retrieval accuracy and downstream model performance. Bigger the *value of N*, the longer the inference time.  |
| **Text strictness** | Controls how strictly the RAG system filters and ranks retrieved text documents before passing them to the generation model.<br><br>- **Low strictness**: More documents are considered relevant, even if they're only loosely related to the query.<br>- **High strictness**: Only documents that closely match the query are used. |
| **Image strictness** (Hybrid multimodal search only) | Similar to text strictness but applies to retrieved image data.<br><br>- **Low strictness**: The system might retrieve a broader range of images.<br>- **High strictness**: Only images that are closely aligned with the query are retrieved. |

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Deploy Agents and Tools with Foundry Local](deploy.md)
- [Configuring the chat solution for Agents and Tools with Foundry Local](build-chat-solution-overview.md)
- [Set up data query](set-up-data-query.md)

