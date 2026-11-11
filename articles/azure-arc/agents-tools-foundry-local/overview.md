---
title: What is Agents and Tools with Foundry Local?
description: "Learn about Agentic Retrieval, Agent Service, and the local chat UI in the Agents and Tools with Foundry Local extension for Azure Local."
ms.reviewer: assafakiva
ms.topic: overview #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ms.custom:
  - build-2025
# Customer intent: As a business decision maker, I want to evaluate Agents and Tools with Foundry Local so that I can determine whether it meets my on-premises AI and compliance needs.
---
# What is Agents and Tools with Foundry Local?

**Agents and Tools with Foundry Local** is an Azure Arc-enabled Kubernetes extension that brings enterprise knowledge retrieval, AI agents, and a built-in chat experience to Azure Local. Use it to build AI applications that work with your organization's data and tools on infrastructure you manage.

The extension includes three components:

| Component | What it provides |
|---|---|
| **Agentic Retrieval** | Retrieval and grounding across indexed documents and connected knowledge sources, helping applications answer questions with relevant information and citations. |
| **Agent Service** | Capabilities to create, version, and run agents with your instructions, model connections, and supported tools. |
| **Local chat UI** | A ready-to-use browser experience for interacting with agents, asking follow-up questions, and viewing answers and sources. |

The extension works with a language model endpoint. Use **Foundry Local on Azure Local** to host supported models locally, or configure a supported external endpoint when your connectivity and data-handling requirements permit.

