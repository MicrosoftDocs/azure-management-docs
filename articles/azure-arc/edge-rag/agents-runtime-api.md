---
title: Agents Runtime Service REST API - Azure Arc
description: Learn about the REST API for managing conversation threads in the Azure Arc Agents Runtime Service.
ms.topic: reference
ms.date: 05/01/2026
author: cwatson-cat
ms.author: cwatson
ms.service: azure-arc
ms.subservice: edge-rag
---

# Agents Runtime Service REST API

Use this REST API to manage conversation threads, messages, and runs with knowledge bases in the Azure Arc Agents Runtime Service.

## Authorization

All endpoints require a valid Bearer token issued by Microsoft Entra ID. The MISE sidecar running alongside the service validates the token.

### Obtain a token

Acquire an access token from Entra ID using the OAuth 2.0 client credentials flow or authorization code flow. The token must be issued for the Agentic RAG app registration audience.

```azurecli
az account get-access-token --resource api://<app-registration-client-id> --query accessToken -o tsv
```

### Pass the token

Include the token in the `Authorization` header of every request:

```http
Authorization: Bearer <access-token>
```

### Thread ownership

The `oid` (Object ID) claim from the token identifies the user. Thread ownership is enforced; users can only access threads they created. All CRUD operations on threads, messages, runs, and steps respect this ownership boundary. No specific role is required beyond being authenticated.

### Disable authentication (Development)

Set `IS_AUTH_ENABLED=false` in the environment to disable authentication. When disabled, all endpoints are accessible without a token and no ownership is enforced.

> [!NOTE]
> The Agents Runtime Service returns timestamps as Unix epoch integers (for example, "1699012345"). The Agent Management Service uses ISO 8601 datetime strings (for example, "2025-01-15T10:30:00Z").

## Create thread

Creates a new conversation thread.

### Request

```http
POST https://{runtime_host}/threads
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |

### POST example

```http
POST https://localhost:8001/threads
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "title": "Customer inquiry about billing",
    "metadata": {
        "user_id": "user123",
        "channel": "web"
    }
}
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 201 | Created. The request was fulfilled and a new thread was created. |
| 400 | Bad Request. Validation error. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 201 Created
```

#### Body

```json
{
    "id": "thread_abc123def456789012345678",
    "object": "thread",
    "created_at": 1699012345,
    "title": "Customer inquiry about billing",
    "owner_id": "00000000-0000-0000-0000-000000000201",
    "metadata": {
        "user_id": "user123",
        "channel": "web"
    }
}
```

## List threads

Lists all conversation threads with cursor-based pagination. When authentication is enabled, returns only threads owned by the authenticated user.

### Request

```http
GET https://{runtime_host}/threads?limit={limit}&order={order}&after={after}&before={before}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |

### Query parameters

| Name | Description | Data type |
| --- | --- | --- |
| limit | Optional. Maximum number of threads to return (1-100). Default is 20. | Integer |
| order | Optional. Sort order by created\_at. Values: `asc`, `desc`. Default is `desc`. | String |
| after | Optional. Cursor for pagination - return threads after this ID. | String |
| before | Optional. Cursor for pagination - return threads before this ID. | String |

### GET example

```http
GET https://localhost:8001/threads?limit=10&order=desc
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. A successful operation with the thread list. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "object": "list",
    "data": [
        {
            "id": "thread_abc123def456789012345678",
            "object": "thread",
            "created_at": 1699012345,
            "title": "Customer inquiry about billing",
            "owner_id": "00000000-0000-0000-0000-000000000201",
            "metadata": {
                "user_id": "user123"
            }
        }
    ],
    "first_id": "thread_abc123def456789012345678",
    "last_id": "thread_abc123def456789012345678",
    "has_more": false
}
```

## Get thread

Retrieves a specific thread by ID. Returns 404 if the thread doesn't exist or belongs to another user.

### Request

```http
GET https://{runtime_host}/threads/{thread_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested thread. |
| 404 | Not Found. The specified thread doesn't exist or belongs to another user. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "thread_abc123def456789012345678",
    "object": "thread",
    "created_at": 1699012345,
    "title": "Customer inquiry about billing",
    "owner_id": "00000000-0000-0000-0000-000000000201",
    "metadata": {
        "user_id": "user123"
    }
}
```

## Modify thread

Modifies an existing thread (title, metadata).

### Request

```http
POST https://{runtime_host}/threads/{thread_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### POST example

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "title": "Updated thread title",
    "metadata": {
        "user_id": "user123",
        "priority": "high"
    }
}
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The thread was modified successfully. |
| 404 | Not Found. The specified thread doesn't exist or belongs to another user. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "thread_abc123def456789012345678",
    "object": "thread",
    "created_at": 1699012345,
    "title": "Updated thread title",
    "owner_id": "00000000-0000-0000-0000-000000000201",
    "metadata": {
        "user_id": "user123",
        "priority": "high"
    }
}
```

