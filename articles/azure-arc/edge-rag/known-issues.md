---
title: Known Issues in Edge RAG Preview, Enabled by Azure Arc
description: "Read about the known issues and fixed issues with Edge RAG."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 05/20/2025
ms.subservice: edge-rag
#CustomerIntent:
ms.custom:
  - build-2025
---

# Known issues in Edge RAG Preview, enabled by Azure Arc

Before you deploy Edge RAG, review the known issues in Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Known issues for public preview

The following table lists the known issues in this release.


|Feature  |Issue |
|---------|---------|
|Automatic evaluation|If you run an automatic evaluation without adding a data source, you receive an error like: "Failed to calculate automatic metrics. Both "answer" and "context must be non-empty strings." To work around this issue, [add a data source](add-data-source.md) before you run an evaluation.|
|Chat feedback    | End users of the chat solution can submit feedback about the chat, but the AI Application Developers/Prompt Engineers that set up the chat solution don't have an easy UI-based way to analyze the feedback. |
|Chat history | Using the chat history for inferencing in subsequent queries is in trial phase. If you see unexpected references in your query session, start a new chat session and try the question again. Be aware that starting a new chat history deletes the existing chat history. |


## Related content

- [What you need for Edge RAG](requirements.md)
- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
