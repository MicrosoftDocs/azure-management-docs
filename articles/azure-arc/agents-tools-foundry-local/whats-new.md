---
title: What's New in Agentic Retrieval in Foundry Local
description: Learn about the latest features and announcements from the past few months.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 07/07/2026
ms.subservice: edge-rag
ai-usage: ai-generated
ms.custom:
  - build-2025
# Customer intent: As an IT administrator or technical decision maker, I want to stay updated on the latest features and improvements for Agentic Retrieval in Foundry Local so that I can effectively plan, deploy, and manage the Agentic Retrieval in Foundry Local solution in my organization.
---

# What's new in Agentic Retrieval in Foundry Local

This article lists recent features and improvements in Agentic Retrieval (formerly Edge RAG enabled by Azure Arc).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## July 2026

This month introduces improvements for ingestion, agentic conversation quality, security posture, and deployment resilience.

### Release of extension version `0.9.5`

This release focuses on ingestion scale and reliability, agentic memory management for long conversations, security hardening, and a smoother, more secure deployment experience.

**Faster, more reliable ingestion**  
Document parsing now scales horizontally across multiple workers with automatic retries, which reduces failures for large and complex documents. The process reports unsupported file formats as skipped instead of silently dropping them. Parsing reports coverage to help you detect missing content. You can cancel ingestion runs cleanly, and the process handles duplicate or stuck-job edge cases gracefully.

**Agentic memory for long conversations**  
Automatic context and memory compaction keeps long, tool-heavy agentic conversations within the model's context window, using a strategy tuned for small local models. The full conversation history is preserved while the active context is compacted automatically, so multistep conversations stay reliable.

**Security hardening**  
Ongoing security hardening includes dependency and vulnerability patching, tighter handling of secrets and tokens so they don't leak into logs or telemetry, and cleaner credential handling during deployment.

**More secure networking and deployment**  
For secure network segmentation, the cluster needs a CNI that enforces Kubernetes NetworkPolicy (Calico recommended), and the installer checks for this requirement. Agentic Retrieval ships and manages its own ingress controller, so bring-your-own ingress isn't supported. Deployment is smoother and more resilient, with GPU/A100 support and automatic prerequisite setup and cleanup.

**Recommended 32K context window**  
Configure the Foundry Local model with a 32K token window to improve retrieval and tool-use quality.

**Agentic retrieval and evaluation improvements**  
Connect multiple knowledge sources to a single knowledge base, use more flexible Chat API inputs with clearer inference error reporting, and measure answer quality across conversation turns with multistep evaluation.

**Improved disconnected operations**  
This release improves support for disconnected and air-gapped deployments, including SharePoint integration and simplified extension packaging. For more information, see [Download and import the Agentic Retrieval expansion pack](disconnected-operations/prepare-disconnected.md).

## June 2026

### Agentic Retrieval (preview)

This release renames Edge RAG enabled by Azure Arc to Agentic Retrieval, a retrieval-augmented generation platform at the edge, and adds an agentic layer for AI agent orchestration. You can build agents that manage multistep conversations, invoke external tools, and ground responses in your on-premises data.

**Agentic layer**

Build multithreaded AI agents that orchestrate across knowledge sources and external tools:

- **Agents Runtime** — Create threads, send messages, execute runs with streaming (SSE). OpenAI Assistants-compatible API.
- **Knowledge Base Manager** — Manages the default knowledge base (GET, PATCH, PUT). Knowledge bases group knowledge sources and define what data the system can access.
- **Knowledge Sources** — Register MCP server connections as self-contained knowledge sources (two kinds: `remote_mcp` and `indexed_sources_mcp`), and organize them into knowledge bases.
- **Built-in MCP Server** — Six search tools (hybrid, vector, text, image, multimodal, list collections) exposed over Model Context Protocol for any MCP-compatible client.
- **Collections** — Multiple named collections for vector data with per-collection RBAC, replacing the single-index model.

**Independent Deployment Modes**

Deploy exactly what you need with the new `layerSelection` parameter:

- `combined` (default) — full platform
- `agentic` — agents without local data ingestion (no GPUs needed)
- `knowledge` — data ingestion and RAG without agent orchestration

**Foundry Local or BYOM endpoints required**

Phi-3.5 and Mistral-7B are no longer bundled with the extension. All deployments now require a language model endpoint. Use a Foundry Local endpoint on Azure Local as the recommended option, or use an external BYOM endpoint that supports an OpenAI-compatible chat completions API, such as one deployed in Microsoft Foundry.

