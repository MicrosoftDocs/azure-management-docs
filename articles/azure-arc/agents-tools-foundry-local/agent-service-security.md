---
title: Agent Service Security and Access - Agents and Tools with Foundry Local
description: Learn how Agent Service in Agents and Tools with Foundry Local signs in users, checks Azure roles, protects credentials, and limits where tokens go.
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/09/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As a platform engineer or security reviewer, I want to understand how Agent Service handles identities, roles, and credentials so that I can deploy it safely.
---

# Security and access for Agent Service in Agents and Tools with Foundry Local

This article explains how Agent Service in Agents and Tools with Foundry Local protects your agents and data. It covers who can call the APIs, which identity the platform uses when it calls models and tools, how it stores credentials, and how it keeps one user's conversations away from other users.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Identities

Three kinds of identities take part when an agent runs:

| Identity | What it is | When the platform uses it |
|---|---|---|
| **User** | A person or app that signs in with Microsoft Entra ID and calls the APIs with an access token. | Azure roles decide access first. Supported APIs can use app roles as a fallback. The object ID (`oid`) identifies the caller who owns conversations and responses. |
| **Extension managed identity** | The identity of the Agents and Tools with Foundry Local extension on your Azure Arc-enabled cluster. | Reads Azure role assignments for API authorization. Also gets tokens when a model connection or MCP connection uses managed identity (`ManagedIdentity` or `AAD`). |
| **Stored API key** | A key that a developer saved in a connection. | When a model connection or MCP connection uses `ApiKey`. |

When an agent searches a [knowledge base](knowledge-bases-guide.md), the platform passes the *user's* identity to the search. A caller allowed by an Azure role can access every collection. App-role fallback keeps the per-collection app-role filter. For more information, see [Collections overview](collections-overview.md).

## Permission checks

When you configure authentication, permission to call Agent Service is separate from permission to access a model or MCP server.

Azure role-based access control (Azure RBAC) is the primary access model. An administrator assigns Azure roles at the extension resource scope. Assignments inherited from the subscription, resource group, or cluster also apply.

| Operation | Identity or credential | Required access |
|---|---|---|
| Read resource settings and list enabled agents | API caller | **Reader**. Most management reads accept `EdgeRAGDeveloper` as a fallback; listing or getting agents and connection metadata accept either app role. |
| Create, change, or delete agents, knowledge bases, and connections | API caller | **Cognitive Services Contributor**. **Contributor** or **Owner** also allow management operations. Fallback: `EdgeRAGDeveloper`. |
| Run agents and manage your Responses and Conversations | API caller | **Cognitive Services OpenAI User**. No app-role fallback. The Local chat UI uses these APIs. |
| Read role definitions, role assignments, and deny assignments | Extension managed identity | **Reader** on the extension resource. Required for the Azure role check, regardless of the model's sign-in method. |
| Call a model | Credential configured in the model connection | Access to the model endpoint. |
| Call an MCP server | Credential configured in the MCP connection, or the caller's token for a `microsoft_entra_id` knowledge source | Access required by the server. Managed identity (`AAD`) connections require an approved destination in `mi-destinations`; `microsoft_entra_id` user-token forwarding is allowed only to cluster-internal MCP endpoints. |

In disconnected deployments, **Reader** allows management reads and **Contributor** allows changes, chat, Responses, and Conversations. The extension maps data permissions to control-plane permissions on the stamp's local Azure Resource Manager endpoint.

The service checks Azure roles through Azure Resource Manager first. If Resource Manager doesn't allow the request, it checks the route's app-role fallback. If neither allows it, the service returns `403` for a denied role check, or `503` when Resource Manager is unavailable.

Within the Resource Manager check, a deny assignment blocks a permission even if a role grants it. Role assignments with conditions add extra rules that the service doesn't evaluate, so they don't grant access. If only a conditional assignment matches the required permission, the Resource Manager check is unavailable. These results still allow app-role fallback on routes that support it.

