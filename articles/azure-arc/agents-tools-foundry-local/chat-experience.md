---
title: How chat works in Agents and Tools with Foundry Local
description: Learn what chat is in Agents and Tools with Foundry Local, what users can do with it, and the default local URL.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/28/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-assisted

#customer intent: As a platform engineer or solution architect, I want to understand the built-in chat in Agents and Tools with Foundry Local so that I can help users get started quickly.
---

# How chat works in Agents and Tools with Foundry Local

In this article, *chat* is the built-in user chat interface in Agents and Tools with Foundry Local.

Chat helps users ask questions over their content and get grounded answers with citations.

This article explains chat in combined and agentic deployments of the Agentic Retrieval in Foundry Local extension. You learn what users can do in chat, how chat differs from the developer portal `/user` path, and how access control and runtime flow work.

## What users can do in chat

When users open chat, they can:

- Start from a **welcome screen** with an input box and optional prompt starters.
- Use a **sidebar with conversation history** that supports rename and delete.
- Get **streaming answers** with markdown, code blocks, and citations.
- **Sign in with a work account** (Microsoft Entra ID) in connected deployments.
- Use chat in disconnected or air-gapped deployments, where sign-in is skipped.

## Chat in combined and agentic deployments

If you deploy Agents and Tools with Foundry Local in combined or agentic deployments, you get chat alongside the developer portal chat at `/user`.

Chat provides:

- Multi-turn conversations with AI agents.
- Thread history that persists per user.
- Streaming responses through Server-Sent Events (SSE).
- Run-step visibility so users can see the tools and knowledge sources the agent used.

The `/user` chat path queries the knowledge layer directly through the Inference API.

Chat routes requests through agents, which can call knowledge bases and MCP tools before generating a response. You can use both interfaces at the same time.

## Default local URL for chat

The default local chat endpoint is `http://localhost:5173`.

Use this endpoint when you run chat locally.

## Collection access and RBAC for chat

Azure role-based access control (Azure RBAC) controls which collections each user can query in chat.

- Users need the `EdgeRAGEndUser` role to sign in and use chat.
- Users also need an app role that matches each collection name they should access. The default collection is `edgeragapp`, so users who query the default collection need the `edgeragapp` app role. For example, the app role `finance-docs` grants access to collection `finance-docs`.
- Users with `EdgeRAGDeveloper` can access all collections.

If a user has `EdgeRAGEndUser` but no matching collection role, chat requests can return `403 Forbidden`.

For setup steps, see [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access).

## How chat fits in Agents and Tools with Foundry Local

Chat is the user-facing entry point. Users type questions in chat, then backend runtime services retrieve content and generate grounded answers.

This separation keeps chat simple while backend services handle orchestration, policy checks, and data access rules.

## Chat runtime architecture and flow

This diagram shows the high-level relationship between the user, chat, runtime services, and knowledge sources.

:::image type="content" source="./media/chat-experience/architecture.png" alt-text="Diagram that shows the chat flow from user to chat, runtime services, agent, and knowledge sources, with responses returned to chat." lightbox="./media/chat-experience/architecture.png" border="false":::

The following sequence shows how a user prompt moves through runtime services and streams back to chat.

:::image type="content" source="./media/chat-experience/end-to-end-chat-flow.png" alt-text="Sequence diagram of end-to-end chat flow from user to chat, runtime services, and agent tools, with streamed responses returned to chat." lightbox="./media/chat-experience/end-to-end-chat-flow.png" border="false":::

### Connected identity path

In connected deployments, the browser gets a user token through Entra ID and sends it to runtime services for validation and user scoping.

In disconnected or air-gapped deployments, chat runs without Entra sign-in, and the network boundary becomes the trust boundary.

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Test the chat solution for Agents and Tools with Foundry Local](test-end-user-app.md)
- [What is Agentic Retrieval Augmented Generation (RAG)?](agentic-overview.md)