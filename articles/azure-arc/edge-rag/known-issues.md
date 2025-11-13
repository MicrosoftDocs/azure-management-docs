---
title: Known Issues in Edge RAG Preview, Enabled by Azure Arc
description: "Read about the known issues and fixed issues with Edge RAG."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article #Don't change
ms.date: 10/16/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a cloud solution architect, I want to understand the known issues in Edge RAG preview so that I can make informed decisions during deployment and mitigate potential problems for my team.
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
|Chat history | With Edge RAG extension version 0.1.5 and later each question is answered based on retrieved content only. The answer doesn't include the context of the chat history. Chat history isn't saved between questions. Treat each question as a new chat. |


## Related content

- [What you need for Edge RAG](requirements.md)
- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
