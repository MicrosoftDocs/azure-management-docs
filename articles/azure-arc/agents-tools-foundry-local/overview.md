---
title: Agents and Tools with Foundry Local Overview
description: "Learn about Agentic Retrieval in Foundry Local, the Azure Arc-enabled Kubernetes extension in the Agents and Tools with Foundry Local platform, used to search on-premises data with generative AI."
author: cwatson-cat
ms.author: cwatson
ms.topic: overview #Don't change
ms.date: 05/26/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a business decision maker, I want to evaluate Agentic Retrieval in Foundry Local for on-premises data retrieval, so that I can determine if it aligns with our regulatory compliance and AI application needs while enhancing insights and decision-making processes.
---
# What is Agents and Tools with Foundry Local?

Agents and Tools with Foundry Local is part of Microsoft's [adaptive cloud](https://azure.microsoft.com/solutions/adaptive-cloud) approach, extending AI reasoning and grounding capabilities to on-premises, distributed, and disconnected environments managed through Azure Arc.

Agentic Retrieval is the [Azure Arc-enabled Kubernetes extension](/azure/azure-arc/kubernetes/extensions-release) at the core of the Agents and Tools with Foundry Local platform. It provides an **agentic Retrieval-Augmented Generation (RAG) platform** at the edge, combining a knowledge layer (document ingestion, embedding, vector search) with an agentic layer (AI agents, knowledge orchestration, MCP server) to deliver intelligent, multistep assistants grounded in your private on-premises data.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

Agentic Retrieval in Foundry Local is supported and validated on Azure Arc-enabled Kubernetes on Azure Local (formerly Azure Stack HCI) infrastructure and as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

For more information, see [Azure Arc](/azure/azure-arc/overview), [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/), and [Azure Arc extensions](/azure/azure-arc/kubernetes/conceptual-extensions). 

## Platform overview

The platform is built on three components that work together:

