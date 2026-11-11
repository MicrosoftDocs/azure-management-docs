---
title: Agent Service Limits - Agents and Tools with Foundry Local
description: Reference for the limits of agents, connections, knowledge bases, knowledge sources, conversations, and responses in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
---

# Agent service limits in Agents and Tools with Foundry Local

This article lists the limits of Agent Service in Agents and Tools with Foundry Local. When a request goes over a limit, the service returns `400 Bad Request`, `409 Conflict` for full conversations, or `429 Too Many Requests` when SQL knowledge sources exceed capacity or concurrency limits.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Names

| Resource | Rule |
|---|---|
| Agent | 1 to 63 characters. Letters, digits, and hyphens. Starts and ends with a letter or digit. Not case-sensitive; stored in lowercase. |
| Connection | 1 to 63 characters. Lowercase letters, digits, and hyphens. Starts and ends with a letter or digit. |
| Knowledge base | 1 to 40 characters. Lowercase letters, digits, and hyphens. Starts and ends with a letter or digit. |
| Knowledge source | 1 to 40 characters. Lowercase letters, digits, and hyphens. Starts and ends with a letter or digit. Can't look like a GUID. |

## Agents and agent versions

| Limit | Value |
|---|---|
| `definition.instructions` | 87,381 characters |
| Tools per agent version | 32 |
| Tool names in `allowed_tools` | 128, each up to 128 characters |
| `server_label` | 64 characters |
| `server_description` | 512 characters |
| Agent version `description` | 512 characters |
| Metadata | 16 key-value pairs. Keys up to 64 characters, values up to 512 characters. |
| Total size of one agent version | 1 MiB |
| Request body size | 2 MiB |
| `temperature` | 0 to 2 |
| `top_p` | 0 to 1 |
| Versions selected at a time | 1 (100% of traffic) |
| Agent and version list page size | 1 to 100, default 20 |

## Connections

| Limit | Value |
|---|---|
| `target` address | 2,048 characters, HTTPS |
| `tokenScope` | 2,048 characters, ending with `/.default` |
| API key | 16,384 characters |
| API key header name | 128 characters |
| Model connection `maxContextTokens` | 4,096 to 10,000,000 |
| Model name | 253 characters |
| Connection list page size | 1 to 100, default 50 |

## Responses

| Limit | Value |
|---|---|
| Text input | 128,000 characters in total |
| Input message items | 1 to 200 |
| Content parts per message | 20 |
| `instructions` in a request | 4,096 characters |
| Conversation history that the agent reads | The most recent 100 items |
| Responses in a `previous_response_id` chain | 100 |
| Wait time for a cancel request | 10 seconds |
| Response list page size | 1 to 100, default 20 |

## Conversations

| Limit | Value |
|---|---|
| Items per conversation | 2,000 |
| Items added in one request | 20 |
| Metadata | 16 key-value pairs. Keys up to 64 characters, values up to 512 characters. |
| Conversation list page size | 1 to 100, default 20 |

## Knowledge bases

| Limit | Value |
|---|---|
| Knowledge sources per knowledge base | 1 to 100 |
| Models per knowledge base | 1 |
| `description` | 512 characters, required |
| `retrievalInstructions` | 8,192 characters |
| `answerInstructions` | 8,192 characters |

## Knowledge sources

| Limit | Value |
|---|---|
| `description` | 500 characters |
| `server_url` | 2,048 characters |
| `server_label` | 64 characters |
| `indexed_source_ref` | 256 characters |
| SQL knowledge sources per cluster | 25 |
| Concurrent SQL database reads | 4 |
| SQL entities per source | 200 |
| SQL fields per entity | 500 |
| SQL `database_label` | 64 characters |
| SQL `include_schemas` in schema preview | 20 |
| List page size | 1 to 100, default 10 |
| Names in one batch delete | 100 |

## Related content

- [Agents REST API reference](agents-api-reference.md)
- [Responses and Conversations REST API reference](responses-api-reference.md)
- [Knowledge Bases REST API reference](knowledge-bases-api-reference.md)
