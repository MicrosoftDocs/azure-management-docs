---
title: How Chat UI works in Agents and Tools with Foundry Local
description: Learn how Chat UI works in Agents and Tools with Foundry Local, including deployment architecture, runtime configuration, identity options, and backend mode integration.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/20/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-assisted

#customer intent: As a platform engineer or solution architect, I want to understand how the Chat UI works in Agents and Tools with Foundry Local so that I can design and deploy a consistent local chat experience.
---

# Chat UI in Agents and Tools in Foundry Local

The Chat user interface (UI) is the web front end for Agents and Tools with Foundry Local. It gives users a chat surface to ask questions over configured knowledge sources and receive grounded responses with citations.

If you design or operate local artificial intelligence (AI) deployments, you need a chat experience that stays consistent across connected and disconnected environments while keeping runtime and policy enforcement in backend services.

This article explains where the Chat UI fits in the Agents and Tools with Foundry Local architecture, how runtime configuration and identity settings shape behavior, and how backend mode selection affects integration patterns.

## What the Chat UI is

The Chat UI is a ready-to-deploy web app that ships with Agents and Tools with Foundry Local. It provides a Microsoft 365-style interface for interacting with an Agents and Tools with Foundry Local agent while keeping orchestration and policy enforcement in backend services.

When users open it in a browser they see:

- A **welcome screen** with an input box and optional prompt starters.
- A **sidebar with conversation history** that supports rename, delete, and optional search.
- An **assistant reply that streams incrementally**, including markdown, code blocks, and **inline citations/source chips** to grounded documents.
- **Sign-in with a work account** (Entra ID) for connected deployments, with user-scoped conversation history.
- The same experience in **disconnected or air-gapped** deployments, where sign-in is skipped and the network boundary becomes the trust boundary.

### User experience capabilities

The following capabilities define what end users can do in the Chat UI.

| Area | What end users get |
|---|---|
| Conversations | Multiturn chat, persisted as threads; switch between threads in the sidebar |
| Streaming | Assistant answers stream in increments |
| Grounding | Citations and source chips rendered alongside the assistant message |
| History | Rename, delete, and optionally search past conversations |
| Titles | Sidebar entries get a backend-generated title after the first reply (Microsoft 365-style deferred title), with local fallback when title generation is unavailable |
| Welcome | Optional prompt starters to nudge first-time users |
| Identity | Entra ID sign‑in on connected deployments; auto‑disabled when no client ID is configured |

### What deployment teams can control

The Chat UI is configuration-driven at deploy time. The same container image is reused everywhere, and behavior is toggled by `VITE_*` environment variables on the pod:

- **Branding**, app name, browser tab title, agent display name, favicon, sidebar header icon, assistant avatar, and new chat icon.
- **Layout**, max content width, card grid columns, and subpath where the UI is served (`/`, `/chat`, ...).
- **Sidebar and history**, sidebar visibility, default open state, initial conversation fetch limit, rename, delete, and search.
- **Chat surface**, max prompt length, welcome-screen prompt starters, URL routes for conversations.
- **Security banner**, classification banner text and color for regulated deployments.
- **Authentication**, Entra client ID, tenant, authority, and redirect URI (Uniform Resource Identifier); omit to disable auth entirely.
- **Backend wiring**, backend mode, runtime URL, agent ID.

### Where it fits in Agents and Tools with Foundry Local

The Chat UI is one deployable component of Agents and Tools with Foundry Local, alongside the agent runtime and knowledge sources. It's the user-facing surface only. It doesn't directly call models, orchestrate tools, validate tokens, enforce role-based access control (RBAC), or scope data. Those responsibilities remain with `agents-runtime`, the MISE sidecar, the agent, and knowledge source services.

### Supported backend modes

The same image supports several backend modes. Agentic retrieval-augmented generation (RAG) for Agents and Tools with Foundry Local is the default and the focus of this article. The other modes exist for adjacent scenarios.

| Mode | `VITE_API_MODE` | Use case |
|---|---|---|
| Agentic RAG | `agents` | Local or Arc-enabled Agents and Tools with Foundry Local deployments that run `agents-runtime` |
| Server + bring your own model (BYOM) | `server` | Teams that need a backend adapter, mutual Transport Layer Security (mTLS), or server-held API keys. See your solution's provider documentation. |
| Frontend-direct completions | `completions` | Browser-direct call to an OpenAI-compatible endpoint (for example, Foundry Local). See your solution's provider documentation. |


## How configuration works