## Delete thread

Deletes a thread by ID.

### Request

```http
DELETE https://{runtime_host}/threads/{thread_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### DELETE example

```http
DELETE https://localhost:8001/threads/thread_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The thread was deleted successfully. |
| 404 | Not Found. The specified thread doesn't exist or belongs to another user. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "thread_abc123def456789012345678",
    "object": "thread.deleted",
    "deleted": true
}
```

## Create message

Creates a new message in a thread.

### Request

```http
POST https://{runtime_host}/threads/{thread_id}/messages
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### POST example

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678/messages
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "role": "user",
    "content": "What are the pricing details for the enterprise plan?",
    "metadata": {
        "source": "web_chat"
    }
}
```

### Body fields

| Name | Required | Description | Data type |
| --- | --- | --- | --- |
| role | **Yes** | Role of the message author. Values: `user`, `assistant`, `system`. | String |
| content | **Yes** | Content of the message (string or content blocks). When provided as a string, the service normalizes it into an array of content blocks in the response. | String \| Array |
| metadata | No | Custom key-value metadata. | Object |
| attachments | No | File attachments. | Array |

### Example: Create message with content blocks

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678/messages
```

#### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

#### Body

```json
{
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": {
                "value": "What does this diagram show?",
                "annotations": []
            }
        },
        {
            "type": "image_url",
            "image_url": {
                "url": "https://example.com/diagram.png",
                "detail": "auto"
            }
        }
    ]
}
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 201 | Created. The request was fulfilled and a new message was created. |
| 400 | Bad Request. Validation error. |
| 404 | Not Found. The specified thread doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 201 Created
```

#### Body

```json
{
    "id": "msg_abc123def456789012345678",
    "object": "thread.message",
    "created_at": 1699012345,
    "thread_id": "thread_abc123def456789012345678",
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": {
                "value": "What are the pricing details for the enterprise plan?",
                "annotations": []
            }
        }
    ],
    "agent_id": null,
    "run_id": null,
    "attachments": null,
    "metadata": {
        "source": "web_chat"
    }
}
```

## List messages

Lists all messages in a thread with pagination.

### Request

```http
GET https://{runtime_host}/threads/{thread_id}/messages?limit={limit}&order={order}&after={after}&before={before}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### Query parameters

| Name | Description | Data type |
| --- | --- | --- |
| limit | Optional. Maximum number of messages to return (1-100). Default is 20. | Integer |
| order | Optional. Sort order by created\_at. Values: `asc`, `desc`. Default is `asc` (chronological). | String |
| after | Optional. Cursor for pagination - return messages after this ID. | String |
| before | Optional. Cursor for pagination - return messages before this ID. | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678/messages?limit=20&order=asc
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. A successful operation with the message list. |
| 404 | Not Found. The specified thread doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "object": "list",
    "data": [
        {
            "id": "msg_abc123def456789012345678",
            "object": "thread.message",
            "created_at": 1699012345,
            "thread_id": "thread_abc123def456789012345678",
            "role": "user",
            "content": [
                {
                    "type": "text",
                    "text": {
                        "value": "What are the pricing details for the enterprise plan?",
                        "annotations": []
                    }
                }
            ],
            "agent_id": null,
            "run_id": null,
            "attachments": null,
            "metadata": null
        },
        {
            "id": "msg_def456ghi789012345678901",
            "object": "thread.message",
            "created_at": 1699012350,
            "thread_id": "thread_abc123def456789012345678",
            "role": "assistant",
            "content": [
                {
                    "type": "text",
                    "text": {
                        "value": "The enterprise plan starts at $99/month and includes...",
                        "annotations": []
                    }
                }
            ],
            "agent_id": "asst_abc123def456789012345678",
            "run_id": "run_abc123def456789012345678",
            "attachments": null,
            "metadata": null
        }
    ],
    "first_id": "msg_abc123def456789012345678",
    "last_id": "msg_def456ghi789012345678901",
    "has_more": false
}
```

## Get message

Retrieves a specific message by ID.

### Request

```http
GET https://{runtime_host}/threads/{thread_id}/messages/{message_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| message\_id | Unique message identifier (format: `msg_{uuid}`) | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678/messages/msg_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested message. |
| 404 | Not Found. The specified thread or message doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "msg_abc123def456789012345678",
    "object": "thread.message",
    "created_at": 1699012345,
    "thread_id": "thread_abc123def456789012345678",
    "role": "user",
    "content": [
        {
            "type": "text",
            "text": {
                "value": "What are the pricing details for the enterprise plan?",
                "annotations": []
            }
        }
    ],
    "agent_id": null,
    "run_id": null,
    "attachments": null,
    "metadata": null
}
```