Assigning an API role doesn't grant access to downstream models or tools. For setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles). For managed identity endpoint approval, see [Approved endpoints for managed identity](#approved-endpoints-for-managed-identity).

## App roles

Assign these Microsoft Entra ID app roles on the app registration for Agents and Tools with Foundry Local. For setup steps, see [Configure authentication](prepare-authentication.md).

> [!IMPORTANT]
> Azure roles are the primary model. App roles remain a fallback for most management and classic APIs, but not for Responses or Conversations. An app role alone no longer lets you run agents.

| Role | Who gets it | What they can do |
|---|---|---|
| `EdgeRAGDeveloper` | Developers and administrators | Fallback access to manage resources. Required for full definitions from agent list and get operations, even when an Azure role allows access. |
| `EdgeRAGEndUser` | People who need supported read or classic query access | Fallback access to list and get enabled agent summaries and read connection metadata. Classic queries also require matching collection app roles. Doesn't grant Responses or Conversations access. |

A request without a valid token returns `401 Unauthorized`. Service principals and managed identities can use Azure roles without a `roles` claim, but the token audience must still be the extension's app registration.

## Conversation and response ownership

Conversations and responses belong to the caller who created them, identified by the `oid` claim in the access token. Stored responses also use the tenant ID (`tid`). Callers can access only their own resources. A request for another caller's conversation or a response with a different owner or tenant returns `404 Not Found`.

## Credentials

- **API keys are write-only.** When you save a connection with an API key, the platform stores the key in a Kubernetes secret. The API never returns it. `GET /connections` shows only the authentication type.
- **Managed identity tokens are short-lived.** The platform gets tokens when it needs them, and doesn't store them in connections or agent versions.
- **User tokens stay with trusted servers.** The platform sends the user's token only to Agentic Retrieval and to knowledge sources that use `microsoft_entra_id`. MCP connections with `AAD` get a managed identity token instead, not the user's token.

## Approved endpoints for managed identity

The platform only sends managed identity tokens to addresses that a cluster administrator approved. This rule covers model connections that use `ManagedIdentity`, and MCP connections that use `AAD`, including Microsoft Foundry toolboxes.

- Approved addresses are HTTPS base addresses with a DNS name. IP addresses aren't allowed.
- An address approves itself and every path under it.
- The host name and port must match exactly, and wildcards aren't supported. Each Foundry Local model has its own host name, so approve the address of each model that you use.
- The ConfigMap `mi-destinations` in the namespace `model-access-<extension-namespace>` stores the list. Only Kubernetes users who you allow to edit that ConfigMap can change it.

For steps, see [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity).

## Network checks

Before the platform connects to a model or an MCP server, it checks the address. Model connections and MCP connections must use HTTPS addresses. The platform pins the network address that it checked, and doesn't follow redirects to other addresses. A remote MCP server that fails these checks can't be saved as a knowledge source, and a tool call to it fails.

## Recommendations

- Assign the narrowest Azure roles at the extension scope. Review inherited assignments: subscription, resource-group, and cluster grants can also allow API access.
- Give chat users **Reader** to list agents and **Cognitive Services OpenAI User** to run them. In disconnected deployments, use **Contributor** for chat.
- Keep `EdgeRAGDeveloper` assignments limited to people who need fallback management access or full agent definitions. Azure-role-authorized collection access has no per-collection filter.
- Use managed identity instead of API keys where your model or MCP server supports it.
- Keep the approved endpoint list as short as possible. Approve the narrowest path that works.
- Use `allowed_tools` in agent versions so each agent can only call the tools it needs.
- Only connect agents to MCP servers that you trust. A tool can return text that tries to change the agent's behavior.
- Delete connections, inline connections, and knowledge sources that you no longer use.

## Related content

- [Agent Service overview](agentic-overview.md)
- [Add tools to an agent](add-agent-tools.md)
- [Configure authentication](prepare-authentication.md)
- [Agent Service limits](reference/agent-service-limits.md)
