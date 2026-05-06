---
title: Release Notes for Agents and Tools with Foundry Local
ms.reviewer: cwatson
description: "Discover the latest updates and features for Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.subservice: edge-rag
ms.topic: "release-notes"  
ms.date: 05/04/2026
ai-usage: ai-generated

#customer intent: As an IT admin, I want to understand the new features in the latest Agents and Tools with Foundry Local release so that I can plan updates for my organization.

---

# Release notes for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local helps you deploy Retrieval Augmented Generation (RAG) solutions at the edge. This article lists new features, improvements, and important changes for each release. Use these notes to plan, deploy, and manage Agents and Tools with Foundry Local in your organization.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## June 2026

**Extension version**: `0.9.0` [Preview]

### Changes

| Change | Details |
|---|---|
| **Product renamed** | "Agents and Tools with Foundry Local" is now **Agents and Tools with Foundry Local**. The Azure extension type remains `microsoft.arc.rag`. |
| **SLM models removed** | Phi-3.5 and Mistral-7B models are no longer bundled. Bring Your Own Model (BYOM) is the only model path. You must provide an external LLM endpoint. |
| **GPU count reduced** | 4 → **2 GPUs** required. GPUs are used for text embedding (BGE-M3) and image embedding (CLIP ViT-L/14). Docling (document parser) now runs on CPU. The LLM runs externally via BYOM. |
| **BYOM is now mandatory** | The model endpoint step is no longer optional. All deployments require an OpenAI-compatible LLM endpoint. |

### New features

| Feature | Description |
|---|---|
| **Independent layer deployment** | New `layerSelection` ARM parameter: `combined` (default), `agentic`, `knowledge`. Deploy layers independently. |
| **Agents Runtime API** | Full agent orchestration with threads, messages, runs, and streaming (SSE). OpenAI Assistants-compatible API. Port 8080. |
| **Agent Manager API** | CRUD operations for agents and knowledge bases. Supports custom agent IDs, metadata. Port 8080. |
| **Knowledge Sources API** | Register MCP server connections as self-contained knowledge sources (two kinds: `remote_mcp` and `indexed_sources_mcp`). Port 3005. |
| **Collections API** | Manage multiple collections for vector data with per-collection RBAC. Port 3002. |
| **MCP Server** | Built-in MCP server with 7 search tools over Model Context Protocol. Port 8080. |
| **Ingestion API** | Public REST API for programmatic document ingestion. Port 8000. |
| **Inference API** | Public REST API for RAG chat completions and model-only queries. Port 3001. |
| **Agentic Chat UI** | New agentic chat interface with multi-turn conversations, thread history, and streaming. |

## February 2026

**Extension version**: `0.8.5` [Preview]

This release includes bug fixes, reliability improvements, and critical security updates.

### Search and query

This release improves the accuracy and reliability of search operations.

- Deep search returns accurate results for complex queries, including time-based and count-based questions.
- Timeouts for LLM inference calls, Vector DB queries, and database operations prevent resource exhaustion.
- Error messages clearly show whether the system is unavailable or the knowledge base is empty.

### Data ingestion

Data ingestion is more robust and handles edge cases without failure.

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
|Data query and chat   | - Added deep search model search type for production-class [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with industry-leading RAG inferencing quality. <br> - Added hybrid multimodal search with image retrieval and image-rich outputs.<br>- Added support for responses in markdown for improved readability.<br>- Added a model-only chat option for direct model chat. <br><br> For more information, see [Configuring the chat solution for Agents and Tools with Foundry Local](build-chat-solution-overview.md) and [Advanced data parsing for Agents and Tools with Foundry Local](advanced-data-parsing.md).|
|Performance enhancements | From previous release version: <br>- Achieved a 5× faster query performance for hybrid search.<br>- Delivered 100× faster ingestion of live-streamed images. |
|Preview support for disconnected scenarios|Agents and Tools with Foundry Local supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).|

For more information, see [What's new in Agents and Tools with Foundry Local](whats-new.md).

## July 2025

**Extension version**: `0.1.5` [Preview]

- Resolved authentication problems for endpoints that Microsoft Foundry created for bring your own model (BYOM) deployments.
- Temporarily disabled chat history to improve performance. Each question is answered based only on retrieved content.
- Improved security.

## May 2025

**Extension version**: `0.1.3` [Preview]

Agents and Tools with Foundry Local is available as a public preview.

See the related blog posts:

- [Transforming On-Premises Data with RAG Capabilities on Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/transforming-on-premises-data-with-rag-capabilities-on-azure-local/4415217)
- [Empowering the Physical World with AI - Unlocking AI at the Edge with Azure Arc](https://techcommunity.microsoft.com/blog/azurearcblog/empowering-the-physical-world-with-ai/4415204)
- [Unlocking AI Apps Across Boundaries with Azure](https://techcommunity.microsoft.com/blog/AzureArcBlog/unlocking-ai-apps-across-boundaries-with-azure/4410457)

## Related content

- [What you need for Agents and Tools with Foundry Local ](requirements.md)
- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)