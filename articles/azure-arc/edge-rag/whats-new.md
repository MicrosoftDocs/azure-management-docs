---
title: What's New in Edge RAG
description: Learn about the latest new features and announcement from the past few months.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 02/02/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As an IT administrator or technical decision maker, I want to stay updated on the latest features and improvements for Edge RAG so that I can effectively plan, deploy, and manage the Edge RAG solution in my organization.
---

# What's new in Edge RAG Preview enabled by Azure Arc

This article lists the various features and improvements that are available in Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

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
Deployment failures now provide accessible logs, so you can identify root causes faster without escalating to support. This improvement reduces time to resolution and helps you get Edge RAG running smoothly.

**Critical security updates**  
This release addresses two important vulnerabilities: a critical Next.js Denial-of-Service vulnerability (GHSA-5j59-xgg2-r9c4) and a high-severity Langchain XXE vulnerability that could allow unauthorized file access. Dependency updates also resolve version conflicts and improve overall system stability.

## November 2025

### Release of extension version `0.8.2`

This release of Edge RAG introduces several new features, enhancements, and improvements designed to expand capabilities, improve performance, and streamline the user experience.

**Deep search**  
Find the most relevant information with the new deep search model. Deep search uses production-class [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with industry-leading RAG inferencing quality. Edge RAG now explores and connects data across sources at query time, so you get comprehensive answers without heavy upfront processing. For more information, see [Search types in Edge RAG](search-types.md).

**High-fidelity parsing**  
Choose between basic text extraction or advanced parsing to capture tables, images, and more. By using advanced parsing, Edge RAG offers OCR-enabled support for documents, tables, and images. Tailor data ingestion to your needs for more accurate results. For more information, see [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md#planning-data-ingestion) and  [Advanced data parsing for Edge RAG](advanced-data-parsing.md).

**Performance and scale**  
Experience up to 5× faster query performance for hybrid search and 100× faster ingestion of live-streamed images from the previous Edge RAG extension version `0.1.5`.

**Advanced search and chat experience**  

Edge RAG now offers a more powerful and flexible search and chat experience, making it easier to find information and interact with your data through new capabilities and interface improvements.

- Use hybrid multimodal search to retrieve images and deliver responses with rich visual content. For more information, see [Search types in Edge RAG](search-types.md).
- Enjoy markdown-formatted responses that support images and rich text for responses that are easier to read and interpret.
- Chat directly with the language model, without using your organization’s data as context. Use the model only option to ask general questions, test the model’s capabilities, or get responses that aren’t influenced by your ingested data. Switch between knowledge-based chat and model-only chat to fit your needs. For more information, see [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md#data-query).

**Preview support for disconnected scenarios**

Edge RAG is supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).

**Related content**

For more information about this release, see:

- Blog: [Transforming City Operations: How Villa Park and DataON Deliver Real-Time Decisions and Resilience with Edge RAG]( https://aka.ms/EdgeAI/EdgeRAG/IgniteBlog2025)

## October 2025

### New article: Quickstart Install Edge RAG

To try Edge RAG in an evaluation or development environment without the need for local hardware, see [Quickstart: Install Edge RAG Preview enabled by Azure Arc](quickstart-edge-rag.md).

## Related content

- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