> [!IMPORTANT]
> Agent Service, managed knowledge-base connections, SQL knowledge sources, and disconnected operations are in preview. Microsoft 365 Local is in **limited preview to approved customers** and is supported only with Agentic Retrieval. For access information, see [Microsoft 365 Local limited preview](#microsoft-365-local-limited-preview). For release-specific information, see [Release notes](release-notes.md).

## Agentic Retrieval

Agentic Retrieval helps AI applications answer questions using your organization's information. It retrieves relevant content from configured sources and supplies supporting context and references for grounded answers.

Agentic Retrieval includes more than document ingestion and vector search. It supports both **indexed sources**, where content is prepared and stored in local collections, and **remote sources**, where information is accessed through connected services.

### Indexed sources

Use indexed sources when you want to ingest supported documents into the extension's local retrieval system. The ingestion pipeline parses documents, divides content into chunks, generates embeddings, and stores searchable content in **collections**.

Collections organize indexed content so that you can manage and query different sets of documents. Supported ingestion sources include NFS file shares and SharePoint Server, with source-specific authentication and configuration requirements.

Indexed retrieval supports text, vector, and hybrid search. Image and multimodal retrieval depend on the deployment configuration and model capabilities.

For more information, see [Collections](collections-overview.md) and [Search types](search-types.md).

### Remote sources

Remote sources provide access to information without requiring you to ingest it into the extension's local document collections. "Remote" describes how the source is accessed; the connected system can still run within your local environment.

Supported integration options include:

| Source | Retrieval approach |
|---|---|
| **Customer-provided MCP servers** | Access information through a configured Model Context Protocol (MCP) server. Available capabilities depend on the server and its configuration. |
| **SQL databases, in preview** | Retrieve information from an approved, read-only set of SQL Server, PostgreSQL, or MySQL tables and columns. |
| **Microsoft 365 Local, in limited preview** | Approved customers can use Agentic Retrieval to retrieve documents, email, and calendar information from supported on-premises SharePoint Server and Exchange Server deployments. |

A **knowledge source** identifies a configured source of information. A **knowledge base** groups knowledge sources and provides instructions for retrieving relevant information. You can combine supported source types to ground answers in more than one information source.

Agentic Retrieval is included by default with the extension. **Local indexing is a separate configuration choice:** you need the indexing workloads for indexed content, but not for every retrieval scenario.

### Microsoft 365 Local limited preview

Microsoft 365 Local connects Agentic Retrieval to supported on-premises SharePoint Server and Exchange Server environments. It retrieves information at request time without requiring that content to be ingested into a separate local document index. The source systems enforce the signed-in user's permissions.

This integration is for the supported on-premises server products, not SharePoint Online or Exchange Online. Administrator preparation is required.

> [!IMPORTANT]
> Microsoft 365 Local integration with Agentic Retrieval is available in **limited preview to approved customers**. To request access, complete the [preview registration form](https://aka.ms/AgenticRetrievalM365LocalPuP).
>
> In this preview, Microsoft 365 Local is supported only with Agentic Retrieval. It isn't supported as a general Agent Service integration.

For prerequisites and setup, see [Prepare infrastructure for Microsoft 365 Local](prepare-microsoft-365-local.md).

## Agent Service

**Agent Service, in preview**, lets you create and run agents tailored to your applications. You define an agent's instructions, select its model connection, and configure the supported tools it can use.

Use Agent Service to:

- **Define agent behavior** through instructions, a model connection, and tool configuration.
- **Manage agent versions**, including selecting the version that answers requests and returning to an earlier version.
- **Connect supported tools**, such as trusted MCP servers and supported knowledge-base connections.
- **Run agents through APIs**, with streaming responses and conversations that preserve context across turns.

During execution, an agent can call its configured tools, use their results, and generate a response. Available operations depend on the tools you configure, their permissions, and the selected model's capabilities.

Agent Service separates agent configuration from application code. Applications use the Responses and Conversations APIs to interact with agents, while developers manage agent definitions and versions through the management APIs.

For an introduction, see [Create an agent and chat with it](quickstart-create-agent.md).

## Local chat UI

The local chat UI provides a built-in browser experience so users can interact with configured agents without a custom frontend.

Users can:

- Select an available agent and start a conversation.
- Ask follow-up questions within a conversation.
- Receive streaming answers with formatted text and citations.
- View tool activity and the sources used to answer a question.
- Return to, rename, or delete their conversations.

The chat UI is the user-facing interface. Backend services handle agent execution, retrieval, model calls, and access checks.

Sign-in is required. Connected deployments use Microsoft Entra ID; disconnected deployments use the configured local identity provider. Administrators must assign the permissions required to list and interact with agents.

For more information, see [How chat works](chat-experience.md).

## Deployment and model options

The validated on-premises deployment runs on **Azure Arc-enabled Kubernetes on Azure Local**. Azure Arc provides extension deployment and lifecycle management.

The Agents and Tools extension doesn't include a generative language model. Model serving is configured separately:

| Model option | When to use it |
|---|---|
| **Foundry Local on Azure Local** | Run supported models locally through the separate Foundry Local extension. This option is recommended for local inference. |
| **Supported external model endpoint** | Use an external OpenAI-compatible endpoint, including a model deployed in Microsoft Foundry, when network connectivity and organizational policy allow it. |

Choose a model that supports the request formats and capabilities required by your application, including tool calling where applicable. Hardware requirements depend on the model, enabled retrieval workloads, and expected usage.

For guidance, see [Choose your language model](prepare-language-model.md) and [Requirements](requirements.md).

### Disconnected operations

Disconnected operations on Azure Local are available in preview. Deployment requires preparing, transferring, and importing the appropriate expansion pack into the disconnected environment.

Models, identity services, data sources, and tools must be available within that environment. Cloud-hosted models and tools that require access to Azure aren't available while disconnected.

See [Disconnected operations overview](disconnected-operations/overview-disconnected.md).

## Security and data location

Local hosting lets you run retrieval and agent workloads close to your data. Your deployment stores indexed content, embeddings, configuration, and conversation state.

Your complete data-processing boundary depends on the connections you configure. External model endpoints and MCP servers can receive user requests, retrieved content, or tool arguments. Connected deployments can also use Azure services for management and monitoring. Review these connections against your organization's requirements.

Configure extension access, model access, and source permissions separately. Access to the extension doesn't automatically grant access to a model or connected system. Authorization behavior differs between indexed collections and live connectors.

For setup guidance, see [Configure authentication](prepare-authentication.md).

## Key concepts

Review the following key concepts for Agents and Tools with Foundry Local:

- **Chunking** splits large documents into smaller, manageable text blocks (chunks).
  The current release uses Docling HybridChunker with the BGE-M3 tokenizer to split text into chunks of about 512 tokens. A token is a small unit of text, such as a word or part of a word. The ingestion API accepts `chunkSize` and `chunkOverlap`, but these settings don't change text chunking in this release.

- **Data ingestion** is process of importing and preparing external content, such as documents or images, to be used for retrieval. This includes preprocessing steps like cleaning, formatting, and organizing data.

- **Embedding models** transform text, images, or other data into dense numerical vectors (embeddings) that capture semantic meaning. These vectors represent relationships between inputs, allowing for similarity comparisons and clustering.

- **Inferencing** refers to the process of using a trained model to generate predictions or outputs based on new input data. In language models, inferencing involves tasks like completing text, answering questions, or generating summaries.

- **Language models** are AI systems trained to understand, generate, and manipulate human language. They predict text based on input, enabling tasks like text generation, translation, summarization, and question answering. Agents and Tools with Foundry Local supports two language model endpoint options. The recommended option is a Foundry Local on Azure Local endpoint. This option runs on the same Azure Arc-connected cluster as the extension. You can also use an external Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry.

- **Model parameters** control how the language model generates text, such as the creativity, diversity, and focus of responses. Common parameters include Temperature and Top-p. Model parameters don't affect which documents are retrieved, only how the model generates its response. For more information, see [Search type parameters in Agents and Tools with Foundry Local](search-types.md#search-type-parameters).

- **Query** is the input provided to a language model to elicit a response or perform a specific task. It can be a question, a prompt, or a set of instructions, depending on the use case.

- **Retrieval-Augmented Generation (RAG)** combines a retrieval system with a generative language model to produce responses enriched by external knowledge. It retrieves relevant context from a database or document store to augment the model's generation capabilities, ensuring accurate and up-to-date information.

- **Search parameters** are settings that control how Agents and Tools with Foundry Local retrieves, filters, and ranks documents from your indexed data before passing them to the language model. These parameters help you fine-tune the relevance, precision, and scope of the information used to answer user queries. For more information, see [Search type parameters in Agents and Tools with Foundry Local](search-types.md#search-type-parameters).

- **Search type**: A search type is the method Agents and Tools with Foundry Local uses to find and rank information from your indexed data. It determines how the system retrieves relevant content to answer user questions, such as by matching keywords, using semantic similarity, or combining multiple approaches. Agents and Tools with Foundry Local supports several search methods for retrieving information, including full text search, hybrid search, hybrid multimodal search, and vector search. For more information, see [Types of search in Agents and Tools with Foundry Local](search-types.md).

- **System prompt** are predefined instructions or messages provided to a language model at the start of a conversation or task to influence its behavior. These prompts define the model's role, tone, or task-specific context. For example, "You're a helpful assistant" or "Provide concise technical explanations." By shaping the initial context, system prompts ensure that the model generates responses aligned with the desired objective or persona.

- **Vector database** is a specialized database to store vector embeddings. It's designed to handle high-dimensional vectors and enables fast and scalable similarity searches.

- **Vectorization** means transforming text into numerical representations, or embeddings, using an embedding model such as Sentence Transformers. These embeddings capture the semantic meaning of text, enabling efficient and accurate comparisons.

- **Agent** is an AI assistant configured with instructions, a model, and tools such as knowledge bases and MCP servers. Each change to an agent creates a new, unchangeable agent version. Agents process user queries through multiturn conversations, invoking tools as needed.

- **Knowledge base** is a grouping of knowledge sources that an agent can search as a tool. When the agent processes a query, it can search all knowledge sources in the knowledge base.

- **Knowledge source** is a named connection used by a knowledge base. Published source kinds include `remote_mcp` for an external MCP server, `indexed_sources_mcp` for an indexed collection, `sql` for an approved read-only database slice, and Microsoft 365 Local (`sharepoint` and `exchange`). SQL knowledge sources are in preview. Microsoft 365 Local knowledge sources are in limited preview for approved customers.

- **Collection** is a logical grouping of ingested vector data. Each collection maps to Milvus vector collections and Postgres tables, and can be independently created, queried, and deleted.

- **MCP (Model Context Protocol)** is an open protocol for connecting AI agents to external tools and data sources. Agents and Tools with Foundry Local includes a built-in MCP server with six search tools and can also connect to external MCP servers.

- **Conversation** is a multiturn session between a user and an agent. Conversations contain ordered items, such as user questions and agent answers, and are scoped to a single user.

- **Response** is one run of an agent. The agent reads the input and earlier conversation turns, invokes tools, and generates an answer. Responses support streaming through Server-Sent Events (SSE).

- **Indexed sources setting** determines whether the local indexed-source stack is deployed. The core Agent Service workloads are always deployed. Set `indexedSourcesEnabled` to `true` (the default) for local ingestion and retrieval, or `false` to omit that stack.

## User roles

The following tasks describe how people use Agents and Tools with Foundry Local. An administrator assigns Azure roles at the extension resource scope. Roles inherited from subscription, resource-group, or cluster scope also apply.

- **Lifecycle management of the extension**: Users manage the lifecycle of the Agents and Tools with Foundry Local Arc extension. This role includes tasks such as setting up the necessary infrastructure, deploying the extension, performing updates, monitoring its performance, and handling its eventual deletion. Typically, an IT administrator with access to the underlying Azure Local and Azure Kubernetes Service (AKS) on Azure Local infrastructure takes on these responsibilities.
- **Development and evaluation of agents and chat endpoints**: Developers configure agents, knowledge bases, and sources, and manage indexed data. Use **Cognitive Services Contributor** for changes and **Reader** for read-only access.
- **Querying data and chatting with agents**: Users and applications need **Cognitive Services OpenAI User** for chat and agent execution. Also assign **Reader** to list agents. Responses and Conversations have no app-role fallback.
- **Agent Service administration**: Administrators manage agents, connections, sources, and knowledge bases. Use **Cognitive Services Contributor**. The `EdgeRAGDeveloper` app role remains a management fallback and is required for full definitions from agent list and get operations.

In disconnected deployments, use **Reader** for reads and **Contributor** for changes and chat. The extension's managed identity also needs **Reader** at its own extension scope to check role assignments. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

## Next steps

- [Plan your deployment](deploy-overview.md).
- [Configure knowledge sources](knowledge-sources-guide.md).
- [Create your first agent](quickstart-create-agent.md).
- [Review known issues and limitations](known-issues.md).
