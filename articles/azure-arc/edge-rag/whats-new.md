---
title: What's New in Edge RAG
description: Learn about the latest new features and announcement from the past few months.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 11/17/2025
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As an IT administrator or technical decision maker, I want to stay updated on the latest features and improvements for Edge RAG so that I can effectively plan, deploy, and manage the Edge RAG solution in my organization.
---

# What's new in Edge RAG Preview enabled by Azure Arc

This article lists the various features and improvements that are available in Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## November 2025

### Release of extension version `0.8.2`

This release of Edge RAG introduces several new features, enhancements, and improvements designed to expand capabilities, improve performance, and streamline the user experience.

**Deep search**  
Find the most relevant information with the new deep search model. Deep search uses production-class [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with industry-leading RAG inferencing quality. Edge RAG now explores and connects data across sources at query time, so you get comprehensive answers without heavy upfront processing. For more information, see [Search types in Edge RAG](search-types.md).

**High-fidelity parsing**  
Choose between basic text extraction or advanced parsing to capture tables, images, and more. With advanced parsing, Edge RAG offers OCR-enabled support for documents, tables, and images. Tailor data ingestion to your needs for more accurate results. For more information, see [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md#planning-data-ingestion) and  [Advanced data parsing for Edge RAG](advanced-data-parsing.md).

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
