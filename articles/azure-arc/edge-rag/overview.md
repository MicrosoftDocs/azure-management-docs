---
title: Agents and Tools with Foundry Local Overview
description: "Learn about the Azure Arc-enabled Kubernetes extension Agents and Tools with Foundry Local used to search on-premises data with generative AI."
author: cwatson-cat
ms.author: cwatson
ms.topic: overview #Don't change
ms.date: 05/04/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a business decision maker, I want to evaluate the Agents and Tools with Foundry Local solution for on-premises data retrieval, so that I can determine if it aligns with our regulatory compliance and AI application needs while enhancing insights and decision-making processes.
---
# What is Agents and Tools with Foundry Local?

Agents and Tools with Foundry Local is an [Azure Arc-enabled Kubernetes extension](/azure/azure-arc/kubernetes/extensions-release) that provides an **agentic Retrieval-Augmented Generation (RAG) platform** at the edge. It combines a Knowledge Layer (document ingestion, embedding, vector search) with an Agentic Layer (AI agents, knowledge orchestration, MCP server) to deliver intelligent, multistep assistants grounded in your private on-premises data.

Agents and Tools with Foundry Local packages everything necessary to build and deploy AI-powered assistants on local data, including:

- **An Agentic Layer** with AI agent orchestration, knowledge bases, knowledge sources, and a built-in MCP (Model Context Protocol) server.
- **A Knowledge Layer** with a turnkey data ingestion and RAG pipeline that keeps all data local, with Azure role-based access controls (Azure RBAC) to prevent unauthorized access.
- **Bring Your Own Model (BYOM)** — connect any OpenAI-compatible language model endpoint (via FoundryOnArc, KAITO, Azure OpenAI, or similar).
- **Two GPU-accelerated models** for text embedding (BGE-M3) and image embedding (CLIP ViT-L/14) — running locally on two GPUs. Docling (document parser) runs on CPU.
- An out-of-the-box developer portal and agentic chat UI, plus REST APIs for integration into business applications.
- **Independent deployment modes** — deploy the full platform, or just the Agentic Layer or Knowledge Layer separately.

Agents and Tools with Foundry Local can ingest and retrieve relevant images as contextual references alongside text. It's not a visual language model (VLM).

Agents and Tools with Foundry Local is supported and validated on Azure Arc-enabled Kubernetes on Azure Local (formerly Azure Stack HCI) infrastructure and as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

For more information, see [Azure Arc](/azure/azure-arc/overview), [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/), and [Azure Arc extensions](/azure/azure-arc/kubernetes/conceptual-extensions). 

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Customer scenarios and use cases

Customers across verticals like manufacturing, financial services, healthcare, government, and defense generate and store valuable data locally. Regulation, latency, business continuity, or the sheer volume of data generated in real time often keep this data outside of the hyperscale cloud. Customers want to use generative AI applications to get insights from this on-premises data.

Agents and Tools with Foundry Local supports Q&A capabilities and multistep agentic conversations that allow customers to query on-premises data via AI agents for scenarios like: 

- A government customer wants to derive insights from sensitive on-premises data to enable quicker decision making, summarize large datasets, create training materials, and more.

- A regional bank wants to use data that must remain on-premises due to regulatory constraints for use cases like compliance checks, customer assistance, and personalized sales pitch generation.

- A global manufacturer wants to create factory floor assistants to reduce time to issue resolution and aid troubleshooting, using data that needs to stay local to comply with organization policies.

- A healthcare provider wants to deploy an agent that can reason across multiple clinical documents, using knowledge bases and MCP tools to correlate patient records, lab results, and treatment guidelines.

- An energy company wants to connect agents to multiple external data sources (SCADA systems, maintenance logs, weather data) via MCP servers, without ingesting all data locally.

## Why Agents and Tools with Foundry Local?

Use Agents and Tools with Foundry Local to:

