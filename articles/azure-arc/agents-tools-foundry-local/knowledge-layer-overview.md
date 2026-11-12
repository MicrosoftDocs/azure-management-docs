---
title: Agentic Retrieval Overview for Agents and Tools with Foundry Local
description: "Learn how Agentic Retrieval in Agents and Tools with Foundry Local uses indexed sources and remote sources, including data ingestion, collections, and data querying."
ms.reviewer: assafakiva
ms.topic: concept-article #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
#CustomerIntent: As a developer or IT professional, I want to understand how to configure data ingestion, optimize chunk and model settings, and integrate the chat endpoint in Agents and Tools with Foundry Local so that I can enable effective, secure, and contextually relevant chat experiences for end users in my organization.
ms.custom:
  - build-2025
---

# Agentic Retrieval overview

Agentic Retrieval is the retrieval and grounding component of Agents and Tools with Foundry Local. It finds relevant information in your organization's data and supplies it, with references, so that AI applications can give grounded answers. A *grounded* answer is based on retrieved content instead of only on the model's training data.

Agentic Retrieval supports two kinds of sources:

- **Indexed sources**: The extension ingests your documents, splits them into chunks, generates embeddings, and stores the searchable content in local *collections*. An *embedding* is a list of numbers that represents the meaning of a piece of text, so that similar text can be found.
- **Remote sources**: The extension gets information at request time from connected systems, without ingesting it into local collections. Remote sources include customer-provided Model Context Protocol (MCP) servers and SQL databases (in preview). *Remote* describes how the source is accessed; the connected system can still run in your local environment.

A *knowledge source* identifies one configured source of information. A *knowledge base* groups knowledge sources so that agents can use them. For remote sources, see [Configure a knowledge source](knowledge-sources-guide.md). For an introduction to all components, see [What is Agents and Tools with Foundry Local?](overview.md)

This article focuses on indexed sources. It describes the ingestion and retrieval pipeline for indexed collections that you register as `indexed_sources_mcp` knowledge sources. Indexed sources support app-role collection access. Callers allowed by an Azure role see all collections; collection app roles don't limit that access.

> [!NOTE]
> Local ingestion and the developer portal are available only when `indexedSourcesEnabled=true`.

## Configure indexed sources

To use indexed sources, complete the following tasks:

- **Data ingestion**: Provide the on-premises data source and customize settings of the RAG pipeline.
- **Data query**: Provide a custom system prompt, modify model parameters, and evaluate the quality of the answers.
- **Collections management**: Create and manage collections to organize your vector data into logical groupings.

## Data ingestion

Data ingestion means you add your on-premises data and set up options so the data is easy to search. This way, when someone asks a question, the system can find the right information and give it to the language model as context.

### Collections

Before you ingest data, create the target **collection**. Collections are logical groupings of vector data. Each collection maps to dedicated Milvus vector collections and Postgres tables.

- If you don't specify a collection in an API request, data goes to the default `edgeragapp` collection, which is created automatically on startup. Verify that it exists with `GET /collections/edgeragapp`.
- Use `POST /collections` only if `edgeragapp` was deleted or when creating custom collections for API ingestion. See the [Collections REST API reference](reference/collections-api-reference.md). Ingestion doesn't create collections.
- Use multiple collections to separate data by department, tenant, or use case. App-role collection access limits callers who use collection app roles. It doesn't limit callers allowed by an Azure role.

For more information, see [Collections overview](collections-overview.md).

### Plan data ingestion

Before you add an indexed source, complete the following steps:

