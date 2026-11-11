---
title: Run Agents with the Responses API - Agents and Tools with Foundry Local
description: Learn how to run agents in Agents and Tools with Foundry Local by using the Responses API, stream answers, keep conversations, and read citations.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
# Customer intent: As an application developer, I want to call my agents from my own application so that my users can ask questions and get grounded, cited answers.
---

# Run agents with the Responses API in Agents and Tools with Foundry Local

Your applications talk to agents through the *Responses API*. It follows the shape of the OpenAI Responses and Conversations APIs, so you can use the OpenAI Python library and familiar request formats. The Local chat UI uses the same API.

This article shows you how to:

- Send a question to an agent and read the answer.
- Stream the answer as the agent writes it.
- Continue a conversation across turns.
- Read citations and tool calls.
- List, get, and cancel responses.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

- An enabled agent. See [Create and manage agents](create-manage-agents.md).
- A Microsoft Entra ID caller with **Cognitive Services OpenAI User** at the extension resource scope. Also assign **Reader** if the caller lists agents. In disconnected deployments, use **Contributor**. Responses and Conversations have no app-role fallback; `EdgeRAGEndUser` and `EdgeRAGDeveloper` alone don't allow these calls. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- **Reader** assigned to the extension's managed identity at the extension scope, so it can check caller roles. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles).

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

The API base address is `https://<cluster-domain>/openai/v1`.

## Send a question to an agent

Send a `POST` request to `/openai/v1/responses`. Use `agent_reference` to name the agent, and `input` for the user's question.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/openai/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "agent_reference": { "type": "agent_reference", "name": "hr-helper" },
    "input": "How many vacation days do new employees get?"
  }'
```

# [Python](#tab/python)

The OpenAI library doesn't have an `agent_reference` parameter, so pass it in `extra_body`:

```python
response = client.responses.create(
    input="How many vacation days do new employees get?",
    extra_body={"agent_reference": {"type": "agent_reference", "name": "hr-helper"}},
)
print(response.status)
print(response.output_text)
```

---

If you leave out `agent_reference.version`, the agent's selected version answers. This version is the latest unless a developer pins another one. To test a specific version, add `"version": "2"`.

The response looks like this (shortened):

```json
{
  "id": "resp_6f1c...",
  "object": "response",
  "created_at": 1792575700,
  "status": "completed",
  "model": "local-model/gpt-oss-20b",
  "output": [
    {
      "type": "mcp_call",
      "server_label": "benefits",
      "name": "knowledge_base_retrieve",
      "arguments": "{\"query\":\"vacation days new employees\"}",
      "status": "completed",
      "output": "..."
    },
    {
      "type": "message",
      "id": "msg_9a2e...",
      "role": "assistant",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "[cite:0] New employees get 20 vacation days per year.",
          "annotations": [
            {
              "type": "file_citation",
              "text": "[cite:0]",
              "file_citation": { "file_id": "policies/leave-policy.pdf", "quote": "..." },
              "file_path": "policies/leave-policy.pdf",
              "title": "leave-policy",
              "page_numbers": [3],
              "collection": "knowledge_base"
            }
          ]
        }
      ]
    }
  ],
  "usage": { "input_tokens": 812, "output_tokens": 46, "total_tokens": 858 },
  "error": null
}
```

To get the answer text, read the `text` of the `output_text` parts inside the `message` items of `output`. The OpenAI Python library joins them for you in `response.output_text`.

### Response status

The `status` field tells you how the response ended. Always check it: a request can return `200 OK` with `status` set to `failed`. Also check [tool errors](#tool-errors) and warnings. A completed response doesn't mean every tool call succeeded.

| Status | Meaning |
|---|---|
| `completed` | The agent finished. The answer is in `output`. |
| `incomplete` | The agent stopped early, for example because it reached the token limit. See `incomplete_details.reason`. |
| `failed` | Something went wrong. See `error.code` and `error.message`. |
| `cancelled` | Someone cancelled the response. |
| `in_progress` | The response is still running. You see this status when you get a running response. |

### Input rules

- `input` can be a string of up to 128,000 characters, or a list of 1 to 200 message items.
- Message items use the role `user` and text content. For example: `{"role": "user", "content": [{"type": "input_text", "text": "Hello"}]}`.
- Text is the only supported input. The service doesn't support images, files, or client-side function calls.
- The service doesn't support the `tools`, `text`, `metadata`, and `background` request fields. Set tools and output format in the agent version instead.

## Stream the answer

To show the answer while the agent writes it, set `stream` to `true`. The service sends Server-Sent Events (SSE).

# [curl](#tab/curl)

```bash
curl -N -X POST "$BASE/openai/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "agent_reference": { "type": "agent_reference", "name": "hr-helper" },
    "input": "Summarize the parental leave policy.",
    "stream": true
  }'