All Chat UI configuration is injected via `VITE_*` environment variables at container startup. The entrypoint writes them into `window.__RUNTIME_CONFIG__` at runtime, so the **same image is reused across environments, no rebuild is ever needed**.

Three layers apply in order:

```text
1. DEFAULTS (hardcoded)
       ↓ overridden by
2. VITE_* env vars (Helm values / Docker / kubectl set env)
       ↓ derived after
3. Derived values (currently only auth.enabled)
```

### Required settings for Agentic RAG mode

Minimum set to surface the Chat UI in an Agentic RAG deployment:

```yaml
env:
  - name: VITE_API_MODE
    value: "agents"           # Activates the Foundry Agents API adapter
  - name: VITE_API_URL
    value: "/runtime"         # Path to agents-runtime (proxied via ingress)
  - name: VITE_AGENT_ID
    value: "asst_xxxx"        # Agent ID from agents-manager
  - name: VITE_BASE_PATH
    value: "/chat"            # Only when serving from a shared ingress subpath
```

| Env var | Config key | Required | Description |
|---|---|---|---|
| `VITE_API_MODE` | `api.mode` | yes | Set to `agents` for Agentic RAG |
| `VITE_API_URL` | `api.baseUrl` | yes | Path to `agents-runtime` (usually `/runtime`) |
| `VITE_AGENT_ID` | `agents.id` | yes | Agent ID from `agents-manager` |
| `VITE_BASE_PATH` | `app.basePath` | when subpath | URL subpath (e.g. `/chat`); injects `<base href>` at startup |

### Authentication configuration (Entra ID)

Set these values for connected deployments. Omit them for disconnected deployments, where auth is auto-disabled.

```yaml
env:
  - name: VITE_AUTH_CLIENT_ID
    value: "00001111-aaaa-2222-bbbb-3333cccc4444"     # Entra app registration client ID (approved fake GUID)
  - name: VITE_AUTH_TENANT_ID
    value: "aaaabbbb-0000-cccc-1111-dddd2222eeee"     # Entra tenant ID (approved fake GUID)
```

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_AUTH_ENABLED` | `auth.enabled` | auto | `true` when `clientId` is set; explicit override allowed |
| `VITE_AUTH_CLIENT_ID` | `auth.clientId` | `""` | Entra app registration client ID |
| `VITE_AUTH_TENANT_ID` | `auth.tenantId` | `""` | Entra tenant ID |

### Branding

Use these settings to customize product and visual identity in the Chat UI.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_APP_NAME` | `app.name` | `Edge AI Chat` | Application display name |
| `VITE_APP_TITLE` | `app.title` | `Edge AI Chat` | Browser tab title |
| `VITE_AGENT_NAME` | `agent.name` | `Edge AI Chat` | Agent/assistant display name |
| `VITE_APP_FAVICON` | `app.favicon` | `""` | Path to favicon (empty = none) |

The Agentic RAG Helm chart sets these values to `Agentic RAG` by default.

### Sidebar and conversation list

These settings control how users browse and manage conversation history.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_SIDEBAR_ENABLED` | `sidebar.enabled` | `true` | Show conversation list sidebar |
| `VITE_SIDEBAR_IS_OPEN` | `sidebar.isOpen` | `true` | Default open state (user preference persists to localStorage) |
| `VITE_SIDEBAR_SHOW_ICON` | `sidebar.showIcon` | `false` | Show brand icon in sidebar header |
| `VITE_SIDEBAR_ICON` | `sidebar.icon` | `""` | Path to header icon (placed in `/public`) |
| `VITE_CONVERSATIONS_INITIAL_LIMIT` | `conversations.initialLimit` | `50` | Initial fetch limit. `0` = let API default apply |

### Chat surface

Use these settings to control prompt input behavior and conversation routing.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_CHAT_MAX_LENGTH` | `chat.maxLength` | `4000` | Max characters per user prompt |
| `VITE_CHAT_SHOW_PROMPT_STARTERS` | `chat.showPromptStarters` | `false` | Show prompt starters on the welcome screen |
| `VITE_CHAT_USE_ROUTES` | `chat.useRoutes` | `true` | Use URL routes for conversations (`/conversation/:id`) |

### Message and new chat icons

