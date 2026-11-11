---
title: Release Notes for Agents and Tools with Foundry Local
ms.reviewer: cwatson
description: "Discover the latest updates and features for Agents and Tools with Foundry Local."
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ms.topic: "release-notes"  
ms.date: 11/08/2026
ai-usage: ai-generated

#customer intent: As an IT admin, I want to understand the new features in the latest Agents and Tools with Foundry Local release so that I can plan updates for my organization.

---

# Release notes for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local helps you build and run agents at the edge with Agent Service, Agentic Retrieval, the local chat UI, and tools that connect agents to data and actions. This article lists new features, improvements, and important changes in each release. Use these notes to plan, deploy, and manage your agent solutions with Foundry Local.

## November 2026

Agents and Tools with Foundry Local is generally available on Azure Arc-enabled Kubernetes on Azure Local. The generally available platform includes document ingestion and retrieval, named knowledge sources, collections, Indexed Sources MCP, and customer-visible monitoring.

Agent Service and managed knowledge-base connections are in preview. SQL knowledge sources and disconnected operations are also in preview. Microsoft 365 Local is in limited preview for approved customers and is supported only with Agentic Retrieval. To request access, complete the [preview registration form](https://aka.ms/AgenticRetrievalM365LocalPuP).

### Agent Service (preview)

| Feature | Description |
|---|---|
| **Agents API** | Create agents and agent versions. Pin a version, roll back, and turn agents on or off. See [Create and manage agents](create-manage-agents.md). |
| **Responses and Conversations APIs** | OpenAI-compatible APIs to run agents, stream answers with citations, and keep multiturn history. See [Run agents with the Responses API](run-agents-responses-api.md). |
| **Model connections** | Connect to Foundry Local on Azure Local, Microsoft Foundry, or an OpenAI-compatible endpoint, with an API key or managed identity. |
| **MCP tools and Foundry toolboxes** | Add MCP servers and Microsoft Foundry toolboxes to agents as tools. See [Add tools to an agent](add-agent-tools.md). |
| **Knowledge bases as tools** | Each knowledge base gets an MCP connection that agents can use as a tool. See [Manage knowledge bases](knowledge-bases-guide.md). |
| **Local chat UI** | The local chat UI lists your enabled agents and uses the Conversations and Responses APIs. See [How chat works](chat-experience.md). |
| **Azure role-based access** | Control access with Azure roles on the extension resource, such as **Reader**, **Cognitive Services Contributor**, and **Cognitive Services OpenAI User**. The `EdgeRAGDeveloper` and `EdgeRAGEndUser` app roles still work as a fallback for most operations. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles). |

### Knowledge and retrieval