```

# [Python](#tab/python)

```python
stream = client.responses.create(
    input="Summarize the parental leave policy.",
    stream=True,
    extra_body={"agent_reference": {"type": "agent_reference", "name": "hr-helper"}},
)
for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
    elif event.type == "response.completed":
        print("\nDone:", event.response.id)
```

---

A simple answer produces events in this order:

| Event | Meaning |
|---|---|
| `response.created` | The service created the response. |
| `response.in_progress` | The agent started working. |
| `response.output_item.added` | A new output item starts, such as a message. |
| `response.content_part.added` | A new content part starts. |
| `response.output_text.delta` | A piece of answer text. Add the `delta` values together to build the answer. |
| `response.output_text.done` | The full text of the content part. |
| `response.content_part.done` | The content part is complete. |
| `response.output_item.done` | The output item is complete. |
| `response.completed` | The response finished. The event contains the full response. |

When the agent calls tools, you also see `response.mcp_list_tools.*`, `response.mcp_call.in_progress`, `response.mcp_call_arguments.delta`, `response.mcp_call_arguments.done`, and `response.mcp_call.completed` or `response.mcp_call.failed` events. The last event is `response.completed`, `response.incomplete`, or `response.failed`. If someone cancels the response, the stream ends without a final event. Get the response to read its `cancelled` status.

## Continue a conversation across turns

An agent only remembers earlier turns if you connect them. You can connect them in two ways. You can't use both methods in the same request.

### Option 1: Use a conversation

A *conversation* stores the turns on the server, up to 2,000 items. This option is the best choice for chat applications.

# [curl](#tab/curl)

1. Create a conversation:

    ```bash
    curl -X POST "$BASE/openai/v1/conversations" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d '{ "metadata": { "topic": "benefits" } }'
    ```

    The response includes the conversation ID, such as `conv_12ab...`.

1. Send each question with the conversation ID:

    ```bash
    curl -X POST "$BASE/openai/v1/responses" \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d '{
        "agent_reference": { "type": "agent_reference", "name": "hr-helper" },
        "conversation": "conv_12ab...",
        "input": "And what about part-time employees?"
      }'
    ```

# [Python](#tab/python)

```python
agent = {"agent_reference": {"type": "agent_reference", "name": "hr-helper"}}

conversation = client.conversations.create(metadata={"topic": "benefits"})

first = client.responses.create(
    input="How many vacation days do new employees get?",
    conversation=conversation.id,
    extra_body=agent,
)
second = client.responses.create(
    input="And what about part-time employees?",
    conversation=conversation.id,
    extra_body=agent,
)
print(second.output_text)
```

---

The service adds the question and the answer to the conversation. Before it answers, the agent reads the most recent 100 items of the conversation.

### Option 2: Chain responses

Pass the ID of the previous response in `previous_response_id`. A chain can be up to 100 responses long. You can chain only responses that aren't part of a conversation. If the previous response belongs to a conversation, the request fails with `400 Bad Request`.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/openai/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "agent_reference": { "type": "agent_reference", "name": "hr-helper" },
    "previous_response_id": "resp_6f1c...",
    "input": "And what about part-time employees?"
  }'
```

# [Python](#tab/python)

```python
follow_up = client.responses.create(
    input="And what about part-time employees?",
    previous_response_id=response.id,
    extra_body={"agent_reference": {"type": "agent_reference", "name": "hr-helper"}},
)
```

---

## Manage conversations

Conversations belong to the user who creates them. Other users can't see them.

| Task | Request | Python |
|---|---|---|
| List your conversations | `GET /openai/v1/conversations?limit=20&order=desc`. Filter by agent with `agent_name=<name>`. | `api.get(f"{BASE}/openai/v1/conversations")` |
| Get a conversation | `GET /openai/v1/conversations/<conversation-id>` | `client.conversations.retrieve(id)` |
| Replace the metadata | `POST /openai/v1/conversations/<conversation-id>` with `{"metadata": {...}}`. The new metadata replaces the old metadata. | `client.conversations.update(id, metadata={...})` |
| List the turns | `GET /openai/v1/conversations/<conversation-id>/items` | `client.conversations.items.list(id)` |
| Add turns without running the agent | `POST /openai/v1/conversations/<conversation-id>/items` with `{"items": [...]}`, 1 to 20 items. | `client.conversations.items.create(id, items=[...])` |
| Delete a turn | `DELETE /openai/v1/conversations/<conversation-id>/items/<item-id>` | `client.conversations.items.delete(item_id, conversation_id=id)` |
| Delete a conversation | `DELETE /openai/v1/conversations/<conversation-id>` | `client.conversations.delete(id)` |