By default, the app uses Fluent UI icons. You can also use custom scalable vector graphics (SVGs).

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_CHAT_SHOW_MESSAGE_ICON` | `chat.showMessageIcon` | `true` | Show assistant avatar |
| `VITE_CHAT_MESSAGE_ICON` | `chat.messageIcon` | `""` | Path to custom assistant icon (in `/public`) |
| `VITE_NEW_CHAT_SHOW_ICON` | `newChat.showIcon` | `true` | Show new‑chat icon |
| `VITE_NEW_CHAT_ICON` | `newChat.icon` | `""` | Path to custom new‑chat icon |

### Layout

These settings define how wide the main content area is and how cards are arranged.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_LAYOUT_MAX_WIDTH` | `layout.maxWidth` | `950px` | Max width of the main content area |
| `VITE_LAYOUT_CARD_COLUMNS` | `layout.cardColumns` | `2` | Card grid columns |

### Feature flags

Use feature flags to enable or disable optional conversation capabilities.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_FEATURES_RENAME_CONVERSATION` | `features.renameConversation` | `true` | Allow renaming conversations |
| `VITE_FEATURES_DELETE_CONVERSATION` | `features.deleteConversation` | `true` | Allow deleting conversations |
| `VITE_FEATURES_SEARCH` | `features.search` | `false` | Enable sidebar conversation search |

### Security classification banner

For regulated deployments. Empty values hide the banner.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_SECURITY_BANNER_TEXT` | `security.bannerText` | `""` | Classification text |
| `VITE_SECURITY_BANNER_COLOR` | `security.bannerColor` | `""` | One of `red`, `yellow`, `green`, `blue` |

### Copilot surface

These settings configure the visual mode used for Copilot-oriented UI elements.

| Env var | Config key | Default | Description |
|---|---|---|---|
| `VITE_COPILOT_MODE` | `copilot.mode` | `canvas` | `canvas` (larger) or `default` (compact) |
| `VITE_COPILOT_DESIGN_VERSION` | `copilot.designVersion` | `next` | Fluent UI Copilot design version |

### Configuration reference scope

The tables above cover the options most relevant to Agents and Tools with Foundry Local deployments. For complete deployment settings in this doc set, see [Deployment reference for Agents and Tools with Foundry Local](deploy-reference.md).

### Common deployment questions

Use this section to quickly map common questions to required Chat UI settings.

| Question | Short answer |
|---|---|
| How do I enable Chat UI for Agents and Tools with Foundry Local? | Set `VITE_API_MODE=agents`, `VITE_API_URL`, and `VITE_AGENT_ID`. |
| How do I host Chat UI on `/chat` instead of `/`? | Set `VITE_BASE_PATH=/chat` so startup injects the correct `<base href>`. |
| How do I enable Microsoft Entra ID sign-in? | Set Entra values such as `VITE_AUTH_CLIENT_ID` and `VITE_AUTH_TENANT_ID`. |
| How do I run disconnected without sign-in? | Omit auth env vars. The UI auto-disables auth and uses the network boundary as the trust boundary. |
| Which backend does this article target? | Agentic RAG mode (`VITE_API_MODE=agents`). |

## How the Chat UI fits in an Agents and Tools with Foundry Local deployment

In Agents and Tools with Foundry Local, the Chat UI acts as the browser-facing entry point while runtime and knowledge components handle execution, retrieval, and policy. This section explains the architecture shape, request flow, routing model, and identity path that make that separation work.

### Architecture

This diagram shows the high-level component relationships for the Chat UI deployment.

:::image type="content" source="./media/chat-web-ui/architecture.png" alt-text="Diagram that shows the Chat UI architecture flow from user to chat UI, agents-runtime, agent, and knowledge sources, with a return path back to the chat UI." lightbox="./media/chat-web-ui/architecture.png" border="false":::

The Chat UI is deployed as a separate pod in the same Kubernetes namespace as `agents-runtime`, behind the same ingress. The browser loads the single-page application (SPA) from `/` (or `/chat`) and calls the backend at `/runtime/*` on the same host. No Cross-Origin Resource Sharing (CORS) configuration is required.

### End-to-end chat flow

The following sequence shows how a user prompt moves through runtime services and streams back to the chat UI.

:::image type="content" source="./media/chat-web-ui/end-to-end-chat-flow.png" alt-text="Sequence diagram of the end-to-end chat flow from user to chat UI, agents-runtime, and agent plus tools, with server-sent events streamed back to the chat UI." lightbox="./media/chat-web-ui/end-to-end-chat-flow.png" border="false":::

### Ingress routing model

In a standard deployment, ingress routes two paths to two services on the same host:

```text
/            -> chat-ui-frontend  (port 80)     # Chat UI route
/runtime/*   -> agents-runtime    (port 8081)   # Already exists
```