## Delete message

Deletes a message by ID.

### Request

```http
DELETE https://{runtime_host}/threads/{thread_id}/messages/{message_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| message\_id | Unique message identifier (format: `msg_{uuid}`) | String |

### DELETE example

```http
DELETE https://localhost:8001/threads/thread_abc123def456789012345678/messages/msg_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The message was deleted successfully. |
| 404 | Not Found. The specified thread or message doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "msg_abc123def456789012345678",
    "object": "thread.message.deleted",
    "deleted": true
}
```

## Create run

Creates and executes a run against a thread using a knowledge base. Supports both synchronous (background) and streaming (SSE) execution modes.

### Request

```http
POST https://{runtime_host}/threads/{thread_id}/runs?stream={stream}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |

### Query parameters

| Name | Description | Data type |
| --- | --- | --- |
| stream | Optional. When `true`, returns a StreamingResponse with SSE events. Default is `false`. | Boolean |

### POST example

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678/runs
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "agent_id": "asst_abc123def456789012345678",
    "instructions": "Focus on billing-related questions only.",
    "additional_messages": [
        {
            "role": "user",
            "content": "What is the refund policy?"
        }
    ],
    "metadata": {
        "request_source": "api"
    }
}
```

### Body fields

| Name | Required | Description | Data type |
| --- | --- | --- | --- |
| agent\_id | **Yes** | ID of the knowledge base to run against (format: `asst_{uuid}`). | String |
| instructions | No | Override the knowledge base's default instructions for this run. | String |
| additional\_messages | No | Additional messages to append to the thread before execution. | Array |
| metadata | No | Custom key-value metadata for the run. | Object |

### Example: Create run with streaming

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678/runs?stream=true
```

#### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

#### Body

```json
{
    "agent_id": "asst_abc123def456789012345678"
}
```

#### Streaming response

When you set `stream=true`, the response is returned as server-sent events (SSE) with `Content-Type: text/event-stream`.

```output
event: thread.run.created
data: {"id": "run_abc123", "object": "thread.run", "status": "queued", "thread_id": "thread_abc123def456789012345678", "agent_id": "asst_abc123def456789012345678"}

event: thread.run.in_progress
data: {"id": "run_abc123", "object": "thread.run", "status": "in_progress", ...}

event: thread.message.delta
data: {"id": "msg_xyz789", "delta": {"content": [{"type": "text", "text": {"value": "The refund "}}]}}

event: thread.message.delta
data: {"id": "msg_xyz789", "delta": {"content": [{"type": "text", "text": {"value": "policy allows..."}}]}}

event: thread.run.completed
data: {"id": "run_abc123", "object": "thread.run", "status": "completed", ...}

event: done
data: [DONE]
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 201 | Created. The run was created and execution has started (background mode). |
| 400 | Bad Request. Validation error. |
| 404 | Not Found. The specified thread doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json` (non-streaming) or `text/event-stream` (streaming)

#### Header

```http
HTTP/1.1 201 Created
```

#### Body (non-streaming)

```json
{
    "id": "run_abc123def456789012345678",
    "object": "thread.run",
    "created_at": 1699012345,
    "thread_id": "thread_abc123def456789012345678",
    "agent_id": "asst_abc123def456789012345678",
    "status": "queued",
    "instructions": "Focus on billing-related questions only.",
    "last_error": null,
    "usage": null,
    "metadata": {
        "request_source": "api"
    },
    "started_at": null,
    "completed_at": null,
    "failed_at": null,
    "cancelled_at": null
}
```

## Get run

Retrieves a specific run by ID.

### Request

```http
GET https://{runtime_host}/threads/{thread_id}/runs/{run_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| run\_id | Unique run identifier (format: `run_{uuid}`) | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678/runs/run_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested run. |
| 404 | Not Found. The specified thread or run doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "run_abc123def456789012345678",
    "object": "thread.run",
    "created_at": 1699012345,
    "thread_id": "thread_abc123def456789012345678",
    "agent_id": "asst_abc123def456789012345678",
    "status": "completed",
    "instructions": null,
    "last_error": null,
    "usage": {
        "prompt_tokens": 100,
        "completion_tokens": 50,
        "total_tokens": 150
    },
    "metadata": null,
    "started_at": 1699012346,
    "completed_at": 1699012350,
    "failed_at": null,
    "cancelled_at": null
}
```

## Cancel run

Cancels a run that's in progress or queued.

### Request

