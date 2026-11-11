---
title: Agents REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for agents, agent versions, and connections in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/09/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
---

# Agents REST API reference

Use this reference to manage agents, agent versions, and connections in Agents and Tools with Foundry Local. For step-by-step guidance, see [Create and manage agents](../create-manage-agents.md) and [Add tools to an agent](../add-agent-tools.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Base address and authorization

- **Base address:** `https://<cluster-domain>`.
- **API version:** Optional query parameter. If you send `api-version`, its value must be `v1`.
- **Authorization:** Send a Microsoft Entra ID access token issued for the app registration that protects Agents and Tools with Foundry Local in the `Authorization: Bearer <token>` header. Get a token with `az account get-access-token --resource api://<app-registration-client-id>`.

| Azure role at the extension scope | Operations | App-role fallback |
|---|---|---|
| **Reader** | List agents and get an agent | `EdgeRAGDeveloper` or `EdgeRAGEndUser` |
| **Reader** | Read agent versions, connections, or the built-in tool catalog | `EdgeRAGDeveloper` |
| **Cognitive Services Contributor** | Create, change, enable, disable, or delete resources | `EdgeRAGDeveloper` |

Azure Resource Manager checks Azure roles first. Read actions use `Microsoft.CognitiveServices/accounts/<resource>/read`; changes use the resource's `write` or `delete` action. **Contributor** and **Owner** also allow management operations. In disconnected deployments, use **Reader** for reads and **Contributor** for changes. For setup, see [Assign Azure roles](../prepare-authentication.md#assign-azure-roles).

`GET /agents` and `GET /agents/{name}` return full definitions only when the token includes the `EdgeRAGDeveloper` app role. Other allowed callers, including Azure-role-only callers, get enabled agents only, with `id`, `name`, and `description`.

The extension's managed identity needs **Reader** at the extension scope to check Azure roles. App-only callers can use Azure roles without a `roles` claim. For fallback details, see [App roles](../agent-service-security.md#app-roles).

Errors from API handlers use `{"error": {"code": "...", "message": "...", "param": "..."}}`. Authentication and authorization errors use `{"detail": "<message>"}`. See [Sign-in and access errors](../troubleshoot-agent-service.md#sign-in-and-access-errors).

## Agents

| Operation | Method and path | Success |
|---|---|---|
| List agents | `GET /agents` | `200` |
| Get an agent | `GET /agents/{agent_name}` | `200` |
| Create a version (and the agent, if needed) | `POST /agents/{agent_name}/versions` | `200` |
| List versions | `GET /agents/{agent_name}/versions` | `200` |
| Get a version | `GET /agents/{agent_name}/versions/{version}` | `200` |
| Select the version that answers users | `PATCH /agents/{agent_name}` | `200` |
| Enable an agent | `POST /agents/{agent_name}:enable` | `204` |
| Disable an agent | `POST /agents/{agent_name}:disable` | `204` |
| Delete a version | `DELETE /agents/{agent_name}/versions/{version}` | `200` |
| Delete an agent and all its versions | `DELETE /agents/{agent_name}` | `200` |
| List built-in tool capabilities | `GET /built-in-tools` | `200` |

There's no separate request to create an agent. Creating the first version creates the agent. For details about versions, see [Agent versions](#agent-versions).

### List agents query parameters

| Name | Type | Description |
|---|---|---|
| `kind` | string | Filter by `prompt` or `prompt-edge`. |
| `limit` | integer | 1 to 100. Default 20. |
| `order` | string | `asc` or `desc`. Default `desc`. |
| `after` | string | Return agents after this ID. Can't be used with `before`. |
| `before` | string | Return agents before this ID. |

The response is a list object: `{"object": "list", "data": [...], "first_id": "...", "last_id": "...", "has_more": false}`.

### Agent object

| Property | Type | Description |
|---|---|---|
| `object` | string | `agent` |
| `id` | string | The agent name. |
| `name` | string | The agent name, in lowercase. |
| `state` | string | `enabled` or `disabled`. |
| `versions.latest` | object | The newest [agent version](#agent-version-object). |
| `agent_endpoint.version_selector` | object | The version that answers users. See the next section. |

### Select a version request body

```json
{
  "agent_endpoint": {
    "version_selector": {
      "version_selection_rules": [
        { "type": "FixedRatio", "agent_version": "2", "traffic_percentage": 100 }
      ]
    }
  }
}
```

| Property | Required | Description |
|---|---|---|
| `version_selection_rules` | Yes | Exactly one rule. |
| `type` | Yes | `FixedRatio`. |
| `agent_version` | Yes | A version number, such as `2`, or `@latest`. |
| `traffic_percentage` | Yes | `100`. |

### Agent name rules

Agent names are 1 to 63 characters long. They use letters, digits, and hyphens, and must start and end with a letter or digit. Names aren't case-sensitive. The service stores them in lowercase.

## Agent versions

The [Agents](#agents) table lists the agent version operations. The **List versions** operation supports the same `limit`, `order`, `after`, and `before` parameters as **List agents**.

You can't delete a version while a response runs with it, or delete the newest version while older versions exist. These requests return `409`.

### Create a version request body

| Property | Type | Required | Description |
|---|---|---|---|
| `description` | string | No | Up to 512 characters. |
| `metadata` | object | No | Up to 16 string key-value pairs. Keys up to 64 characters, values up to 512 characters. |
| `definition` | object | Yes | See the next table. |

### Definition properties

| Property | Type | Required | Description |
|---|---|---|---|
| `kind` | string | Yes | `prompt` or `prompt-edge`. |
| `model` | string | Yes | `<model-connection-name>/<model-name>`. |
| `instructions` | string | No | System instructions for the agent, up to 87,381 characters. |
| `temperature` | number | No | `0` to `2`. |
| `top_p` | number | No | `0` to `1`. |
| `tools` | array | No | Up to 32 [MCP tools](#mcp-tool-properties) or [built-in local tools](#built-in-tool-declarations). |
| `tool_choice` | string | No | `auto` or `none`. |
| `text.format` | object | No | `{"type": "text"}`, `{"type": "json_object"}`, or `{"type": "json_schema", "name": "...", "schema": {...}, "strict": true}`. |

### MCP tool properties

| Property | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | `mcp`. |
| `server_label` | string | Yes | Unique within the version, up to 64 printable characters. |
| `server_description` | string | No | Up to 512 characters. For knowledge base tools, the service sets it from the knowledge base description. |
| `project_connection_id` | string | One of two | The name of an existing MCP connection, or `kb-<knowledge-base-name>`. |
| `connection` | object | One of two | Settings for a new [MCP connection](#mcp-connection-properties). The service creates it. |
| `allowed_tools` | object | No | `{"tool_names": ["..."]}`. Up to 128 unique tool names. |
| `require_approval` | string | No | `never`. This is the only supported value. |

### Built-in tool declarations

The `definition.tools` schema includes the Microsoft 365 Local built-in tool types `exchange_local` and `sharepoint_local`, and `GET /built-in-tools` lists their tool names and schemas (`{"value": [{"type": "...", "tools": [...]}], "nextLink": null}`). Microsoft 365 Local is in limited preview for approved customers, and it's supported only with Agentic Retrieval. It isn't supported as a general Agent Service integration, so don't declare these tool types on an agent directly. Instead, create `sharepoint` or `exchange` knowledge sources, add them to a knowledge base, and add the knowledge base to the agent as an `mcp` tool. See [Connect Microsoft 365 Local](../connect-microsoft-365-local.md) and [Add a knowledge base to an agent](../add-agent-tools.md#add-a-knowledge-base-to-an-agent).

### Agent version object

| Property | Type | Description |
|---|---|---|
| `object` | string | `agent.version` |
| `id` | string | `<agent-name>:<version>` |
| `name` | string | The agent name. |
| `version` | string | The version number, such as `1`. |
| `description` | string | The description. |
| `created_at` | integer | Creation time, in Unix seconds. |
| `definition` | object | The definition, as stored. |
| `metadata` | object | The metadata. |
| `status` | string | `active`. |

## Connections

Connections store how to reach a language model or an MCP server.

| Operation | Method and path | Success |
|---|---|---|
| List connections | `GET /connections` | `200` |
| Get a connection | `GET /connections/{connection_name}` | `200` |
| Create or replace a connection | `PUT /connections/{connection_name}` | `200` |
| Update part of a connection | `PATCH /connections/{connection_name}` | `200` |
| Delete a connection | `DELETE /connections/{connection_name}` | `204` |

Connection names are 1 to 63 lowercase letters, digits, and hyphens, and start and end with a letter or digit. Responses never include API keys. Don't change or delete connections that the platform manages, such as `kb-<knowledge-base-name>`. Change the knowledge base instead. You can't delete an MCP connection that an agent version uses (`409`). The service doesn't block you from deleting a model connection that agents or knowledge bases use, so check before you delete one.

`GET /connections` accepts optional query parameters (`type` to filter by `RemoteTool_Preview` or `ModelConnection`, `limit` from 1 to 100 with default 50, `continuation`, and `target-api=foundry`). The list response is `{"value": [...], "nextLink": "<url-or-null>"}`. When stored connections are skipped because their authentication is unsupported or their configuration is invalid, `GET /connections` includes the count in the `x-edge-rag-skipped-connections` response header.

`GET`, `PUT`, and `PATCH` connection responses return `name` and `id` on all connections. For `RemoteTool_Preview` connections (`ConnectionResponse`), responses return `credentials: {"type": "AAD" | "ApiKey" | "None"}` instead of the write-side `authType` field, and include `managed: true` when the connection is platform-managed.

### Model connection properties

| Property | Type | Required | Description |
|---|---|---|---|
| `type` | string | Yes | `ModelConnection`. |
| `kind` | string | Yes | `FoundryOnArc`, `FoundryCloud`, or `OpenAICompatible`. |
| `endpoint` | string | Yes | HTTPS base address of the model API, ending with `/v1/`. |
| `model` | string | Yes | Model or deployment name, up to 253 characters, without `/`. |
| `authentication.type` | string | Yes | `ManagedIdentity` or `ApiKey`. `OpenAICompatible` requires `ApiKey`. |
| `authentication.audience` | string | No | `ManagedIdentity` only. Token audience. |
| `authentication.apiKey` | string | For `ApiKey` | Write-only. |
| `maxContextTokens` | integer | Yes | 4,096 to 10,000,000. |
| `isReasoningModel` | boolean | No | Default `false`. |
| `reasoningEffort` | string | No | Requires `isReasoningModel: true`. |

`ManagedIdentity` requires a DNS host name that a cluster administrator approved in the `serviceUrls` extension setting.

### MCP connection properties

| Property | Type | Required | Description |
|---|---|---|---|
| `type` | string | No | `RemoteTool_Preview` (default). |
| `target` | string | Yes | HTTPS address of the MCP server, up to 2,048 characters. |
| `authType` | string | Yes | `AAD`, `ApiKey`, or `None`. |
| `tokenScope` | string | For `AAD` | Scope that ends with `/.default`. The service requests a token for it with its managed identity. |
| `apiKey` | string | For `ApiKey` | Write-only. |
| `apiKeyHeaderName` | string | For `ApiKey` | Header name for the key. Only valid with `ApiKey`. |
| `apiKeyValueFormat` | string | For `ApiKey` | `Raw` or `Bearer`. Only valid with `ApiKey`. |
| `isDefault` | boolean | No | Default `false`. |
| `metadata` | object | No | Up to 16 string key-value pairs. |

`AAD` requires a server address that a cluster administrator approved in the `serviceUrls` extension setting.

## Related content

- [Responses and Conversations REST API reference](responses-api-reference.md)
- [Knowledge Bases REST API reference](knowledge-bases-api-reference.md)
- [Agent Service overview](../agentic-overview.md)