- **Build intelligent agents** that orchestrate across multiple knowledge sources, tools, and external services by using the built-in MCP server and knowledge base framework.
- **Reduce time to market** by using a turnkey experience that accelerates the development and deployment of AI applications on local data. 
- **Simplify operations** and **end-to-end management** by using an enterprise-quality solution that delivers the same standard of security, compliance, and manageability you expect from Microsoft, including lifecycle and version management of all components and Microsoft Entra integration for Azure RBAC.  
- **Remove the need for separate developer skillsets** by using cloud-consistent developer experiences. 
- Stay on top of this rapidly evolving space with **continuous innovation from Microsoft**, the leader in AI technologies, and continue to focus on delivering business value.

## Key concepts

Review the following key concepts for Agents and Tools with Foundry Local:

- **Chunking** splits large documents into smaller, manageable text blocks (chunks).
  - Chunk size: Chunking divides large documents into smaller units, with settings like chunk size (for example, 1000-2000 characters) and chunk overlap (for example, 100-500 characters) controlling their granularity and continuity. Smaller chunks improve retrieval precision but might lose context, while larger chunks ensure comprehensive context at the cost of precision.
  - Chunk overlap: Overlapping chunks maintain context across boundaries but increase storage and computation requirements.
  
  Optimal chunk settings depend on the use case, balancing accuracy, efficiency, and performance.

- **Data ingestion** is process of importing and preparing external content, such as documents or images, to be used for retrieval. This includes preprocessing steps like cleaning, formatting, and organizing data.

- **Embedding models** transform text, images, or other data into dense numerical vectors (embeddings) that capture semantic meaning. These vectors represent relationships between inputs, allowing for similarity comparisons and clustering.

- **Inferencing** refers to the process of using a trained model to generate predictions or outputs based on new input data. In language models, inferencing involves tasks like completing text, answering questions, or generating summaries.

- **Language models** are AI systems trained to understand, generate, and manipulate human language. They predict text based on input, enabling tasks like text generation, translation, summarization, and question answering. Agents and Tools with Foundry Local requires you to Bring Your Own Model (BYOM) — an external LLM endpoint compatible with the OpenAI chat completions API. Recommended options include models deployed via FoundryOnArc, KAITO, or Azure OpenAI.

- **Model parameters** control how the language model generates text, such as the creativity, diversity, and focus of responses. Common parameters include Temperature, and Top-p. Model parameters don't affect which documents are retrieved, only how the model generates its response. For more information, see [Search type parameters in Agents and Tools with Foundry Local](search-types.md#search-type-parameters).

- **Query** is the input provided to a language model to elicit a response or perform a specific task. It can be a question, a prompt, or a set of instructions, depending on the use case.

- **Retrieval Augmented Generation (RAG)** combines a retrieval system with a generative language model to produce responses enriched by external knowledge. It retrieves relevant context from a database or document store to augment the model's generation capabilities, ensuring accurate and up-to-date information.

