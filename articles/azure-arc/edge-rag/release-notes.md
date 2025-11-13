---
title: Release Notes for Edge RAG Preview Enabled by Azure Arc
ms.reviewer: cwatson
description: "Discover the latest updates and features for Edge RAG Preview enabled by Azure Arc."
author: cwatson-cat
ms.author: cwatson
ms.subservice: edge-rag
ms.topic: "release-notes"  
ms.date: 11/10/2025
ai-usage: ai-assisted

#customer intent: As an IT admin, I want to understand the new features in the latest Edge RAG release so that I can plan updates for my organization.

---

# Release notes for Edge RAG Preview enabled by Azure Arc

Edge RAG Preview enabled by Azure Arc helps you deploy Retrieval Augmented Generation (RAG) solutions at the edge. This article lists new features, improvements, and important changes for each release. Use these notes to plan, deploy, and manage Edge RAG in your organization.

Release notes are updated with each new version.

## November 2025

**Extension version**: `0.8.0` [Preview]

The following table summarizes the updates included in this release.

| Capability                | Description |
|---------------------------|---------|
| Data ingestion  | - Improved parsing for documents, including tables and charts.<br>- Added new ingestion type options for high-fidelity document processing.  <br></br>For more information, see [Advanced data parsing for Edge RAG](advanced-data-parsing.md)|
|Data query and chat   | - Added deep search model search type for production-class [LazyGraph RAG](https://www.microsoft.com/research/blog/lazygraphrag-setting-a-new-standard-for-quality-and-cost/?msockid=322913564b6d68c00e1d07c14a0269f0) with industry-leading RAG inferencing quality. <br> - Added hybrid multimodal search with image retrieval & image-rich outputs<br>- Added support for responses in markdown for improved readability.<br>- Added a model-only chat option for direct model chat. <br><br> For more information, see [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md) and [Advanced data parsing for Edge RAG](advanced-data-parsing.md).|
|Performance enhancements | From previous release version: <br>- Achieved a 5× faster query performance for hybrid search.<br>- Delivered 100× faster ingestion of live-streamed images. |
|Preview support for disconnected scenarios|Edge RAG supported as part of a preview for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview).|

For more information, see [What's new in Edge RAG](whats-new.md).

## July 2025

**Extension version**: `0.1.5` [Preview]

- Resolved authentication issues for endpoints created by Azure AI Foundry for bring your own model (BYOM) deployments.
- Temporarily disabled chat history to improve performance. Each question is answered based only on retrieved content.
- Improved security.

## May 2025

**Extension version**: `0.1.3` [Preview]

Edge RAG is available as a public preview.

See the related blog posts:

- [Transforming On-Premises Data with RAG Capabilities on Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/transforming-on-premises-data-with-rag-capabilities-on-azure-local/4415217)
- [Empowering the Physical World with AI - Unlocking AI at the Edge with Azure Arc](https://techcommunity.microsoft.com/blog/azurearcblog/empowering-the-physical-world-with-ai/4415204)
- [Unlocking AI Apps Across Boundaries with Azure](https://techcommunity.microsoft.com/blog/AzureArcBlog/unlocking-ai-apps-across-boundaries-with-azure/4410457)

## Related content

- [What you need for Edge RAG Preview enabled by Azure Arc](requirements.md)
- [Deployment overview for Edge RAG Preview enabled by Azure Arc](deploy-overview.md)