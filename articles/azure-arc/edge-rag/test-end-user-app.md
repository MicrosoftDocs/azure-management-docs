---
title: Test Chat Solution for Agentic RAG
description: "Learn how to test the end user experience of the Agentic RAG chat solution to evaluate AI-powered search in hybrid or multicloud environments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 04/30/2026
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a developer or IT administrator, I want to test the Agentic RAG chat solution using the provided application, so that I can assess the performance and user experience of AI-powered search in hybrid or multicloud environments.
---

# Test the chat solution for Agentic RAG Preview, enabled by Azure Arc

After you configure the Knowledge Layer (or the full Agentic RAG platform), test the solution using the built-in chat applications.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

To access the chat solution, you must have the "EdgeRAGEndUser" role in Microsoft Entra.

## Use the chat application

To try the chat for end users, start from to the local chat portal.

1. Go to the developer portal by using the domain name provided at deployment and app registration, appended with "/user". For example: `https://arcrag.contoso.com/user`.
1. Sign in with the end user credentials that has the "EdgeRAGEndUser" role assigned. If you have the right access configured, you're automatically redirected to the chat portal.
1. Start using the simple chat interface by entering a query.

    Be aware that with Edge RAG extension version `0.1.5` and later each question is answered based on retrieved content only. The answer doesn't include the context of the chat history. Chat history isn't saved between questions. Treat each question as a new chat.
1. (Optional) To share feedback to Microsoft, select thumbs up or thumbs down.

## Agentic Chat UI

If you deployed Agentic RAG in **combined** or **agentic** mode, a new **agentic chat interface** is available alongside the legacy developer portal chat playground.

The agentic chat UI provides:
- Multi-turn conversations with AI agents
- Thread history (conversations are persisted per user)
- Streaming responses via Server-Sent Events (SSE)
- Run step visibility (see what tools/knowledge sources the agent used)

To access the agentic chat UI, navigate to the agentic chat endpoint on your cluster domain.

The legacy chat at `/user` queries the Knowledge Layer directly (Inference API). The agentic chat UI routes queries through agents, which can invoke knowledge bases and MCP tools before generating a response. Both interfaces are available simultaneously.

## Collection access for end users

When an end user with the `EdgeRAGEndUser` role accesses the chat, their access to collections is controlled by RBAC:

- The user must have an Entra ID app role matching the collection name (e.g., role `finance-docs` grants access to collection `finance-docs`).
- The `EdgeRAGDeveloper` role bypasses RBAC and grants access to all collections.
- If a user has `EdgeRAGEndUser` but **no collection-specific roles**, they will receive a `403 Forbidden` error with no guidance on which roles are needed.

Make end users are assigned app roles matching the collection names they need to access before granting them the `EdgeRAGEndUser` role. The default `edgeragapp` collection also requires an `edgeragapp` role for end users.

## Related content

- [Configuring the chat solution for Agentic RAG](build-chat-solution-overview.md)
- [Add data source for the chat solution in Agentic RAG](add-data-source.md)
[Set up the data query for Agentic RAG chat solution](set-up-data-query.md)

