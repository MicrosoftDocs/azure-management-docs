---
title: Test Chat Solution for Edge RAG
description: "Learn how to test the end user experience of the Edge RAG chat solution to evaluate AI-powered search in hybrid or multicloud environments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 07/01/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a developer or IT administrator, I want to test the Edge RAG chat solution using the provided application, so that I can assess the performance and user experience of AI-powered search in hybrid or multicloud environments.
---

# Test the chat solution for Edge RAG Preview, enabled by Azure Arc

After you built the chat solution, use the out-of-the-box chat application for end user testing or for end users to get started quickly.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

To access the chat solution, you must have the "EdgeRAGEndUser" role in Microsoft Entra.

## Use the chat application

To try the chat for end users, start from to the local chat portal.

1. Go to the developer portal by using the domain name provided at deployment and app registration, appended with "/user". For example: `https://arcrag.contoso.com/user`.
1. Sign in with the end user credentials that has the "EdgeRAGEndUser" role assigned. If you have the right access configured, you're automatically redirected to the chat portal.
1. Start using the simple chat interface by entering a query.

    Be aware that with Edge RAG extension version `0.1.5` each question is answered based on retrieved content only and doesn't include the context of the chat history. Chat history is not saved between questions. Treat each question as a new chat.
1. (Optional) To share feedback to Microsoft, select thumbs up or thumbs down.

## Related content

- [Configuring the chat solution for Edge RAG](build-chat-solution-overview.md)
- [Add data source for the chat solution in Edge RAG](add-data-source.md)
[Set up the data query for Edge RAG chat solution](set-up-data-query.md)