**GPU Count Reduced**

GPU requirement reduced from 4 to **2** (embedding, image). Docling now runs on CPU. The LLM runs externally via BYOM.

**Seven REST API references published**

Complete API documentation for: Agents Runtime, Knowledge Base Manager, Knowledge Sources, Collections, MCP Server, Ingestion, and Inference.

For more information, see the following articles:

- [Agentic Layer Overview](agentic-overview.md) — Architecture and concepts
- [Query Your Data Quickstart](quickstart-edge-rag.md) — End-to-end tutorial
- [Collections Overview](collections-overview.md) — Collection architecture and RBAC
- [MCP Server Overview](mcp-server-overview.md) — Built-in and external MCP servers

## February 2026

### Release of extension version `0.8.5`

This release focuses on reliability, stability, and security improvements for Edge RAG. These updates make the system more robust and predictable, with better error handling and enhanced security protections.

**Enhanced search reliability**  
Search operations are more dependable with improved deep search accuracy for complex queries. You get clearer error messages that tell you whether the system is unavailable or the knowledge base is empty, so you know exactly what's happening and can take the right action.

**More robust data ingestion**  
Data ingestion now handles edge cases gracefully, including files without extensions and empty folders. The system reports which files it skipped and why, giving you full visibility into the ingestion process. NFS connections validate accessibility, sockets, and permissions upfront, with clear error messages when problems occur.

**Improved system reliability**  
Configurable timeouts for document conversion, SharePoint connections, LLM inference calls, and database operations prevent indefinite hangs and resource exhaustion. Operations fail predictably with clear diagnostics, so you can tune settings for your environment.

**Better troubleshooting**  
Deployment failures now provide accessible logs, so you can identify root causes faster without escalating to support. This improvement reduces time to resolution and helps you get Agentic Retrieval running smoothly.

**Critical security updates**  
This release addresses two important vulnerabilities: a critical Next.js Denial-of-Service vulnerability (GHSA-5j59-xgg2-r9c4) and a high-severity Langchain XXE vulnerability that could allow unauthorized file access. Dependency updates also resolve version conflicts and improve overall system stability.

## November 2025

### Release of extension version `0.8.2`

This release of Edge RAG introduces several new features, enhancements, and improvements designed to expand capabilities, improve performance, and streamline the user experience.

**Deep search**  
Find the most relevant information with the new deep search model. Deep search uses production-class [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with industry-leading RAG inferencing quality. Edge RAG now explores and connects data across sources at query time, so you get comprehensive answers without heavy upfront processing. For more information, see [Search types in Agentic Retrieval](search-types.md).

**High-fidelity parsing**  
Choose between basic text extraction or advanced parsing to capture tables, images, and more. By using advanced parsing, Edge RAG offers OCR-enabled support for documents, tables, and images. Tailor data ingestion to your needs for more accurate results. For more information, see [Advanced data parsing for Agentic Retrieval](advanced-data-parsing.md).

**Performance and scale**  
Experience up to 5× faster query performance for hybrid search and 100× faster ingestion of live-streamed images from the previous Edge RAG extension version `0.1.5`.

**Advanced search and chat experience**  

Edge RAG now offers a more powerful and flexible search and chat experience, making it easier to find information and interact with your data through new capabilities and interface improvements.

- Use hybrid multimodal search to retrieve images and deliver responses with rich visual content. For more information, see [Search types in Agentic Retrieval](search-types.md).
- Enjoy markdown-formatted responses that support images and rich text for responses that are easier to read and interpret.
- Chat directly with the language model, without using your organization’s data as context. Use the model only option to ask general questions, test the model’s capabilities, or get responses that aren’t influenced by your ingested data. Switch between knowledge-based chat and model-only chat to fit your needs. For more information, see [Knowledge layer configuration](knowledge-layer-overview.md#data-query).

**Preview support for disconnected scenarios**

Edge RAG is supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

**Related content**

For more information about this release, see:

- Blog: [Transforming City Operations: How Villa Park and DataON Deliver Real-Time Decisions and Resilience with Edge RAG](https://aka.ms/EdgeAI/EdgeRAG/IgniteBlog2025)


## Related content

- [Complete Agentic Retrieval deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agentic Retrieval extension](deploy.md)