| Component | What it does |
|---|---|
| [**Local agentic RAG**](#local-agentic-rag) | AI agent orchestration with knowledge bases, knowledge sources, and an MCP server for multistep reasoning over your data. |
| [**Local knowledge sources**](#local-knowledge-sources) | Data ingestion, embedding, and retrieval pipeline that indexes your on-premises documents into searchable collections. |
| [**Local chat experience**](#local-chat-experience) | A built-in chat UI for interacting with agents, managing conversations, and viewing citations. No custom frontend is required. |

Additional platform capabilities include:

- **Foundry Local language model endpoint (recommended)** - Use a Foundry Local on Azure Local endpoint to run the language model on the same Azure Arc-connected cluster as the extension.
- **Bring Your Own Model (BYOM)** - Connect an external language model endpoint that supports the OpenAI-compatible chat completions API, such as an endpoint deployed in Microsoft Foundry.
- **Two GPU-accelerated models** for text embedding (BGE-M3) and image embedding (CLIP ViT-L/14) running locally on two GPUs. Docling (document parser) runs on CPU.
- **Independent deployment modes** - Deploy the full platform, or just the agentic layer or knowledge layer separately.
- **Image retrieval** - Ingest and retrieve relevant images as contextual references alongside text. Agentic Retrieval in Foundry Local isn't a visual language model (VLM).

## Key components

The platform includes three key components that work together to help you build and run agentic RAG solutions on your on-premises data.

### Local agentic RAG

The agentic layer adds planning, tool use, and conversation orchestration to the platform. It lets you build AI assistants that manage multistep interactions, call MCP-connected knowledge tools, and generate responses grounded in your private data.

Key capabilities:

- **Agent execution** - agents process user queries by reasoning over instructions, invoking tools, and generating responses.
- **Knowledge orchestration** - connect agents to one or more data sources through knowledge bases and knowledge sources.
- **MCP server** - a built-in Model Context Protocol (MCP) server with search tools, plus support for connecting to external MCP servers.
- **Conversation management** - threads, messages, and runs for managing multistep interactions with state.

You can deploy the agentic layer together with the knowledge layer or by itself. In a combined deployment, agents query collections indexed locally. In an agentic-only deployment, agents connect to external MCP servers instead.

For more information, see [Agentic layer overview](agentic-overview.md).

### Local knowledge sources

The knowledge layer provides a turnkey data ingestion and RAG pipeline that keeps all data on-premises. It handles the full lifecycle of your data, from document parsing to vector search.

Key capabilities:

- **Data ingestion** - parse, chunk, and embed documents from on-premises file shares with customizable pipeline settings.
- **Collections** - organize vector data into logical groupings with independent lifecycle and per-collection RBAC.
- **Multiple search types** — choose from hybrid, vector, text, and hybrid multimodal to match your query needs.
- **Developer portal** - configure ingestion settings, tune search parameters, and test queries through a local web interface.

Access is controlled through Azure RBAC to prevent unauthorized access to ingested data.

For more information, see [Collections overview](collections-overview.md) and [Search types](search-types.md).

### Local chat experience

Agentic Retrieval in Foundry Local includes a built-in chat solution that provides a ready-to-use interface for interacting with agents. The chat solution is a static React app served by nginx that communicates with the agents runtime through the Foundry Agents API.

The chat solution provides:

- **Conversation management** - create, rename, delete, and browse conversations in a sidebar.
- **Streaming responses** - real-time assistant responses via Server-Sent Events (SSE).
- **Citations and sources** - view the sources the agent used to generate each response.
- **Authentication** - optional Entra ID integration for user sign-in, with token-based authorization handled by the backend.

The chat solution handles the browser experience only. Model orchestration, tool invocation, token validation, and data scoping are handled by the agents runtime and backend services.

For more information, see [Chat solution in Agentic Retrieval in Foundry Local](chat-experience.md).

## Customer scenarios and use cases

Customers across verticals like manufacturing, financial services, healthcare, government, and defense generate and store valuable data locally. Regulation, latency, business continuity, or the sheer volume of data generated in real time often keep this data outside of the hyperscale cloud. Customers want to use generative AI applications to get insights from this on-premises data.

Agentic Retrieval in Foundry Local supports Q&A capabilities and multistep agentic conversations that allow customers to query on-premises data via AI agents for scenarios like: 

- A government customer wants to derive insights from sensitive on-premises data to enable quicker decision making, summarize large datasets, create training materials, and more.

- A regional bank wants to use data that must remain on-premises due to regulatory constraints for use cases like compliance checks, customer assistance, and personalized sales pitch generation.

- A global manufacturer wants to create factory floor assistants to reduce time to issue resolution and aid troubleshooting, using data that needs to stay local to comply with organization policies.

- A healthcare provider wants to deploy an agent that can reason across multiple clinical documents, using knowledge bases and MCP tools to correlate patient records, lab results, and treatment guidelines.

- An energy company wants to connect agents to multiple external data sources (SCADA systems, maintenance logs, weather data) via MCP servers, without ingesting all data locally.

## Why Agentic Retrieval in Foundry Local?

Use Agentic Retrieval in Foundry Local to:

- **Build intelligent agents** that orchestrate across multiple knowledge sources, tools, and external services by using the built-in MCP server and knowledge base framework.
- **Reduce time to market** by using a turnkey experience that accelerates the development and deployment of AI applications on local data. 
- **Simplify operations** and **end-to-end management** by using an enterprise-quality solution that delivers the same standard of security, compliance, and manageability you expect from Microsoft, including lifecycle and version management of all components and Microsoft Entra integration for Azure RBAC.  
- **Remove the need for separate developer skillsets** by using cloud-consistent developer experiences. 
- Stay on top of this rapidly evolving space with **continuous innovation from Microsoft**, the leader in AI technologies, and continue to focus on delivering business value.

## Key concepts

Review the following key concepts for Agentic Retrieval in Foundry Local:

- **Chunking** splits large documents into smaller, manageable text blocks (chunks).
  - Chunk size: Chunking divides large documents into smaller units, with settings like chunk size (for example, 1000-2000 characters) and chunk overlap (for example, 100-500 characters) controlling their granularity and continuity. Smaller chunks improve retrieval precision but might lose context, while larger chunks ensure comprehensive context at the cost of precision.
  - Chunk overlap: Overlapping chunks maintain context across boundaries but increase storage and computation requirements.
  
  Optimal chunk settings depend on the use case, balancing accuracy, efficiency, and performance.

- **Data ingestion** is process of importing and preparing external content, such as documents or images, to be used for retrieval. This includes preprocessing steps like cleaning, formatting, and organizing data.

- **Embedding models** transform text, images, or other data into dense numerical vectors (embeddings) that capture semantic meaning. These vectors represent relationships between inputs, allowing for similarity comparisons and clustering.

- **Inferencing** refers to the process of using a trained model to generate predictions or outputs based on new input data. In language models, inferencing involves tasks like completing text, answering questions, or generating summaries.

- **Language models** are AI systems trained to understand, generate, and manipulate human language. They predict text based on input, enabling tasks like text generation, translation, summarization, and question answering. Agentic Retrieval in Foundry Local supports two language model endpoint options. The recommended option is a Foundry Local on Azure Local endpoint. This option runs on the same Azure Arc-connected cluster as the extension. You can also use an external Bring Your Own Model (BYOM) endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry.

- **Model parameters** control how the language model generates text, such as the creativity, diversity, and focus of responses. Common parameters include Temperature, and Top-p. Model parameters don't affect which documents are retrieved, only how the model generates its response. For more information, see [Search type parameters in Agentic Retrieval in Foundry Local](search-types.md#search-type-parameters).

- **Query** is the input provided to a language model to elicit a response or perform a specific task. It can be a question, a prompt, or a set of instructions, depending on the use case.

- **Retrieval Augmented Generation (RAG)** combines a retrieval system with a generative language model to produce responses enriched by external knowledge. It retrieves relevant context from a database or document store to augment the model's generation capabilities, ensuring accurate and up-to-date information.

- **Search parameters** are settings that control how Agentic Retrieval in Foundry Local retrieves, filters, and ranks documents from your indexed data before passing them to the language model. These parameters help you fine-tune the relevance, precision, and scope of the information used to answer user queries. For more information, see [Search type parameters in Agentic Retrieval in Foundry Local](search-types.md#search-type-parameters).

- **Search type**: A search type is the method Agentic Retrieval in Foundry Local uses to find and rank information from your indexed data. It determines how the system retrieves relevant content to answer user questions, such as by matching keywords, using semantic similarity, or combining multiple approaches. Agentic Retrieval in Foundry Local supports several search methods for retrieving information, including full text search, hybrid search, hybrid multimodal search, and vector search. For more information, see [Types of search in Agentic Retrieval in Foundry Local](search-types.md).

- **System prompt** are predefined instructions or messages provided to a language model at the start of a conversation or task to influence its behavior. These prompts define the model's role, tone, or task-specific context. For example, "You're a helpful assistant" or "Provide concise technical explanations." By shaping the initial context, system prompts ensure that the model generates responses aligned with the desired objective or persona.

- **Vector database** is a specialized database to store vector embeddings. It's designed to handle high-dimensional vectors and enables fast and scalable similarity searches.

- **Vectorization** means transforming text into numerical representations, or embeddings, using an embedding model such as Sentence Transformers. These embeddings capture the semantic meaning of text, enabling efficient and accurate comparisons.

- **Agent** is an AI assistant configured with instructions, a model endpoint, and optionally a knowledge base. Agents process user queries through multi-turn conversations, invoking tools and knowledge sources as needed.

- **Knowledge Base** is a grouping of knowledge sources assigned to an agent. When the agent processes a query, it can access all knowledge sources in its knowledge base.

- **Knowledge Source** is a self-contained registration of an MCP server connection. Each knowledge source carries its own connection details (URL, auth type). Two kinds: `remote_mcp` for external MCP servers, and `indexed_sources_mcp` for the built-in MCP server with a specific indexed source reference (e.g., a collection name).

- **Collection** is a logical grouping of ingested vector data. Each collection maps to Milvus vector collections and Postgres tables, and can be independently created, queried, and deleted.

- **MCP (Model Context Protocol)** is an open protocol for connecting AI agents to external tools and data sources. Agentic Retrieval in Foundry Local includes a built-in MCP server with 6 search tools and can also connect to external MCP servers.

- **Thread** is a conversation session between a user and an agent. Threads contain ordered messages and are scoped to a single user.

- **Run** is an execution of an agent against a thread. The agent reads the thread's messages, invokes tools, and generates a response. Runs support streaming via Server-Sent Events (SSE).

- **Deployment mode** determines which layers are deployed. Options: `combined` (default, full platform), `agentic` (agents only, no local data ingestion), `knowledge` (data ingestion and RAG only, no agents).

## Compare with AI services in Azure

Agentic Retrieval in Foundry Local runs on customer infrastructure outside the public cloud, so customers can search their on-premises data by using Retrieval Augmented Generation (RAG). The data plane, including all customer data and the language model, is hosted locally.

In contrast, AI services in Azure such as Azure AI Search and Microsoft Foundry also provide RAG capabilities but are hosted in hyperscale cloud regions. Customers need to bring their data and applications to Azure infrastructure.

Agentic Retrieval in Foundry Local provides local developer UI experiences that align to Foundry experiences.

## Data on-premises versus cloud

Agentic Retrieval in Foundry Local sends only system metadata and organizational identifiable information like subscription ID and cluster names to Microsoft. All customer content, including ingested documents, embeddings, agent configurations, and conversation threads, always stays in the on-premises infrastructure within the network boundaries defined by customers.

## User roles

The Agentic Retrieval in Foundry Local solution has four distinct user roles:

- **Lifecycle management of the extension**: Users manage the lifecycle of the Agentic Retrieval in Foundry Local Arc extension. This role includes tasks such as setting up the necessary infrastructure, deploying the extension, performing updates, monitoring its performance, and handling its eventual deletion. Typically, these responsibilities go to an IT administrator with access to the underlying Azure Local and Azure Kubernetes (AKS) on Azure Local infrastructure.
- **Development and evaluation of agents and chat endpoints**: Users configure agents, knowledge bases, and knowledge sources; provide the data source; customize the RAG pipeline settings; provide custom system prompts; evaluate, monitor, and update the solution. Typically, these responsibilities go to a prompt engineer or an AI application developer. Requires the `EdgeRAGDeveloper` Entra ID role.
- **Consuming the endpoint to query the on-premises data**: Users integrate the chat endpoint into line-of-business applications and use a chat interface, custom, or the one provided out-of-the-box, to query on-premises data.
- **Agentic Layer administration**: Users configure and manage knowledge bases and knowledge sources by using the Knowledge Base Manager API. This role includes registering MCP servers as knowledge sources, updating the default knowledge base, and linking knowledge sources to it. Requires the `EdgeRAGDeveloper` Entra ID role.

## Related content

- [What you need for Agentic Retrieval in Foundry Local](requirements.md)
- [Arc Jumpstart](https://jumpstart.azure.com/)