- **Prepare the data**. Review [supported data sources](requirements.md#supported-data-sources). For NFS, use an accessible share and provide its path. For AUTH_SYS authentication, also provide the user ID and group ID. For Kerberos, configure the worker nodes as described in [NFS with Kerberos authentication](connect-file-share-kerberos-overview.md). For SharePoint Server, configure server-to-server authentication and provide the site and folder information required by the ingestion request.

  Ensure that the files aren't password protected or otherwise encrypted so that the extension can access the data.

- **Choose the sync frequency**. Before you add a data source, decide whether to run ingestion once or sync it on a schedule. Parsing and text chunking use the built-in Docling settings.

### Ingestion parsing

During ingestion, Agents and Tools with Foundry Local parses documents to extract text and structure for chunking and embedding. This parsing is designed for mixed-content files, including tables, charts, and images, so retrieved context is more accurate for chat responses.

Expect ingestion time to vary based on document complexity and size. Documents with more detailed structure usually take longer to process, but they produce higher-fidelity content for retrieval and grounding.

For parsing details and supported behaviors, see [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md).

### Chunk settings

The service uses Docling HybridChunker with the BGE-M3 tokenizer to split text into chunks of about 512 tokens. A token is a small unit of text, such as a word or part of a word. The chunker uses document structure to keep related text together.

> [!NOTE]
> The portal shows chunk size and overlap fields, and the API accepts `chunkSize` and `chunkOverlap`. The current release doesn't apply these values.

### Data ingestion by using REST APIs

You can also perform data ingestion programmatically by using the REST APIs:

- [Collections API](reference/collections-api-reference.md) — Create and manage collections before ingesting.
- Data ingestion can take a long time depending on the size of the data, the compute resources available to the embedding model, and other factors.
- Only one ingestion run executes at a time. Other runs wait with the `PENDING` status. The platform stores the data in the existing collection you specify, or in the existing default `edgeragapp` collection.

## Data query

In Agents and Tools with Foundry Local, setting up a data query means you create a system prompt, adjust model settings for your needs, and check that the solution works as expected.

### Choose the right prompt and model parameters

A critical part of prompt engineering is providing the right system prompt and model parameters according to your data and use case.

- To choose the right system prompt, see [Foundry Tools - safety system messages](/azure/ai-services/openai/concepts/system-message) for high-level guidance.
- For information and guidance about choosing model and search parameters, see [Search type parameters](search-types.md#search-type-parameters).

### Test queries

Compare these query experiences:

- **Knowledge-base**: Query the model with your ingested data as retrieved context.
- **Model-only**: Query the language model without retrieving ingested data.

Switch between **Knowledge-base** chat and **Model-only** chat depending on what you need.

### Data querying by using REST APIs

In the developer portal, **View the endpoint** displays the deprecated classic inference URL (`POST /chat/completions`), which is restricted to the developer portal browser origin and returns `403 Forbidden` for external callers. To configure and run queries from external applications, use Agent Service (`/openai/v1/responses` and `/openai/v1/conversations`) or Indexed Sources MCP (`/mcp`).

### Use the chat endpoint

After you validate data ingestion and prompt quality in the developer portal, integrate your line-of-business applications with Agent Service (`/openai/v1/responses` and `/openai/v1/conversations`) or Indexed Sources MCP (`/mcp`), or use the Local chat UI. (**View the endpoint** in the developer portal displays the deprecated classic `POST /chat/completions` URL, which is restricted to the developer portal browser origin and returns `403 Forbidden` for external callers.) The Local chat UI uses Responses and Conversations, which require an Azure role and have no app-role fallback. In connected deployments, assign **Reader** to list agents and **Cognitive Services OpenAI User** to run them. In disconnected deployments, use **Contributor**.

Classic chat completions still support app-role fallback. With that fallback, assign `EdgeRAGEndUser` and a collection app role whose value matches the collection name, such as `edgeragapp`. Without the matching collection role, the classic query fails with `403`. Callers allowed by an Azure role or the `EdgeRAGDeveloper` app-role fallback aren't limited by collection app roles. For role setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

For more information, see the following articles:

- [How the Local chat UI works](chat-experience.md)
- [Test the Local chat UI](test-end-user-app.md)

If you want to integrate chat and retrieval into your line-of-business applications, use the REST APIs for Agent Service (`/openai/v1/responses` and `/openai/v1/conversations`) or Indexed Sources MCP (`/mcp`).

### Connect collections to Agent Service

If you deployed with indexed sources enabled, AI agents in Agent Service can access your ingested collections:

1. Create [Knowledge Sources](knowledge-sources-guide.md) pointing to your collections (using `kind: indexed_sources_mcp` with `indexed_source_ref` = collection name), and check source readiness.
1. Group them into a [Knowledge Base](knowledge-bases-guide.md).
1. Add the knowledge base to an agent as a tool. See [Add tools to an agent](add-agent-tools.md#add-a-knowledge-base-to-an-agent).

A successful Knowledge Bases API create response completes validation and creates the managed `kb-<knowledge-base-name>` connection. There's no separate knowledge-base readiness poll.

For a full walkthrough, see [Quickstart: Create an agent and chat with it](quickstart-create-agent.md).

## Related content

- [Supported data sources](requirements.md#supported-data-sources)
- [Add a data source](add-data-source.md)
- [Set up the data query](set-up-data-query.md)