- **Search parameters** are settings that control how Agents and Tools with Foundry Local retrieves, filters, and ranks documents from your indexed data before passing them to the language model. These parameters help you fine-tune the relevance, precision, and scope of the information used to answer user queries. For more information, see [Search type parameters in Agents and Tools with Foundry Local](search-types.md#search-type-parameters).

- **Search type**: A search type is the method Agents and Tools with Foundry Local uses to find and rank information from your indexed data. It determines how the system retrieves relevant content to answer user questions, such as by matching keywords, using semantic similarity, or combining multiple approaches. Agents and Tools with Foundry Local supports several search methods for retrieving information, including deep search, full text search, hybrid search, hybrid multimodal search, and vector search. For more information, see [Types of search in Agents and Tools with Foundry Local](search-types.md).

- **System prompt** are predefined instructions or messages provided to a language model at the start of a conversation or task to influence its behavior. These prompts define the model's role, tone, or task-specific context. For example, "You're a helpful assistant" or "Provide concise technical explanations." By shaping the initial context, system prompts ensure that the model generates responses aligned with the desired objective or persona.

- **Vector database** is a specialized database to store vector embeddings. It's designed to handle high-dimensional vectors and enables fast and scalable similarity searches.

- **Vectorization** means transforming text into numerical representations, or embeddings, using an embedding model such as Sentence Transformers. These embeddings capture the semantic meaning of text, enabling efficient and accurate comparisons.

- **Agent** is an AI assistant configured with instructions, a model endpoint, and optionally a knowledge base. Agents process user queries through multi-turn conversations, invoking tools and knowledge sources as needed.

- **Knowledge Base** is a grouping of knowledge sources assigned to an agent. When the agent processes a query, it can access all knowledge sources in its knowledge base.

- **Knowledge Source** is a self-contained registration of an MCP server connection. Each knowledge source carries its own connection details (URL, auth type). Two kinds: `remote_mcp` for external MCP servers, and `indexed_sources_mcp` for the built-in MCP server with a specific indexed source reference (e.g., a collection name).

- **Collection** is a logical grouping of ingested vector data. Each collection maps to Milvus vector collections and Postgres tables, and can be independently created, queried, and deleted.

- **MCP (Model Context Protocol)** is an open protocol for connecting AI agents to external tools and data sources. Agents and Tools with Foundry Local includes a built-in MCP server with 6 search tools and can also connect to external MCP servers.

- **Thread** is a conversation session between a user and an agent. Threads contain ordered messages and are scoped to a single user.

- **Run** is an execution of an agent against a thread. The agent reads the thread's messages, invokes tools, and generates a response. Runs support streaming via Server-Sent Events (SSE).

- **Deployment mode** determines which layers are deployed. Options: `combined` (default, full platform), `agentic` (agents only, no local data ingestion), `knowledge` (data ingestion and RAG only, no agents).

## Compare with AI services in Azure

Agents and Tools with Foundry Local runs on customer infrastructure outside the public cloud, so customers can search their on-premises data by using Retrieval Augmented Generation (RAG). The data plane, including all customer data and the language model, is hosted locally.

In contrast, AI services in Azure such as Azure AI Search and Microsoft Foundry also provide RAG capabilities but are hosted in hyperscale cloud regions. Customers need to bring their data and applications to Azure infrastructure.

Agents and Tools with Foundry Local provides local developer UI experiences that align to Foundry experiences.

## Data on-premises versus cloud

Agents and Tools with Foundry Local sends only system metadata and organizational identifiable information like subscription ID and cluster names to Microsoft. All customer content, including ingested documents, embeddings, agent configurations, and conversation threads, always stays in the on-premises infrastructure within the network boundaries defined by customers.

## User roles

The Agents and Tools with Foundry Local solution has four distinct user roles:

- **Lifecycle management of the extension**: Users manage the lifecycle of the Agents and Tools with Foundry Local Arc extension. This role includes tasks such as setting up the necessary infrastructure, deploying the extension, performing updates, monitoring its performance, and handling its eventual deletion. Typically, these responsibilities go to an IT administrator with access to the underlying Azure Local and Azure Kubernetes (AKS) on Azure Local infrastructure.
- **Development and evaluation of agents and chat endpoints**: Users configure agents, knowledge bases, and knowledge sources; provide the data source; customize the RAG pipeline settings; provide custom system prompts; evaluate, monitor, and update the solution. Typically, these responsibilities go to a prompt engineer or an AI application developer. Requires the `EdgeRAGDeveloper` Entra ID role.
- **Consuming the endpoint to query the on-premises data**: Users integrate the chat endpoint into line-of-business applications and use a chat interface, custom, or the one provided out-of-the-box, to query on-premises data.
- **Agentic Layer administration**: Users configure and manage knowledge bases and knowledge sources by using the Knowledge Base Manager API. This role includes registering MCP servers as knowledge sources, updating the default knowledge base, and linking knowledge sources to it. Requires the `EdgeRAGDeveloper` Entra ID role.

## Related content

- [What you need for Agents and Tools with Foundry Local](requirements.md)
- [Arc Jumpstart](https://jumpstart.azure.com/)
