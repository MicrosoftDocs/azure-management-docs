---
title: Responses API Reference - Agents and Tools with Foundry Local
description: REST API reference for the OpenAI-compatible Responses and Conversations APIs in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Responses and Conversations REST API reference

Use this reference to run agents and manage conversations in Agents and Tools with Foundry Local. The APIs follow the shape of the OpenAI Responses and Conversations APIs. For step-by-step guidance, see [Run agents with the Responses API](../run-agents-responses-api.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Base address and authorization

- **Base address:** `https://<cluster-domain>/openai/v1`.
- **API version:** Optional query parameter. If you send `api-version`, its value must be `v1`.
- **Authorization:** Send a Microsoft Entra ID access token issued for the app registration that protects Agents and Tools with Foundry Local in the `Authorization: Bearer <token>` header.
- **Azure role:** **Cognitive Services OpenAI User** at the extension resource scope. In disconnected deployments, use **Contributor**. These APIs require an Azure role; there's **no app-role fallback**.
- **Permissions:** Responses use `Microsoft.CognitiveServices/accounts/OpenAI/responses/read`, `write`, and `delete` data actions. Conversations use the corresponding `Microsoft.CognitiveServices/accounts/OpenAI/responses/conversations/` actions.
- **Ownership:** Conversations and responses belong to the caller identified by the `oid` claim. Stored responses also check the tenant ID (`tid`). A different owner or response tenant returns `404`.

`EdgeRAGEndUser` or `EdgeRAGDeveloper` alone doesn't grant access. App-only tokens from service principals and managed identities can use Azure roles without a `roles` claim. If you also list agents through `GET /agents`, assign **Reader** or use a supported app-role fallback for that management route.

The extension's managed identity needs **Reader** at the extension scope to check role assignments. If the check is unavailable, these APIs return `503` with `{"detail": "Authorization service unavailable"}`. For setup, see [Assign Azure roles](../prepare-authentication.md#assign-azure-roles).

Errors from API handlers use `{"error": {"code": "...", "message": "...", "param": "..."}}`. Authentication and authorization errors use `{"detail": "<message>"}`. See [Sign-in and access errors](../troubleshoot-agent-service.md#sign-in-and-access-errors).

## Responses

| Operation | Method and path | Success |
|---|---|---|
| Create a response | `POST /responses` | `200`, or an SSE stream when `stream` is `true` |
| List responses | `GET /responses` | `200` |
| Get a response | `GET /responses/{response_id}` | `200` |
| List the input items of a response | `GET /responses/{response_id}/input_items` | `200` |
| Cancel a response | `POST /responses/{response_id}/cancel` | `200` |
| Delete a response | `DELETE /responses/{response_id}` | `200` |

### Create a response request body

| Property | Type | Required | Description |
|---|---|---|---|
| `agent_reference` | object | One of two | `{"type": "agent_reference", "name": "<agent-name>", "version": "<version>"}`. `version` is optional. If you leave it out, the agent's selected version runs. |
| `model` | string | One of two | `<model-connection-name>/<model-name>`. Calls the model without an agent. |
| `input` | string or array | Yes, unless `conversation` is set | A string up to 128,000 characters, 1 to 200 message items, or a single `mcp_approval_response` item when resuming from `previous_response_id`. |
| `conversation` | string or object | No | A conversation ID, or `{"id": "<conversation-id>"}`. Can't be used with `previous_response_id`. |
| `previous_response_id` | string | No | Continue from an earlier response that isn't part of a conversation. Can't be used with `conversation`. |
| `stream` | boolean | No | `true` returns Server-Sent Events. |
| `store` | boolean | No | Default `true`. |
| `instructions` | string | No | Up to 4,096 characters. |
| `temperature` | number | No | `0` to `2`. |
| `top_p` | number | No | `0` to `1`. |

The `background`, `tools`, `text`, and `metadata` properties aren't supported and return `400`.

### Input message item

```json
{
  "type": "message",
  "role": "user",
  "content": [ { "type": "input_text", "text": "What is our travel policy?" } ]
}
```

`content` can also be a plain string. Only text input is supported.

To approve or reject a paused tool call (`mcp_approval_request`), send a single `mcp_approval_response` item in `input` together with `previous_response_id`:

```json
{
  "type": "mcp_approval_response",
  "approval_request_id": "<approval-request-id>",
  "approve": true
}
```

### Response object

| Property | Type | Description |
|---|---|---|
| `id` | string | The response ID. |
| `object` | string | `response` |
| `created_at` | integer | Creation time, in Unix seconds. |
| `status` | string | `queued`, `in_progress`, `completed`, `incomplete`, `failed`, or `cancelled`. |
| `model` | string | The model that answered. |
| `output` | array | Output items. See the next table. |
| `usage` | object | `input_tokens`, `output_tokens`, and `total_tokens`. |
| `error` | object | `code` (`server_error`, `rate_limit_exceeded`, or `invalid_prompt`) and `message`, when `status` is `failed`. |
| `incomplete_details` | object | `reason` (`max_output_tokens` or `content_filter`), when `status` is `incomplete`. |
| `warnings` | array | Tool problems that didn't stop the response. Each item has `code`, `server_label`, `message`, and optional `connection_id`. Codes: `mcp_initialization_failed`, `mcp_authentication_failed`, `mcp_tools_unavailable`, `mcp_tool_validation_failed`, `mcp_call_failed`, `mcp_toolbox_inspection_failed`, `mcp_policy_refresh_failed`. |

### Output item types

| Type | Description |
|---|---|
| `message` | The assistant's answer. `content` holds `output_text` parts with `text` and `annotations`. |
| `mcp_list_tools` | The tools that the agent found on an MCP server. |
| `mcp_call` | One tool call, with `server_label`, `name`, `arguments` (a JSON string), `status`, and `output` or `error`. |
| `mcp_approval_request` | A tool call waiting for user approval, with `id`, `server_label`, `name`, and `arguments` (a JSON string). Resume by sending an `mcp_approval_response` input item with `previous_response_id`. |

### Tool errors

A tool failure doesn't always fail the response. Check each `mcp_call` item's `status` and `error`, the response's `warnings`, and the overall response `status` before using an answer that depends on a tool.

A failed knowledge base call can produce a generic error in `mcp_call.error` or the agent's answer rather than the underlying source's exact error. For example, the PostgreSQL and MySQL SQL aggregation helper returns no partial aggregate when its scan exceeds a budget. Don't treat a failed or incomplete scan as a complete total. Narrow the supported filters to reduce the scan, and retry. See [SQL aggregation scan budgets](../connect-sql-database.md#aggregation-for-postgresql-and-mysql).

### Citation annotation

| Property | Type | Description |
|---|---|---|
| `type` | string | `file_citation` |
| `text` | string | The marker in the answer text, such as `[cite:0]`. |
| `start_index`, `end_index` | integer | The position of the marker in the text, or `-1` if the citation has no marker. |
| `file_citation.file_id` | string | The source document. |
| `file_citation.quote` | string | A short quote from the source. |
| `file_path` | string | The source document path. |
| `title` | string | The document title. |
| `document_type` | string | The document type, such as `PDF`. |
| `page_numbers` | array | Pages where the content appears. |
| `chunk_id` | string | The position of the chunk in the document. |
| `score` | number | The relevance score. |
| `collection` | string | Where the content came from. |
| `last_modified` | number | Last-modified time, in Unix seconds. |
| `web_url` | string | A link to the source, when available. |

### Streaming events

When `stream` is `true`, the service sends these events. Each event's `data` is a JSON object.

| Event | Description |
|---|---|
| `response.created` | The response was created. |
| `response.in_progress` | The response started. |
| `response.output_item.added`, `response.output_item.done` | An output item started or finished. |
| `response.content_part.added`, `response.content_part.done` | A content part started or finished. |
| `response.output_text.delta`, `response.output_text.done` | A piece of answer text, or the complete text of a part. |
| `response.mcp_list_tools.in_progress`, `response.mcp_list_tools.completed`, `response.mcp_list_tools.failed` | Tool discovery on an MCP server. |
| `response.mcp_call.in_progress`, `response.mcp_call_arguments.delta`, `response.mcp_call_arguments.done`, `response.mcp_call.completed`, `response.mcp_call.failed` | A tool call. |
| `conversation.title.updated` | The conversation title was generated or updated during the stream. |
| `response.completed`, `response.incomplete`, `response.failed` | The response ended. If the response is cancelled, the stream ends without one of these events. |

### List responses query parameters

| Name | Type | Description |
|---|---|---|
| `limit` | integer | 1 to 100. Default 20. |
| `order` | string | `asc` or `desc`. Default `desc`. |
| `after`, `before` | string | Paging cursors. |
| `agent_name` | string | Only responses from this agent. |
| `agent_id` | string | Only responses from this agent version, as `<agent-name>:<version>`. |
| `conversation_id` | string | Only responses in this conversation. |
| `output_item_limit` | integer | Maximum number of output items to return for each response in the list. |
| `$filter` | string | Join expressions with `and`. Use single quotes for strings. Supported expressions: `status` or `id` with `eq` or `ne` (for example `status eq 'failed'`); `created_at` with `eq`, `ne`, `gt`, `ge`, `lt`, or `le` and an ISO 8601 time (for example `created_at ge '2026-10-01T00:00:00Z'`); `status in ('failed','cancelled')` or `id in (...)`; and `contains(id, '...')`, `startswith(id, '...')`, or `endswith(id, '...')`. |

### Cancel a response

You can cancel only a response that's running. The service waits up to 10 seconds for the agent to stop, and returns the response with any partial output. A response that already finished returns `400`. If the service can't confirm the cancellation in time, it returns `504`.

### Delete a response

`DELETE /responses/{response_id}` deletes a stored response that belongs to the caller and returns `{"id": "<response-id>", "object": "response.deleted", "deleted": true}`. Deleting a response that's still running (`in_progress` or `queued`) returns `409 Conflict`.

## Conversations

| Operation | Method and path | Success |
|---|---|---|
| Create a conversation | `POST /conversations` | `200` |
| List conversations | `GET /conversations` | `200` |
| Get a conversation | `GET /conversations/{conversation_id}` | `200` |
| Replace conversation metadata | `POST /conversations/{conversation_id}` | `200` |
| Delete a conversation | `DELETE /conversations/{conversation_id}` | `200` |
| Add items | `POST /conversations/{conversation_id}/items` | `200` |
| List items | `GET /conversations/{conversation_id}/items` | `200` |
| Get an item | `GET /conversations/{conversation_id}/items/{item_id}` | `200` |
| Delete an item | `DELETE /conversations/{conversation_id}/items/{item_id}` | `200` |

### Create a conversation request body

| Property | Type | Required | Description |
|---|---|---|---|
| `metadata` | object | No | Up to 16 string key-value pairs. Keys up to 64 characters, values up to 512 characters. |
| `items` | array | No | Up to 20 message items to start the conversation. |

Conversation IDs start with `conv_`. A conversation can hold up to 2,000 items. When an agent runs in a conversation, it reads the most recent 100 items.

### Conversation items you can add

You can add only `message` items:

- `role: "user"` with `input_text` content.
- `role: "assistant"` with `output_text` content.

The API doesn't support system, developer, or tool roles, or images, files, and function-call items.

### List conversations query parameters

| Name | Type | Description |
|---|---|---|
| `limit` | integer | 1 to 100. Default 20. |
| `order` | string | `asc` or `desc`. Default `desc`. |
| `after`, `before` | string | Paging cursors. |
| `agent_name` | string | Only conversations with this agent. |
| `agent_id` | string | Only conversations with this agent version, as `<agent-name>:<version>`. |

`GET /conversations/{conversation_id}/items` also accepts `limit` (1 to 100, default 20), `order` (`asc` or `desc`, default `desc`), `after`, and `before`.

### Delete a conversation

The service deletes the conversation and its items, and returns `{"id": "<conversation-id>", "object": "conversation.deleted", "deleted": true}`. Stored responses stay available, but they're no longer linked to the conversation.

## Related content

- [Agents REST API reference](agents-api-reference.md)
- [Knowledge Bases REST API reference](knowledge-bases-api-reference.md)
- [Run agents with the Responses API](../run-agents-responses-api.md)
