---
title: What's New in Edge RAG
description: Learn about the latest new features and announcement from the past few months.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 07/01/2025
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.custom:
  - build-2025
# Customer intent: As an IT administrator or technical decision maker, I want to stay updated on the latest features and improvements for Edge RAG so that I can effectively plan, deploy, and manage the Edge RAG solution in my organization.
---

# What's new in Edge RAG Preview enabled by Azure Arc

This article lists the various features and improvements that are available in Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## July 2025

### Release of extension version `0.1.5`

The newly released Edge RAG extension version `0.1.5` includes the following changes:

- Resolved an authentication issue for endpoints created by Azure AI Foundry for bring your own model (BYOM) deployments.
- Disabled chat history temporarily while performance improvements are being made for a future release. Each question is answered based on retrieved content only and doesn't include the context of the chat history. Treat each question as a new chat.
- Improved security.

## June 2025

### New article: Create an endpoint to use for Edge RAG deployment

If you plan to use your own language model instead of one of the models provided by Microsoft, you must set up an OpenAI API compatible endpoint to use with Edge RAG. For more information, see [Create an endpoint to use for Edge RAG deployment](prepare-model-endpoint.md).

### Prerequisites for deployment reorganized into a checklist

To prepare for your deployment of Edge RAG, complete the steps listed in the new checklist:
[Deployment prerequisites checklist for Edge RAG Preview enabled by Azure Arc](complete-prerequisites.md). To improve the documentation quality and experience, each deployment prerequisite is now in a separate article.

## May 2025

### Edge RAG in public preview

**Extension version**: `0.1.3`

Edge RAG is now available as a public preview. To learn more, see the following documentation:

- [What is Edge Retrieval Augmented Generation (RAG)?](overview.md)
- [What you need for Edge RAG](requirements.md)

See the related blog posts:

- [Transforming On-Premises Data with RAG Capabilities on Azure Local](https://techcommunity.microsoft.com/blog/azurearcblog/transforming-on-premises-data-with-rag-capabilities-on-azure-local/4415217)
- [Empowering the Physical World with AI - Unlocking AI at the Edge with Azure Arc](https://techcommunity.microsoft.com/blog/azurearcblog/empowering-the-physical-world-with-ai/4415204)
- [Unlocking AI Apps Across Boundaries with Azure](https://techcommunity.microsoft.com/blog/AzureArcBlog/unlocking-ai-apps-across-boundaries-with-azure/4410457)

## Related content

- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