Metadata can have up to 16 key-value pairs. Keys can be up to 64 characters, and values up to 512 characters. A conversation can hold up to 2,000 items.

## Read citations

When an agent uses a knowledge base, the answer text includes markers such as `[cite:0]`. Each marker has a matching item in the `annotations` list of the `output_text` part. The following table describes the main citation fields.

| Field | Description |
|---|---|
| `text` | The marker in the answer, such as `[cite:0]`. |
| `file_citation.file_id` | The source document. |
| `file_citation.quote` | A short quote from the source. |
| `title` | The display title of the document. |
| `page_numbers` | The pages where the content appears. |
| `web_url` | A link to the source, when one is available. |

Use the following Python code to print each citation:

```python
for item in response.output:
    if item.type == "message":
        for part in item.content:
            for note in part.annotations or []:
                note = note if isinstance(note, dict) else note.model_dump()
                print(note["text"], note.get("title"), note.get("page_numbers"))
```

Show these citations to users so they can check where the answer came from.

## List, get, and cancel responses

Responses also belong to the user who created them.

- **List your responses:** `GET /openai/v1/responses?limit=20&order=desc`. Filter with `agent_name`, `conversation_id`, or `$filter`, for example `$filter=status eq 'failed'`. For the supported filter expressions, see the [REST API reference](reference/responses-api-reference.md#list-responses-query-parameters).
- **Get a response:** `GET /openai/v1/responses/<response-id>`, or `client.responses.retrieve(id)` in Python.
- **Get the input of a response:** `GET /openai/v1/responses/<response-id>/input_items`.
- **Cancel a running response:** `POST /openai/v1/responses/<response-id>/cancel`, or `client.responses.cancel(id)` in Python. The service returns the response with any output that the agent produced so far. If the response already finished, the request fails with `400 Bad Request`.

## Errors

Errors use this format:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Input must not be empty",
    "param": "input"
  }
}
```

| Status code | Common cause |
|---|---|
| `400` | The request isn't valid. |
| `401` | The access token is missing or not valid. |
| `403` | The user doesn't have permission, or the agent is disabled. |
| `404` | The agent, version, conversation, or response doesn't exist, or belongs to another user. |
| `409` | The request conflicts with the current state, for example a conversation that already has the maximum of 2,000 items. |
| `502`, `503`, `504` | A platform service isn't available or didn't answer in time. |

If the response fails while the agent runs, the service can still return `200`. The response has `status` set to `failed`, and `error.code` and `error.message` describe the response failure. For more help, see [Troubleshoot Agent Service](troubleshoot-agent-service.md).

### Tool errors

Tool failures and response failures are different. The agent can produce an answer even when an `mcp_call` item has `status` set to `failed` and an `error`, or the response's `warnings` list reports a tool problem. Check these fields along with the response status before using an answer that depends on a tool.

For knowledge base calls, the agent's answer or `mcp_call.error` can describe a generic knowledge base tool failure rather than the underlying source's exact error. For example, a SQL aggregation helper can fail because it couldn't complete its scan within the page budget, without that helper message appearing verbatim in the outer response.

Don't interpret a failed or incomplete SQL aggregation scan as a complete total. Narrow the supported filters to reduce the scan, and retry. See [SQL aggregation scan budgets](connect-sql-database.md#aggregation-for-postgresql-and-mysql).

## Call a model without an agent

For simple tests, you can call a model connection directly. Replace `agent_reference` with `model`, using the format `<model-connection-name>/<model-name>`:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/openai/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{ "model": "local-model/gpt-oss-20b", "input": "Say hello." }'
```

# [Python](#tab/python)

```python
response = client.responses.create(
    model="local-model/gpt-oss-20b",
    input="Say hello.",
)
print(response.output_text)
```

---

The model answers without instructions or tools.

## Related content

- [Quickstart: Create an agent and chat with it](quickstart-create-agent.md)
- [Create and manage agents](create-manage-agents.md)
- [Responses and Conversations REST API reference](reference/responses-api-reference.md)
- [How the Local chat UI works](chat-experience.md)
