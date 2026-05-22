---
title: How Chat UI Works in Agents and Tools with Foundry Local
description: Learn how chat UI works in Agents and Tools with Foundry Local and where it fits in architecture, identity, and deployment planning.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/22/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-assisted

#customer intent: As a platform engineer or solution architect, I want to understand how chat UI works in Agents and Tools with Foundry Local so that I can design and deploy a consistent local chat experience.
---

# Chat UI in Agents and Tools with Foundry Local

The Chat user interface (UI) is the web front end for Agents and Tools with Foundry Local. It gives users a chat surface to ask questions over configured knowledge sources and receive grounded responses with citations.

If you design or operate local artificial intelligence (AI) deployments, you need a consistent chat experience across connected and disconnected environments. Runtime and policy enforcement stay in backend services.

This article explains where the chat UI fits in the Agents and Tools with Foundry Local architecture, how runtime configuration and identity settings shape behavior, and how backend mode selection affects integration patterns.

## What the chat UI is

The chat UI is a ready-to-deploy web app that ships with Agents and Tools with Foundry Local. It gives you a Microsoft 365-style interface for interacting with an Agents and Tools with Foundry Local agent while keeping orchestration and policy enforcement in backend services.

When users open it in a browser they see:

- A **welcome screen** with an input box and optional prompt starters.
- A **sidebar with conversation history** that supports rename, delete, and optional search.
- An **assistant reply that streams incrementally**, including markdown, code blocks, and **inline citations/source chips** to grounded documents.
- **Sign-in with a work account** (Microsoft Entra ID) for connected deployments, with user-scoped conversation history.
- The same experience in **disconnected or air-gapped** deployments, where sign-in is skipped and the network boundary becomes the trust boundary.

### User experience capabilities

The following capabilities define what end users can do in the chat UI.

| Area | What end users get |
|---|---|
| Conversations | Multi-turn chat, persisted as threads; switch between threads in the sidebar |
| Streaming | Assistant answers stream in increments |
| Grounding | Citations and source chips rendered alongside the assistant message |
| History | Rename, delete, and optionally search past conversations |
| Titles | Sidebar entries get a backend-generated title after the first reply (Microsoft 365-style deferred title), with local fallback when title generation is unavailable |
| Welcome | Optional prompt starters to nudge first-time users |
| Identity | Microsoft Entra ID sign‑in on connected deployments; auto‑disabled when no client ID is configured |

### What deployment teams can control

The chat UI is configuration-driven at deploy time. The same container image is reused everywhere, and behavior is toggled by `VITE_*` environment variables on the pod:

- **Branding**, app name, browser tab title, agent display name, favicon, sidebar header icon, assistant avatar, and new chat icon.
- **Layout**, max content width, card grid columns, and subpath where the UI is served (`/`, `/chat`, ...).
- **Sidebar and history**, sidebar visibility, default open state, initial conversation fetch limit, rename, delete, and search.
- **Chat surface**, max prompt length, welcome-screen prompt starters, URL routes for conversations.
- **Security banner**, classification banner text and color for regulated deployments.
- **Authentication**, Entra client ID, tenant, authority, and redirect URI (Uniform Resource Identifier); omit to disable auth entirely.
- **Backend wiring**, backend mode, runtime URL, agent ID.

### Where it fits in Agents and Tools with Foundry Local

The chat UI is one deployable component of Agents and Tools with Foundry Local, alongside the agent runtime and knowledge sources. It's the user-facing surface only. It doesn't directly call models, orchestrate tools, validate tokens, enforce role-based access control (RBAC), or scope data. Those responsibilities remain with `agents-runtime`, the MISE sidecar, the agent, and knowledge source services.

### Supported backend modes

The same image supports several backend modes. Agentic retrieval-augmented generation (RAG) for Agents and Tools with Foundry Local is the default and the focus of this article. The other modes exist for adjacent scenarios.

