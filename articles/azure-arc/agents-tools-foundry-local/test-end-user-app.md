---
title: Test the End-User Query Experience in Agentic Retrieval in Foundry Local
description: "Learn how to test the end user experience of the Agentic Retrieval in Foundry Local chat solution to evaluate AI-powered search in hybrid or multicloud environments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 06/01/2026
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a developer or IT administrator, I want to test chat in Agentic Retrieval in Foundry Local by using the provided application so that I can assess AI-powered search performance and user experience.
---

# Test the end-user query experience in Agentic Retrieval in Foundry Local

After you configure the Knowledge Layer (or the full Agentic Retrieval platform), test the solution by using the built-in chat applications.

To learn more about chat behavior, end user access requirements, and architecture, see [How chat works in Agentic Retrieval](chat-experience.md).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Decide which collection your end users should query (for example, the default `edgeragapp` collection).
- Make sure each end user is assigned to the app role for the target collection. For setup steps, see [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access).
- To access chat, users must have the **EdgeRAGEndUser** role in Microsoft Entra. For more information see, [Create app roles for Agentic Retrieval](prepare-authentication.md#create-app-roles-for-agentic-retrieval).

## Verify end-user query results in chat

To try the built-in chat for end users, start from the registered URL for your Agentic Retrieval app and use the `/chat` path.

1. Go to your registered app URL with `/chat` appended (for example, `https://arcrag.contoso.com/chat`).
1. Sign in by using the end user credentials that have the **EdgeRAGEndUser** role assigned. If you have the right access configured, you're automatically redirected to the chat portal.
1. Start using chat by entering a query, then enter a follow-up query in the same conversation.
1. Confirm that responses include citations and that conversation history is visible.
1. (Optional) To share feedback to Microsoft, select thumbs up or thumbs down.

## Related content

- [Knowledge layer configuration](knowledge-layer-overview.md)
- [Add a data source for chat](add-data-source.md)
- [Set up data query for chat](set-up-data-query.md)

