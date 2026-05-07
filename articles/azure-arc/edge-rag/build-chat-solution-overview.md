---
title: Configure the Knowledge Layer for Agents and Tools with Foundry Local
description: "Learn about configuring the Knowledge Layer in Agents and Tools with Foundry Local, including data ingestion, collections, and data querying."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/04/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a developer or IT professional, I want to understand how to configure data ingestion, optimize chunk and model settings, and integrate the chat endpoint in Agents and Tools with Foundry Local so that I can enable effective, secure, and contextually relevant chat experiences for end users in my organization.
ms.custom:
  - build-2025
---

# Configure the Knowledge Layer for Agents and Tools with Foundry Local

This article provides a high-level overview of configuring the Knowledge Layer in Agents and Tools with Foundry Local, including data ingestion, collections, and data querying.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Configuring the solution in the developer portal

As part of Agents and Tools with Foundry Local, you deploy a local developer portal on the AKS cluster. Developers can access this portal to complete the following tasks:

- **Data ingestion**: Provide the on-premises data source and customize settings of the RAG pipeline.
- **Data query**: Provide a custom system prompt, modify model parameters, and evaluate the efficacy of the chat solution by using the chat playground.
- **Collections management**: Create and manage collections to organize your vector data into logical groupings.

Access the portal via the redirect URI (for example, `https://arcrag.contoso.com`) that you provide at the time of extension deployment or the redirect URI you provide during app registration.

To authenticate and authorize your access to the portal, make sure you have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Data ingestion

Data ingestion means you add your on-premises data and set up options so the data is easy to search. This way, when someone asks a question, the system can find the right information and give it to the language model as context.

### Collections

Before ingesting data, you can optionally create a **collection** to organize your vector data. Collections are logical groupings. Each collection maps to dedicated Milvus vector collections and Postgres tables.

- If you don't specify a collection, data is ingested into the default `edgeragapp` collection.
- To create a new collection, use the [Collections API](collections-overview.md) or specify a collection name during ingestion.
- Multiple collections let you separate data by department, tenant, or use case, with per-collection RBAC.

For more information, see [Collections overview](collections-overview.md).

### Planning data ingestion

Before you start configuring your chat solution, complete the following steps:

