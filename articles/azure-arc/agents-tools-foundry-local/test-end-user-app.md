---
title: Test the Local chat UI in Agents and Tools with Foundry Local
description: "Learn how to test the end-user query experience in the Local chat UI of Agents and Tools with Foundry Local to evaluate AI-powered search in hybrid or multicloud environments."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a developer or IT administrator, I want to test chat in Agents and Tools with Foundry Local by using the provided application so that I can assess AI-powered search performance and user experience.
---

# Test the end-user query experience in the Local chat UI

After you configure Agentic Retrieval and create an enabled agent in Agent Service, test the solution by using the Local chat UI. The Local chat UI is the browser chat experience for agents.

To learn more about chat behavior, end user access requirements, and architecture, see [How the Local chat UI works](chat-experience.md).

## Prerequisites

Before you begin:

- Decide which collection your end users should query (for example, the default `edgeragapp` collection).
- Create an enabled agent connected to a knowledge base and an `indexed_sources_mcp` knowledge source for collection grounding. Settings in the developer portal **Chat playground** (`/developer/chat`) don't configure `/chat`. See [Quickstart: Create an agent and chat with it](quickstart-create-agent.md) and [Create and manage agents](create-manage-agents.md).
- Ask an administrator to assign **Reader** to users so they can list enabled agents, and **Cognitive Services OpenAI User** so they can run agents and use chat. Assign these Azure roles at the extension resource scope. In disconnected deployments, use **Contributor**. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- Assign **Reader** at the extension scope to the extension's managed identity, so it can check caller roles. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles).

> [!IMPORTANT]
> The Local chat UI uses Responses and Conversations, which have no app-role fallback. `EdgeRAGEndUser` alone doesn't grant chat access. Callers allowed by an Azure role can access every collection; collection app roles don't filter their access.

## Verify end-user query results in chat

To try the Local chat UI, append `/chat` to the registered application URL.

1. Go to your registered app URL with `/chat` appended (for example, `https://arcrag.contoso.com/chat`).
1. Sign in with an account that has the required Azure roles.
1. Select an enabled agent, start using chat by entering a query, and then enter a follow-up query in the same conversation.
1. Confirm that responses include citations and that conversation history is visible.

## Related content

- [Agentic Retrieval overview](knowledge-layer-overview.md)
- [Add a data source](add-data-source.md)
- [Set up the data query](set-up-data-query.md)