| Mode | `VITE_API_MODE` | Use case |
|---|---|---|
| Agentic RAG | `agents` | Local or Arc-enabled Agents and Tools with Foundry Local deployments that run `agents-runtime` |
| Server + bring your own model (BYOM) | `server` | Teams that need a backend adapter, mutual Transport Layer Security (mTLS), or server-held API keys. See your solution's provider documentation. |
| Front-end direct completions | `completions` | Browser-direct call to an OpenAI-compatible endpoint (for example, Foundry Local). See your solution's provider documentation. |

## How configuration works

All chat UI configuration is injected via `VITE_*` environment variables at container startup. The entrypoint writes them into `window.__RUNTIME_CONFIG__` at runtime, so the same image is reused across environments and no rebuild is needed.

Three layers apply in order:

| Order | Layer | Notes |
|---|---|---|
| 1 | Defaults | Hardcoded baseline values |
| 2 | `VITE_*` environment variables | Overrides defaults at startup (Helm values, Docker, or `kubectl set env`) |
| 3 | Derived values | Computed after merge (currently only `auth.enabled`) |

### What you configure for chat UI

To surface chat UI, plan these configuration categories:

- Backend wiring so the UI calls `agents-runtime` by using the selected mode and agent ID.
- Hosting and routing so the app path and runtime path are reachable on the intended host.
- Identity mode for connected environments (Entra ID) or disconnected environments (network boundary).
- Experience settings such as branding, sidebar behavior, prompt starters, feature toggles, and classification banner.

For complete variable and API details, see [Chat UI configuration and API reference](chat-experience-reference.md).

## Surface chat UI: high-level flow

Use this sequence to plan chat UI rollout without going into implementation steps:

1. Choose where chat UI is hosted (root path or subpath).
1. Configure runtime connectivity (`agents` mode, runtime URL, and agent ID).
1. Decide identity model (connected Entra ID or disconnected mode).
1. Apply UX and policy settings (branding, feature flags, security banner).
1. Validate health, runtime connectivity, and streaming behavior.

For step-by-step deployment and validation, see [Deploy and validate Chat UI for Agents and Tools with Foundry Local](chat-experience-deploy.md).

## How the chat UI fits in an Agents and Tools with Foundry Local deployment

In Agents and Tools with Foundry Local, the chat UI acts as the browser-facing entry point while runtime and knowledge components handle execution, retrieval, and policy. This section explains the architecture shape, request flow, routing model, and identity path that make that separation work.

### Architecture

This diagram shows the high-level component relationships for the chat UI deployment.

:::image type="content" source="./media/chat-web-ui/architecture.png" alt-text="Diagram that shows the chat UI architecture flow from user to chat UI, agents-runtime, agent, and knowledge sources, with a return path back to the chat UI." lightbox="./media/chat-web-ui/architecture.png" border="false":::

The chat UI is deployed as a separate pod in the same Kubernetes namespace as `agents-runtime`, behind the same ingress. The browser loads the single-page application (SPA) from `/` (or `/chat`) and calls the backend at `/runtime/*` on the same host. No Cross-Origin Resource Sharing (CORS) configuration is required.

### End-to-end chat flow

The following sequence shows how a user prompt moves through runtime services and streams back to the chat UI.

:::image type="content" source="./media/chat-web-ui/end-to-end-chat-flow.png" alt-text="Sequence diagram of the end-to-end chat flow from user to chat UI, agents-runtime, and agent plus tools, with server-sent events streamed back to the chat UI." lightbox="./media/chat-web-ui/end-to-end-chat-flow.png" border="false":::

### Connected identity path

In connected deployments, the browser acquires a user token through Entra ID and sends it to `agents-runtime`, where token validation and user scoping occur.

In disconnected or air-gapped deployments, chat UI runs without Entra sign-in and the network boundary becomes the trust boundary.

For implementation steps and required identity settings, see [Deploy and validate Chat UI for Agents and Tools with Foundry Local](chat-experience-deploy.md) and [Chat UI configuration and API reference](chat-experience-reference.md).

## Related content

- [Deploy and validate Chat UI for Agents and Tools with Foundry Local](chat-experience-deploy.md)
- [Chat UI configuration and API reference](chat-experience-reference.md)
- [Deployment overview for Agents and Tools with Foundry Local](deploy-overview.md)
- [What is Agentic Retrieval Augmented Generation (RAG)?](agentic-overview.md)