- **Prepare the data**. Review [supported data sources](requirements.md#supported-data-sources).  Make sure all your private data is in a network file system (NFS) share that's accessible from Agents and Tools with Foundry Local. For data ingestion, you need the NFS share path, NFS user ID, and NFS group ID.

  Make sure that the files aren't password protected or otherwise encrypted for the Agents and Tools with Foundry Local application to be able to access the data.

- **Choose the right settings for data ingestion**. Before you add a data source in Agents and Tools with Foundry Local, choose the appropriate ingestion type, chunk settings, and sync frequency.

### Ingestion type

When you're working with documents that include tables and charts, it's important to choose the right parsing approach during ingestion. Agents and Tools with Foundry Local gives you two options for ingestion:

- **Basic**: Quickly extracts free-form text from your documents. This is the default option that's relatively fast and efficient, but it might not capture the structure of tables, charts, or images.
- **Advanced**: Goes deeper by extracting text structure, tables, images, and other elements. It's slower than basic parsing, but you get higher accuracy and fidelity, especially for complex documents.

Pick the option that best matches your needs. If you're after speed, use basic. If you need detailed, structured data, use advanced parsing. For more information, see [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md).

### Chunk settings

Before you add a data source in Agents and Tools with Foundry Local, choose the appropriate chunk size, chunk overlap, and sync frequency. Here's some high-level guidance to help you select the right chunk settings for your data, as provided by Azure:

- **Chunk size**: Define a fixed size that's sufficient for semantically meaningful paragraphs (for example, 200 words) and allows for some overlap (for example, 10-15% of the content). This approach can produce good chunks as input for embedding vector generators.

  | **Processor** | **Recommended Chunk Size** | **Max Supported Size** |
  |---|---|---|
  | **GPU** | 2000 | 4000 |
  | **CPU-only** | 2000 | 2000 |

- **Chunk overlap**: When you chunk data, overlapping a small amount of text between chunks can help preserve context. Start with an overlap of approximately 10%. For example, given a fixed chunk size of 256 tokens, begin testing with an overlap of 25 tokens. The actual amount of overlap varies depending on the type of data and the specific use case, but 10-15% works for many scenarios.

  |**Processor** | **Recommended Chunk Overlap** | **Max Supported Overlap** |
  | ---|---|---|
  | **GPU** | 200 | 1000 |
  | **CPU-only** | 200 | 200 |

When you chunk data, consider these factors:

- **Shape and density of your documents**: If you need intact text or passages, larger chunks and variable chunking that preserves sentence structure can produce better results.

- **User queries**: Larger chunks and overlapping strategies help preserve context and semantic richness for queries that target specific information.

- **Large Language Models (LLM) have performance guidelines for chunk size**. Set a chunk size that works best for all of the models you're using. For instance, if you use models for summarization and embeddings, choose an optimal chunk size that works for both.


### Data ingestion by using REST APIs

You can also perform data ingestion programmatically by using the REST APIs:

- [Collections API](collections-overview.md) — Create and manage collections before ingesting.
- [Ingestion API](agent-manager-api.md) — Start ingestion jobs targeting a specific collection.

- Data ingestion can take a long time depending on the size of the data, the compute resources available to the embedding model, and other factors.
- Create as many data ingestions as you'd like. Data is vectorized and stored in the collection you specify (or the default `edgeragapp` collection if none is specified). You can create multiple collections to organize data by domain, department, or use case.

## Data query

In Agents and Tools with Foundry Local, setting up a data query means you create a system prompt, adjust model settings for your needs, and check that the solution works as expected.

### Choosing the right prompt and model parameters

A critical part of prompt engineering is providing the right system prompt and model parameters according to your data and use case.

- To choose the right system prompt, see [Foundry Tools - safety system messages](/azure/ai-services/openai/concepts/system-message) for high-level guidance.
- For information and guidance about choosing model and search parameters, see [Search type parameters](search-types.md#search-type-parameters).

### Chatting with Agents and Tools with Foundry Local

Agents and Tools with Foundry Local offers two chat experiences:

- **Knowledge-based chat**: Chat with the model using your own ingested data as context. This means the model’s answers are based on your organization’s documents and data sources, so you get responses that are relevant and grounded in your latest information.
- **Model chat**: Chat directly to the language model, without using your ingested data as context. This is helpful when you want to ask general questions, test the model’s raw capabilities, or just see how it responds without any extra information.

Switch between knowledge-based chat and model chat depending on what you need.

### Data querying by using REST APIs

In addition to the developer portal, you can use the REST APIs to configure the chat solution, such as providing the system message and model parameters.

### Consuming the chat endpoint

After you set up data ingestion and you're satisfied with the chat solution as the prompt engineer, you can integrate the chat endpoint in downstream line-of-business applications. Alternately, end users can use the chat application provided out-of-the-box to get started quickly. For more information, see [Test the chat solution for Agents and Tools with Foundry Local](test-end-user-app.md).

If you want to integrate the chat endpoint in one of your line-of-business applications, use the  REST APIs.

### Connecting to the Agentic Layer

If you deployed in combined mode, AI agents can access your ingested collections through the Agentic Layer:

1. Create [Knowledge Sources](knowledge-sources-guide.md) pointing to your collections (using `kind: indexed_sources_mcp` with `indexed_source_ref` = collection name).
1. Group them into a [Knowledge Base](knowledge-bases-guide.md) by linking them to your default knowledge base.

For a full walkthrough, see [Query your data quickstart](create-agent-quickstart.md).

## Related content

- [Supported data sources](requirements.md#supported-data-sources)
- [Add data source for the chat solution](add-data-source.md)
- [Set up the data query for chat solution](set-up-data-query.md)