| Feature | Description |
|---|---|
| **Current knowledge APIs** | Manage named knowledge sources through `/knowledgesources` and knowledge bases through `/knowledgebases?api-version=2026-08-01-preview`. |
| **SQL knowledge sources (preview)** | Expose an approved read-only set of SQL Server, PostgreSQL, or MySQL tables and columns through a managed SQL MCP workload. |
| **Microsoft 365 Local (limited preview)** | Approved customers can connect on-premises SharePoint Server and Exchange Server as Agentic Retrieval knowledge sources with per-user permissions. Agents use these sources through knowledge bases. Microsoft 365 Local is supported only with Agentic Retrieval. See [Prepare infrastructure for Microsoft 365 Local](prepare-microsoft-365-local.md) and [Connect Microsoft 365 Local](connect-microsoft-365-local.md). |
| **Exchange citation links (limited preview)** | Set the optional `exchange_parameters.owa_url` on an Exchange knowledge source to add Outlook on the web links to email and calendar citations. Releases before this one reject the field. See [Connect Microsoft 365 Local](connect-microsoft-365-local.md#create-an-exchange-knowledge-source). |
| **Indexed Sources MCP** | Search allowed collections through the built-in MCP endpoint at `/mcp`. |

### Breaking changes

| Change | Description |
|---|---|
| **Agent APIs from earlier previews retired** | `/threads`, `/knowledge-bases`, and `POST /agents` with the old request body return `410 Gone`. The earlier Knowledge Sources API, which used generated IDs, is replaced by the current API. See [Agent APIs from earlier previews](agentic-overview.md#agent-apis-from-earlier-previews). |
| **Azure roles required to run agents** | The Responses and Conversations APIs, including the local chat UI, need an Azure role such as **Cognitive Services OpenAI User** on the extension resource. The `EdgeRAGEndUser` app role alone no longer allows users to run agents. Also assign **Reader** on the extension resource to the extension's managed identity. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles). |
| **Knowledge Sources API path** | The Knowledge Sources API path is `/knowledgesources`. URLs from earlier builds that end in `/knowledgesources/v2` no longer work, and there's no redirect. Update your clients. See the [Knowledge Sources REST API reference](reference/knowledge-sources-api-reference.md). |

## July 2026

**Extension version**: `0.9.5` [Preview]

This release focuses on ingestion scale and reliability, agentic memory management for long conversations, security hardening, and a smoother, more secure deployment experience. For current limitations and workarounds, see [Known issues in Agents and Tools with Foundry Local](known-issues.md).

### Ingestion

| Feature | Description |
|---|---|
| **Faster, more reliable large-scale ingestion** | Document parsing scales horizontally across multiple workers, with automatic retries, so large and complex documents are far less likely to fail. |
| **Clearer visibility into parsing** | Files in unsupported formats are reported as *skipped* instead of being silently dropped, and parsing reports coverage so any missing or dropped content is detectable. |
| **More reliable job handling** | Ingestion runs can be cancelled cleanly, and duplicate or stuck-job edge cases are handled without failing. |

### Agentic memory and long conversations

| Feature | Description |
|---|---|
| **Automatic context and memory compaction** | Keeps long, tool-heavy agentic conversations within the model's context window, using a strategy tuned for small local models. The full conversation history is preserved while the active context is compacted automatically, so multistep conversations stay reliable. |

### Security

| Feature | Description |
|---|---|
| **Ongoing security hardening** | Dependency and vulnerability patching, tighter handling of secrets and tokens so they don't leak into logs or telemetry, and cleaner credential handling during deployment. |

### Networking and deployment

| Feature | Description |
|---|---|
| **NetworkPolicy-capable CNI required** | For secure network segmentation, the cluster needs a CNI that enforces Kubernetes NetworkPolicy (Calico recommended). The installer checks for this requirement and provides guidance if it's missing. |
| **Bring-your-own ingress controller wasn't supported** | At the time of this release, the extension shipped and managed its own `ingress-nginx` controller. Current releases use Envoy Gateway by default and support bring-your-own ingress through `networking.mode=none`. |
| **Smoother, more resilient deployment** | Includes GPU/A100 support and automatic prerequisite setup and cleanup. |

### Foundry Local model configuration

| Feature | Description |
|---|---|
| **Recommended context window: 32K** | Configure the Foundry Local model with a 32K token window for the best results with retrieval and tool use. |

### Agentic Retrieval

| Feature | Description |
|---|---|
| **Multiple knowledge sources per knowledge base** | Connect multiple knowledge sources to a single knowledge base. |
| **More flexible Chat API inputs** | Accepts more flexible inputs, with clearer and more accurate inference error reporting. |

### Disconnected operations on Azure Local

| Feature | Description |
|---|---|
| **Improved disconnected support** | Improved support for disconnected deployments, including SharePoint integration and simplified extension packaging. For more information, see [Download and import the Agents and Tools with Foundry Local expansion pack](disconnected-operations/prepare-disconnected.md). |

## June 2026

**Extension version**: `0.9.3` [Preview]

> [!NOTE]
> The following notes describe the June 2026 preview release (`0.9.3`). Current releases support Foundry Local on the cluster, replace `layerSelection` with `indexedSourcesEnabled`, retire the V1 `Agents Runtime` and `Knowledge Base Manager` APIs (`410 Gone`), and restrict the deprecated Inference API to the developer portal.

### Changes

| Change | Details |
|---|---|
| **Product renamed** | Edge RAG enabled by Azure Arc is now Agents and Tools with Foundry Local. The Azure extension type remains `microsoft.arc.rag`. |
| **SLM models removed** | Phi-3.5 and Mistral-7B models are no longer bundled. Bring Your Own Model (BYOM) is the only model path. You must provide an external LLM endpoint. |
| **GPU count reduced** | 4 → **2 GPUs** required. Text embedding (BGE-M3) and image embedding (CLIP ViT-L/14) require the GPUs. Docling (document parser) now runs on CPU. The LLM runs externally through BYOM. |
| **BYOM is now mandatory** | The model endpoint step is no longer optional. All deployments require an OpenAI-compatible LLM endpoint. |

### New features

| Feature | Description |
|---|---|
| **Independent layer deployment** | New `layerSelection` ARM parameter: `combined` (default), `agentic`, `knowledge`. Deploy layers independently. |
| **Agents Runtime API** | Full agent orchestration with threads, messages, runs, and streaming (SSE). OpenAI Assistants-compatible API. Port 8080. |
| **Knowledge Base Manager API** | Manages the default knowledge base (GET, PATCH, PUT). Port 8080. |
| **Knowledge Sources API** | Register MCP server connections as self-contained knowledge sources (two kinds: `remote_mcp` and `indexed_sources_mcp`). Port 3005. |
| **Collections API** | Manage multiple collections for vector data with per-collection RBAC. Port 3002. |
| **MCP server** | Built-in MCP server with 6 search tools over Model Context Protocol. Port 8080. |
| **Ingestion API** | Public REST API for programmatic document ingestion. Port 8000. |
| **Inference API** | Public REST API for RAG chat completions and model-only queries. Port 3001. |
| **Local chat UI** | New agentic chat interface with multi-turn conversations, thread history, and streaming. |

## February 2026

**Extension version**: `0.8.5` [Preview]

This release includes bug fixes, reliability improvements, and critical security updates.

### Search and query

This release improves the accuracy and reliability of search operations.

- Deep search returns accurate results for complex queries, including time-based and count-based questions.
- Timeouts for LLM inference calls, Vector DB queries, and database operations prevent resource exhaustion.
- Error messages clearly show whether the system is unavailable or the knowledge base is empty.

### Data ingestion

Data ingestion is more reliable and handles edge cases without failure.

- The system skips files without extensions and reports which files it skipped and why.
- You can now ingest to empty folders or folders with mixed content without failures.
- NFS connection validation checks accessibility, socket connections, and permissions, and provides clear error messages when problems occur.
- You can configure timeouts for document conversion, SharePoint connections, and file processing to prevent indefinite hangs.

### Diagnostics and observability

Improved log access makes troubleshooting deployment issues faster. You can access logs during and after deployment failures, so you can identify root causes faster.

### Security updates

This release addresses critical and high-severity vulnerabilities.

- This release fixes a critical Next.js denial-of-service vulnerability (GHSA-5j59-xgg2-r9c4) through an upgrade to version 14.2.35.
- This release resolves a high-severity Langchain XXE vulnerability that could allow unauthorized file access during document parsing.
- Dependency updates resolve version conflicts and improve system stability.

## November 2025

**Extension version**: `0.8.2` [Preview]

The following table summarizes the updates included in this release.

| Capability                | Description |
|---------------------------|---------|
| Data ingestion  | - Improved parsing for documents, including tables and charts.<br>- Added new ingestion type options for high-fidelity document processing.  <br></br>For more information, see [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md)|
|Data query and chat   | - Added deep search model search type for production-ready [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with high-quality RAG inferencing. <br> - Added hybrid multimodal search with image retrieval and image-rich outputs.<br>- Added support for responses in markdown for improved readability.<br>- Added a model-only chat option for direct model chat. <br><br> For more information, see [Agentic Retrieval overview](knowledge-layer-overview.md) and [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md).|
|Performance enhancements | Improved hybrid-search query performance and ingestion performance for live-streamed images. |
|Preview support for disconnected scenarios|Agents and Tools with Foundry Local supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).|

For more information, see [What's new in Agents and Tools with Foundry Local](whats-new.md).

## July 2025

**Extension version**: `0.1.5` [Preview]

- Resolved authentication problems for endpoints that Microsoft Foundry created for bring your own model (BYOM) deployments.
- Temporarily disabled chat history to improve performance. Each question is answered based only on retrieved content.
- Improved security.

## May 2025

**Extension version**: `0.1.3` [Preview]

Agents and Tools with Foundry Local, then named Edge RAG enabled by Azure Arc, is available in preview.

See the related blog posts:

- [Transforming On-Premises Data with RAG Capabilities on Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/transforming-on-premises-data-with-rag-capabilities-on-azure-local/4415217)
- [Empowering the Physical World with AI - Unlocking AI at the Edge with Azure Arc](https://techcommunity.microsoft.com/blog/azurearcblog/empowering-the-physical-world-with-ai/4415204)
- [Unlocking AI Apps Across Boundaries with Azure](https://techcommunity.microsoft.com/blog/AzureArcBlog/unlocking-ai-apps-across-boundaries-with-azure/4410457)

## Related content

- [What you need for Agents and Tools with Foundry Local](requirements.md)
- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