```http
POST https://{runtime_host}/threads/{thread_id}/runs/{run_id}/cancel
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| run\_id | Unique run identifier (format: `run_{uuid}`) | String |

### POST example

```http
POST https://localhost:8001/threads/thread_abc123def456789012345678/runs/run_abc123def456789012345678/cancel
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The run was cancelled successfully. |
| 400 | Bad Request. The run isn't in a cancellable state. It must be `queued` or `in_progress`. |
| 404 | Not Found. The specified thread or run doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "run_abc123def456789012345678",
    "object": "thread.run",
    "created_at": 1699012345,
    "thread_id": "thread_abc123def456789012345678",
    "agent_id": "asst_abc123def456789012345678",
    "status": "cancelled",
    "instructions": null,
    "last_error": null,
    "usage": null,
    "metadata": null,
    "started_at": 1699012346,
    "completed_at": null,
    "failed_at": null,
    "cancelled_at": 1699012348
}
```

## List run steps

Lists steps belonging to a run. Each step has a `type` field that indicates the kind of action:

| Step type | Description |
| --- | --- |
| `message_creation` | The agent generated a response message. |
| `tool_calls` | The agent invoked a built-in function tool. |
| `mcp_call` | The agent invoked an MCP (Model Context Protocol) server tool. |

### Request

```http
GET https://{runtime_host}/threads/{thread_id}/runs/{run_id}/steps?limit={limit}&order={order}&after={after}&before={before}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| run\_id | Unique run identifier (format: `run_{uuid}`) | String |

### Query parameters

| Name | Description | Data type |
| --- | --- | --- |
| limit | Optional. Maximum number of steps to return (1-100). Default is 20. | Integer |
| order | Optional. Sort order by created\_at. Values: `asc`, `desc`. Default is `desc`. | String |
| after | Optional. Cursor for pagination - return steps after this ID. | String |
| before | Optional. Cursor for pagination - return steps before this ID. | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678/runs/run_abc123def456789012345678/steps?limit=10
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. A successful operation with the step list. |
| 404 | Not Found. The specified thread doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "object": "list",
    "data": [
        {
            "id": "step_abc123def456789012345678",
            "object": "thread.run.step",
            "created_at": 1699012346,
            "run_id": "run_abc123def456789012345678",
            "thread_id": "thread_abc123def456789012345678",
            "type": "tool_calls",
            "step_details": {
                "tool_calls": [
                    {
                        "name": "search_knowledge_base",
                        "arguments": "{\"query\": \"enterprise pricing\"}",
                        "result": "{\"results\": [...]}"
                    }
                ]
            },
            "completed_at": 1699012348
        },
        {
            "id": "step_def456ghi789012345678901",
            "object": "thread.run.step",
            "created_at": 1699012349,
            "run_id": "run_abc123def456789012345678",
            "thread_id": "thread_abc123def456789012345678",
            "type": "message_creation",
            "step_details": {
                "message_creation": {
                    "message_id": "msg_def456ghi789012345678901"
                }
            },
            "completed_at": 1699012350
        },
        {
            "id": "step_ghi789jkl012345678901234",
            "object": "thread.run.step",
            "created_at": 1699012351,
            "run_id": "run_abc123def456789012345678",
            "thread_id": "thread_abc123def456789012345678",
            "type": "mcp_call",
            "step_details": {
                "mcp_call": {
                    "mcp_server": "my-mcp-server",
                    "request_payload": "{\"tool\": \"lookup\", \"args\": {\"id\": \"123\"}}",
                    "response_payload": "{\"result\": \"found\"}"
                }
            },
            "completed_at": 1699012352
        }
    ],
    "first_id": "step_abc123def456789012345678",
    "last_id": "step_ghi789jkl012345678901234",
    "has_more": false
}
```

## Get run step

Retrieves a specific run step by ID.

### Request

```http
GET https://{runtime_host}/threads/{thread_id}/runs/{run_id}/steps/{step_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| runtime\_host | Hostname of the Agents Runtime Service | String |
| thread\_id | Unique thread identifier (format: `thread_{uuid}`) | String |
| run\_id | Unique run identifier (format: `run_{uuid}`) | String |
| step\_id | Unique step identifier (format: `step_{uuid}`) | String |

### GET example

```http
GET https://localhost:8001/threads/thread_abc123def456789012345678/runs/run_abc123def456789012345678/steps/step_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested run step. |
| 404 | Not Found. The specified thread, run, or step doesn't exist. |
| 500 | Internal Server Error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 200 OK
```

#### Body

```json
{
    "id": "step_abc123def456789012345678",
    "object": "thread.run.step",
    "created_at": 1699012346,
    "run_id": "run_abc123def456789012345678",
    "thread_id": "thread_abc123def456789012345678",
    "type": "tool_calls",
    "step_details": {
        "tool_calls": [
            {
                "name": "search_knowledge_base",
                "arguments": "{\"query\": \"enterprise pricing\"}",
                "result": "{\"results\": [...]}"
            }
        ]
    },
    "completed_at": 1699012348
}
```
