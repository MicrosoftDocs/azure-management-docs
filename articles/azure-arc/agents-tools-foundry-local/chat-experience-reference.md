---
title: Chat UI configuration and API reference in Agents and Tools with Foundry Local
description: Reference for Chat UI runtime configuration, environment variables, and runtime API operations in Agents and Tools with Foundry Local.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/22/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-assisted

#customer intent: As a platform engineer, I want a complete Chat UI reference so I can configure runtime behavior and understand API usage.
---

# Chat UI configuration and API reference in Agents and Tools with Foundry Local

Use this article as the reference for Chat UI runtime configuration in Agents and Tools with Foundry Local.

## Configuration model

All Chat UI configuration is injected through `VITE_*` environment variables at container startup.

Three layers apply in order:

```text
1. DEFAULTS (hardcoded)
       ↓ overridden by
2. VITE_* env vars (Helm values / Docker / kubectl set env)
       ↓ derived after
3. Derived values (currently only auth.enabled)
```

## Required settings for agentic RAG mode

Use these settings to connect the chat UI to `agents-runtime` in agentic RAG mode.

Minimum settings:

```yaml
env:
  - name: VITE_API_MODE
    value: "agents"
  - name: VITE_API_URL
    value: "/runtime"
  - name: VITE_AGENT_ID
    value: "asst_xxxx"
  - name: VITE_BASE_PATH
    value: "/chat"
```

| Env var | Config key | Required | Description |
|---|---|---|---|
| `VITE_API_MODE` | `api.mode` | yes | Set to `agents` for agentic retrieval-augmented generation (RAG) |
| `VITE_API_URL` | `api.baseUrl` | yes | Path to `agents-runtime` (usually `/runtime`) |
| `VITE_AGENT_ID` | `agents.id` | yes | Agent ID from `agents-manager` |
| `VITE_BASE_PATH` | `app.basePath` | when subpath | URL subpath (for example, `/chat`) |

## Authentication configuration (Entra ID)

Set these values for connected deployments. Omit them for disconnected deployments.

```yaml
env:
  - name: VITE_AUTH_CLIENT_ID
    value: "00001111-aaaa-2222-bbbb-3333cccc4444"
  - name: VITE_AUTH_TENANT_ID
    value: "aaaabbbb-0000-cccc-1111-dddd2222eeee"
```

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_AUTH_ENABLED` | `auth.enabled` | auto | `true` when `clientId` is set; explicit override allowed |
| `VITE_AUTH_CLIENT_ID` | `auth.clientId` | `""` | Entra app registration client ID |
| `VITE_AUTH_TENANT_ID` | `auth.tenantId` | `""` | Entra tenant ID |

## Branding

Use these settings to control the app name, browser title, and visual branding elements.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_APP_NAME` | `app.name` | `Edge AI Chat` | Application display name |
| `VITE_APP_TITLE` | `app.title` | `Edge AI Chat` | Browser tab title |
| `VITE_AGENT_NAME` | `agent.name` | `Edge AI Chat` | Agent display name |
| `VITE_APP_FAVICON` | `app.favicon` | `""` | Path to favicon (empty means none) |

## Sidebar and conversation list

Use these settings to control how the sidebar and saved conversations appear when the app opens.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_SIDEBAR_ENABLED` | `sidebar.enabled` | `true` | Show conversation list sidebar |
| `VITE_SIDEBAR_IS_OPEN` | `sidebar.isOpen` | `true` | Default open state |
| `VITE_SIDEBAR_SHOW_ICON` | `sidebar.showIcon` | `false` | Show brand icon in sidebar header |
| `VITE_SIDEBAR_ICON` | `sidebar.icon` | `""` | Path to header icon (in `/public`) |
| `VITE_CONVERSATIONS_INITIAL_LIMIT` | `conversations.initialLimit` | `50` | Initial fetch limit |

## Chat surface

Use these settings to control prompt behavior and conversation routing in the main chat area.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_CHAT_MAX_LENGTH` | `chat.maxLength` | `4000` | Max characters per prompt |
| `VITE_CHAT_SHOW_PROMPT_STARTERS` | `chat.showPromptStarters` | `false` | Show prompt starters on welcome screen |
| `VITE_CHAT_USE_ROUTES` | `chat.useRoutes` | `true` | Use URL routes for conversations |

## Message and new chat icons

Use these settings to control the icons shown for assistant messages and the new-chat action.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_CHAT_SHOW_MESSAGE_ICON` | `chat.showMessageIcon` | `true` | Show assistant avatar |
| `VITE_CHAT_MESSAGE_ICON` | `chat.messageIcon` | `""` | Path to custom assistant icon |
| `VITE_NEW_CHAT_SHOW_ICON` | `newChat.showIcon` | `true` | Show new-chat icon |
| `VITE_NEW_CHAT_ICON` | `newChat.icon` | `""` | Path to custom new-chat icon |

## Layout

Use these settings to control the width and card layout of the chat UI.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_LAYOUT_MAX_WIDTH` | `layout.maxWidth` | `950px` | Max width of main content area |
| `VITE_LAYOUT_CARD_COLUMNS` | `layout.cardColumns` | `2` | Card grid columns |

## Feature flags

Use these flags to turn selected chat UI capabilities on or off.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_FEATURES_RENAME_CONVERSATION` | `features.renameConversation` | `true` | Allow renaming conversations |
| `VITE_FEATURES_DELETE_CONVERSATION` | `features.deleteConversation` | `true` | Allow deleting conversations |
| `VITE_FEATURES_SEARCH` | `features.search` | `false` | Enable sidebar search |

## Security classification banner

Use these settings to show a classification banner at the top of the chat UI.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_SECURITY_BANNER_TEXT` | `security.bannerText` | `""` | Classification text |
| `VITE_SECURITY_BANNER_COLOR` | `security.bannerColor` | `""` | One of `red`, `yellow`, `green`, `blue` |

## Copilot surface

Use these settings to control the Copilot-specific layout and design mode.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_COPILOT_MODE` | `copilot.mode` | `canvas` | `canvas` (larger) or `default` (compact) |
| `VITE_COPILOT_DESIGN_VERSION` | `copilot.designVersion` | `next` | Fluent UI Copilot design version |

## Foundry Agents API surface used by Chat UI

The chat UI calls these Foundry Agents API operations during normal conversation flow.

| Operation | Method | Path |
|---|---|---|
| Create thread | POST | `/threads` |
| List threads | GET | `/threads?limit=100` |
| Delete thread | DELETE | `/threads/{id}` |
| Create message | POST | `/threads/{id}/messages` |
| List messages | GET | `/threads/{id}/messages?order=asc&limit=N&after=ID` |
| Create run (streaming) | POST | `/threads/{id}/runs?stream=true` |

## Common deployment questions

Use these quick answers to resolve common chat UI deployment and configuration questions.

| Question | Short answer |
|---|---|
| How do I enable Chat UI for Agents and Tools with Foundry Local? | Set `VITE_API_MODE=agents`, `VITE_API_URL`, and `VITE_AGENT_ID`. |
| How do I host Chat UI on `/chat` instead of `/`? | Set `VITE_BASE_PATH=/chat` so the startup base URL is correct. |
| How do I enable Entra ID sign-in? | Set `VITE_AUTH_CLIENT_ID` and `VITE_AUTH_TENANT_ID`. |
| How do I run disconnected without sign-in? | Omit auth values so Chat UI runs without Entra ID sign-in. |

## Related content

- [How Chat UI works in Agents and Tools with Foundry Local](chat-experience.md)
- [Deploy and validate Chat UI for Agents and Tools with Foundry Local](chat-experience-deploy.md)