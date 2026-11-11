---
title: What's New in Agents and Tools with Foundry Local
description: Learn about the latest features and announcements from the past few months.
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/08/2026
ms.subservice: edge-rag
ai-usage: ai-generated
ms.custom:
  - build-2025
# Customer intent: As an IT administrator or technical decision maker, I want to understand recent updates to Agents and Tools with Foundry Local so that I can plan and manage my deployment.
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
---

# What's new in Agents and Tools with Foundry Local

This article lists recent features and improvements in Agents and Tools with Foundry Local (formerly Edge RAG enabled by Azure Arc).

## November 2026

Agents and Tools with Foundry Local is generally available. This release also introduces Agent Service in preview and retires the agent APIs from earlier previews.

### Generally available platform

The generally available platform includes document ingestion and retrieval, named knowledge sources, collections, Indexed Sources MCP, and customer-visible monitoring.

SQL knowledge sources and disconnected operations are also in preview. Microsoft 365 Local is in limited preview for approved customers. To request access, complete the [preview registration form](https://aka.ms/AgenticRetrievalM365LocalPuP).

### Agent Service (preview)

This release replaces the threads, messages, and runs APIs from earlier previews with Agent Service. Agent Service uses the same API shapes as [Microsoft Foundry Agent Service](/azure/foundry/agents/overview), so your Foundry knowledge and client code carry over. For an introduction, see [Agent Service overview](agentic-overview.md).

#### Agents with versions
Create agents with the Agents API. Every change creates a new version that you can't change later. Choose which version answers users, roll back, and turn agents on or off. See [Create and manage agents](create-manage-agents.md).

#### Responses and conversations APIs
Run agents with the OpenAI-compatible Responses API, and keep conversation history across turns with the Conversations API. Answers can stream and include citations. You can use the OpenAI SDK. See [Run agents with the Responses API](run-agents-responses-api.md).

#### Model connections
Connect agents to a model in Foundry Local on Azure Local, in Microsoft Foundry, or with another OpenAI-compatible endpoint. Use an API key or the extension's managed identity.

#### Tools for agents
Give agents MCP servers (with an API key, Microsoft Entra ID, or no sign-in), Microsoft Foundry toolboxes, and your knowledge bases as tools. See [Add tools to an agent](add-agent-tools.md).

#### Knowledge bases and knowledge sources
Manage knowledge bases with the Knowledge Bases API (`2026-08-01-preview`), and knowledge sources with the updated Knowledge Sources API at `/knowledgesources`. See [Manage knowledge bases](knowledge-bases-guide.md).

#### SQL knowledge sources (preview)
Connect an approved read-only set of SQL Server, PostgreSQL, or MySQL tables and columns. See [Connect a SQL database as a knowledge source](connect-sql-database.md).

#### Microsoft 365 Local (limited preview)
Approved customers can connect on-premises SharePoint Server and Exchange Server as Agentic Retrieval knowledge sources that enforce each signed-in user's permissions. Agents use these sources through knowledge bases. Microsoft 365 Local is supported only with Agentic Retrieval. See [Prepare infrastructure for Microsoft 365 Local](prepare-microsoft-365-local.md) and [Connect Microsoft 365 Local](connect-microsoft-365-local.md).

#### Azure role-based access
Control who can manage and run agents with Azure roles on the extension resource, such as **Reader**, **Cognitive Services Contributor**, and **Cognitive Services OpenAI User**. Running agents and using the local chat UI need an Azure role. The `EdgeRAGDeveloper` and `EdgeRAGEndUser` app roles still work as a fallback for most other operations. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

#### Agent APIs from earlier previews retired
Threads, messages, runs, and the automatic default agent are retired. Their routes return `410 Gone`. To move your apps, see [Agent APIs from earlier previews](agentic-overview.md#agent-apis-from-earlier-previews).

## July 2026

This month introduces improvements for ingestion, agentic conversation quality, security posture, and deployment resilience.

### Release of extension version `0.9.5`

This release focuses on ingestion scale and reliability, agentic memory management for long conversations, security hardening, and a smoother, more secure deployment experience.

#### Faster, more reliable ingestion
Document parsing now scales horizontally across multiple workers with automatic retries, which reduces failures for large and complex documents. The process reports unsupported file formats as skipped instead of silently dropping them. Parsing reports coverage to help you detect missing content. You can cancel ingestion runs cleanly, and the process handles duplicate or stuck-job edge cases without failing.

#### Agentic memory for long conversations
Automatic context and memory compaction keeps long, tool-heavy agentic conversations within the model's context window, using a strategy tuned for small local models. The full conversation history is preserved while the active context is compacted automatically, so multistep conversations stay reliable.

#### Security hardening
Ongoing security hardening includes dependency and vulnerability patching, tighter handling of secrets and tokens so they don't leak into logs or telemetry, and cleaner credential handling during deployment.

#### More secure networking and deployment
For secure network segmentation, the cluster needs a CNI that enforces Kubernetes NetworkPolicy (Calico recommended), and the installer checks for this requirement. At the time of this release, the extension shipped its own ingress controller. Current releases use Envoy Gateway by default and support bring-your-own ingress. Deployment is smoother and more resilient, with GPU/A100 support and automatic prerequisite setup and cleanup.

#### Recommended 32K context window
Configure the Foundry Local model with a 32K token window to improve retrieval and tool-use quality.

#### Agentic Retrieval improvements
Connect multiple knowledge sources to a single knowledge base and use more flexible Chat API inputs with clearer inference error reporting.

#### Improved disconnected operations
This release improves support for disconnected deployments, including SharePoint integration and simplified extension packaging. For more information, see [Download and import the Agents and Tools with Foundry Local expansion pack](disconnected-operations/prepare-disconnected.md).

## June 2026

> [!NOTE]
> The following notes describe the June 2026 preview release (`0.9.3`). Current releases support Foundry Local on the cluster, replace `layerSelection` with `indexedSourcesEnabled`, retire the V1 `Agents Runtime` and `Knowledge Base Manager` APIs (`410 Gone`), and restrict the deprecated Inference API to the developer portal.

### Agents and Tools with Foundry Local (preview)

This release renames Edge RAG enabled by Azure Arc to Agents and Tools with Foundry Local, a retrieval-augmented generation platform at the edge. It adds AI agent orchestration, which is now part of Agent Service. You can build agents that manage multistep conversations, invoke external tools, and ground responses in your on-premises data.

#### Agent orchestration and retrieval

Build multithreaded AI agents that orchestrate across knowledge sources and external tools:

- **Agents Runtime** — Create threads, send messages, execute runs with streaming (SSE). OpenAI Assistants-compatible API.
- **Knowledge Base Manager** — Manages the default knowledge base (GET, PATCH, PUT). Knowledge bases group knowledge sources and define what data the system can access.
- **Knowledge Sources** — Register MCP server connections as self-contained knowledge sources (two kinds: `remote_mcp` and `indexed_sources_mcp`), and organize them into knowledge bases.
- **Built-in MCP Server** — Six search tools (hybrid, vector, text, image, multimodal, list collections) exposed over Model Context Protocol for any MCP-compatible client.
- **Collections** — Multiple named collections for vector data with per-collection RBAC, replacing the single-index model.

#### Independent deployment modes

Deploy exactly what you need with the new `layerSelection` parameter:

- `combined` (default) — full platform
- `agentic` — agents without local data ingestion (no GPUs needed)
- `knowledge` — at the time, data ingestion and RAG without agent orchestration. The current chart also retains the core Agent Service workloads in this mode.

#### Foundry Local or BYOM endpoints required

Phi-3.5 and Mistral-7B are no longer bundled with the extension. All deployments now require a language model endpoint. Use a Foundry Local endpoint on Azure Local as the recommended option, or use an external BYOM endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry.

#### GPU count reduced

GPU requirement reduced from 4 to **2** (embedding, image). Docling now runs on CPU. The LLM runs externally through BYOM.

#### Seven REST API references published

Complete API documentation for: Agents Runtime, Knowledge Base Manager, Knowledge Sources, Collections, MCP Server, Ingestion, and Inference.

For more information, see the following articles:

- [Agent Service overview](agentic-overview.md) — Architecture and concepts
- [Query Your Data Quickstart](quickstart-edge-rag.md) — End-to-end tutorial
- [Collections Overview](collections-overview.md) — Collection architecture and RBAC
- [MCP Server Overview](mcp-server-overview.md) — Built-in and external MCP servers

## February 2026

### Release of extension version `0.8.5`

This release focuses on reliability, stability, and security improvements for the extension. These updates make the system more reliable and predictable, with better error handling and enhanced security protections.

#### Enhanced search reliability
Search operations are more dependable with improved deep search accuracy for complex queries. You get clearer error messages that tell you whether the system is unavailable or the knowledge base is empty, so you know exactly what's happening and can take the right action.

#### More reliable data ingestion
Data ingestion now handles edge cases without failing, including files without extensions and empty folders. The system reports which files it skipped and why, giving you full visibility into the ingestion process. NFS connections validate accessibility, sockets, and permissions upfront, with clear error messages when problems occur.

#### Improved system reliability
Configurable timeouts for document conversion, SharePoint connections, LLM inference calls, and database operations prevent indefinite hangs and resource exhaustion. Operations fail predictably with clear diagnostics, so you can tune settings for your environment.

#### Better troubleshooting
Deployment failures now provide accessible logs, so you can identify root causes faster without escalating to support. This improvement reduces time to resolution and helps you get Agents and Tools with Foundry Local running smoothly.

#### Critical security updates
This release addresses two important vulnerabilities: a critical Next.js Denial-of-Service vulnerability (GHSA-5j59-xgg2-r9c4) and a high-severity Langchain XXE vulnerability that could allow unauthorized file access. Dependency updates also resolve version conflicts and improve overall system stability.

## November 2025

### Release of extension version `0.8.2`

This release introduces several new features, enhancements, and improvements designed to expand capabilities, improve performance, and streamline the user experience.

#### Deep search
Find the most relevant information with the new deep search model. Deep search uses production-ready [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with high-quality RAG inferencing. Deep search now explores and connects data across sources at query time, so you get comprehensive answers without heavy upfront processing. For more information, see [Search types in Agents and Tools with Foundry Local](search-types.md).

#### High-fidelity parsing
Choose between basic text extraction or advanced parsing to capture tables, images, and more. Advanced parsing offers OCR-enabled support for documents, tables, and images. Tailor data ingestion to your needs for more accurate results. For more information, see [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md).

#### Performance and scale
This release improves hybrid-search query performance and ingestion performance for live-streamed images.

#### Advanced search and chat experience

The extension now offers a more capable and flexible search and chat experience, making it easier to find information and interact with your data through new capabilities and interface improvements.

- Use hybrid multimodal search to retrieve images and deliver responses with relevant images. For more information, see [Search types in Agents and Tools with Foundry Local](search-types.md).
- Get markdown-formatted responses that support images and formatted text for responses that are easier to read and interpret.
- Chat directly with the language model, without using your organization’s data as context. Use the model only option to ask general questions, test the model’s capabilities, or get responses that aren’t influenced by your ingested data. Switch between knowledge-based chat and model-only chat to fit your needs. For more information, see [Agentic Retrieval overview](knowledge-layer-overview.md#data-query).

#### Preview support for disconnected scenarios

The extension is supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

#### Related content

For more information about this release, see:

- Blog: [Transforming City Operations: How Villa Park and DataON Deliver Real-Time Decisions and Resilience with Edge RAG](https://techcommunity.microsoft.com/blog/AzureArcBlog/transforming-city-operations-how-villa-park-and-dataon-deliver-real-time-decisio/4470360)


## Related content

- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