Example ingress addition:

```yaml
- path: /
  pathType: Prefix
  backend:
    service:
      name: chat-ui-frontend
      port:
        number: 80
```

When you expose the Chat UI on a shared subpath (such as `/chat`), set `VITE_BASE_PATH=/chat`. The entrypoint injects `<base href="/chat/">` into the HTML so relative asset paths resolve correctly from deep SPA routes such as `/chat/conversation/thread_xxx`. The same image works at any subpath.

> **Why the same namespace and ingress are common:** The browser calls `/runtime` on the same host that serves the page. When Chat UI and `agents-runtime` share ingress, the relative URL resolves naturally. Cross-namespace deployments require extra ingress wiring.

### Health endpoint behavior

Use the health endpoint to verify container liveness and readiness.

```text
GET /health -> 200 "ok"
```

Use it for liveness and readiness:

```yaml
livenessProbe:
  httpGet: { path: /health, port: 80 }
  initialDelaySeconds: 5
  periodSeconds: 30
readinessProbe:
  httpGet: { path: /health, port: 80 }
  initialDelaySeconds: 3
  periodSeconds: 10
```

### Foundry Agents API surface used by the Chat UI

The following operations are the primary runtime API calls used by the Chat UI.

| Operation | Method | Path |
|---|---|---|
| Create thread | POST | `/threads` |
| List threads | GET | `/threads?limit=100` |
| Delete thread | DELETE | `/threads/{id}` |
| Create message | POST | `/threads/{id}/messages` |
| List messages | GET | `/threads/{id}/messages?order=asc&limit=N&after=ID` |
| Create run (streaming) | POST | `/threads/{id}/runs?stream=true` |


### Auth chain in connected deployments

This flow explains how user sign-in and token validation work in connected environments.

```text
User opens Chat UI
  -> Microsoft Authentication Library (MSAL) redirects to login.microsoftonline.com
  -> User authenticates with Entra ID
  -> Chat UI acquires access token
  -> Every API call: Authorization: Bearer <token>
  -> agents-runtime MISE sidecar validates token
  -> Extracts user object ID (oid claim) -> scopes threads to user
```

**Entra app registration**: the Chat UI reuses the RAG developer portal's app registration. Redirect URIs needed:

- `http://localhost:5173` (local dev, already registered)
- `https://<chat-ui-deployment-url>` (production, needs stable DNS; dynamic ingress IPs don't work)

**Role assignment**: users need `EdgeRAGEndUser` or `EdgeRAGDeveloper` assigned in the Entra enterprise app.

When you omit auth environment variables, MSAL isn't loaded, no auth headers are attached, and the network perimeter is the trust boundary (disconnected or air-gapped deployments).

## Integration checklist

Use this checklist to surface the Chat UI in an Agents and Tools with Foundry Local deployment.

- [ ] Add the Chat UI container to the Helm chart with the required env vars from §2.1.
- [ ] Configure ingress routing: `/` → chat‑ui, `/runtime` → agents‑runtime (§ Ingress routing).
- [ ] Set `VITE_AGENT_ID` to the agent ID returned by `agents-manager`.
- [ ] Set `VITE_BASE_PATH=/chat` when the UI is served from a shared subpath.
- [ ] For connected deployments: set `VITE_AUTH_CLIENT_ID` and `VITE_AUTH_TENANT_ID`; register the Chat UI URL as a redirect URI in the Entra app; assign users the `EdgeRAGEndUser` or `EdgeRAGDeveloper` role.
- [ ] Apply branding overrides if needed (`VITE_APP_NAME`, `VITE_APP_TITLE`, `VITE_AGENT_NAME`, favicon, sidebar icon).
- [ ] Apply the security banner env vars if the deployment is classified.
- [ ] Assign stable DNS to the Chat UI ingress.
- [ ] Validate `curl http://<chat-ui>/health` returns `200 "ok"`.
- [ ] Validate browser calls to `/runtime/threads` reach `agents-runtime` and return the expected payload.
- [ ] Confirm streaming works: send a prompt and verify assistant text streams in.
- [ ] Confirm titles: after the first run, the sidebar entry appears with a RAG‑generated title (or local fallback if BYOM is not configured).

## Related content

- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [Deploy Agents and Tools with Foundry Local](deploy.md)
- [Deployment reference for Agents and Tools with Foundry Local](deploy-reference.md)
- [What is Agentic Retrieval Augmented Generation (RAG)?](agentic-overview.md)
