---
title: How the Local chat UI works in Agents and Tools with Foundry Local
description: Learn what the Local chat UI is in Agents and Tools with Foundry Local, what users can do with it, and how users access it.
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted

#customer intent: As a platform engineer or solution architect, I want to understand the Local chat UI in Agents and Tools with Foundry Local so that I can help users get started quickly.
---

# How the Local chat UI works in Agents and Tools with Foundry Local

The *Local chat UI* is the browser chat experience for agents in Agents and Tools with Foundry Local. It's one of the three components of the extension, together with Agentic Retrieval and Agent Service. In this article, *chat* means the Local chat UI.

Chat helps users ask questions over their content and get grounded answers with citations.

This article explains the Local chat UI. You learn what users can do in chat, how users access chat, and how access control and runtime flow work.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## What users can do in chat

When users open chat, they can:

- Start from a **welcome screen** with an input box and optional prompt starters.
- Use a **sidebar with conversation history** that supports rename and delete.
- Get **streaming answers** with markdown, code blocks, and citations.
- **Sign in to access chat.** Both `auth.clientId` and `auth.tenantId` are required for all deployments. Connected deployments use Microsoft Entra ID, and disconnected deployments use the local identity provider configured in `auth.aadAuthorityHost`.

## Access the local chat UI

End users access chat from the registered application URL by using the `/chat` path.

Chat provides:

- Multi-turn conversations with AI agents.
- Conversation history that persists per user.
- Streaming responses through Server-Sent Events (SSE).
- Tool-call visibility so users can see the tools and knowledge sources the agent used.

Chat routes requests through agents, which can call knowledge bases and MCP tools before generating a response.

## Chat URL for end users

Append `/chat` to the registered application URL. For example: `https://arcrag.contoso.com/chat`.

## Collection access and RBAC for chat

Authentication (`auth.clientId` and `auth.tenantId`) is required for all deployments. Azure role-based access control (Azure RBAC) checks permissions at the extension resource scope. An administrator assigns **Reader** to let users list enabled agents and **Cognitive Services OpenAI User** to let them run agents through Responses and Conversations. In disconnected deployments, use **Contributor** for chat.

> [!IMPORTANT]
> Responses and Conversations have no app-role fallback. `EdgeRAGEndUser` or `EdgeRAGDeveloper` alone doesn't let you use the local chat UI. The extension's managed identity also needs **Reader** at the extension scope to check Azure roles.

Azure-role-authorized callers can access every collection. Azure RBAC doesn't filter individual collections.

Classic chat completions and search APIs still support *app-role collection access* as a fallback. With that fallback, `EdgeRAGEndUser` needs an app role matching each collection name, such as `edgeragapp` or `finance-docs`. `EdgeRAGDeveloper` can access every collection. These collection app roles don't replace the Azure role needed by the Local chat UI.

For setup steps, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles) and [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access).

## How the Local chat UI fits in Agents and Tools with Foundry Local

Chat is the user-facing entry point. Users type questions in chat. Agent Service runs the agent, and Agentic Retrieval retrieves content so that the agent can generate grounded answers.

This separation keeps chat simple while backend services handle orchestration, policy checks, and data access rules.

## Chat runtime architecture and flow

Chat uses the same APIs as your own applications. It gets the list of enabled agents from the agents management service. It then uses the Conversations API and the Responses API on the agents execution service. The execution service runs the agent version, calls the model and tools, and streams the answer back to chat.

:::image type="content" source="./media/chat-experience/architecture.png" alt-text="Diagram that shows chat listing agents from agents management, and sending conversations and responses to agents execution, which calls the language model and tools and streams the answer back to chat." lightbox="./media/chat-experience/architecture.png" border="false":::

The following sequence shows how a user prompt moves through the agent services and streams back to chat.

:::image type="content" source="./media/chat-experience/end-to-end-chat-flow.png" alt-text="Sequence diagram. Chat gets agents and the user's conversations, creates a conversation for a new chat, sends a streaming response request, and renders the streamed answer and citations." lightbox="./media/chat-experience/end-to-end-chat-flow.png" border="false":::

For the APIs that chat uses, see [Run agents with the Responses API](run-agents-responses-api.md).

### Identity path

All deployments require `auth.clientId` and `auth.tenantId`. The browser gets a user token from the configured identity provider and sends it to the agent services for validation and user scoping. Connected deployments use Microsoft Entra ID, and disconnected deployments use the local identity provider configured in `auth.aadAuthorityHost` on the disconnected stamp.

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Test the Local chat UI](test-end-user-app.md)
- [Agent Service overview](agentic-overview.md)
