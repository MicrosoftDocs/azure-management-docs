---
title: Agents and Tools with Foundry Local MCP Server Overview
description: Learn about the Model Context Protocol (MCP) server in Agents and Tools with Foundry Local, including built-in search tools, connection methods, and how to register external MCP servers.
author: cwatson-cat
ms.author: cwatson
ms.date: 04/30/2026
ms.topic: concept-article
ms.subservice: edge-rag
ai-usage: ai-generated
---

# MCP server in Agents and Tools with Foundry Local

The Agents and Tools with Foundry Local platform includes a built-in Model Context Protocol (MCP) server that exposes RAG retrieval capabilities as tools. Any MCP-compatible AI agent or client can connect to search your ingested collections.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## What is the Model Context Protocol?

MCP is an open standard for connecting AI agents to external tools and data sources. It defines a JSON-RPC 2.0-based protocol where:

- A *server* exposes tools (functions that agents can call) and resources (data endpoints).
- A *client* discovers and invokes those tools.

This plug-and-play architecture allows agents to connect to new data sources without code modifications. You simply register a new MCP server.

## Agents and Tools with Foundry Local's MCP server implementation

The built-in MCP server `indexed-sources-mcp-server` runs on port 8080 and exposes search tools that query your ingested collections. It also supports consuming external MCP servers by registering them as knowledge sources, allowing agents to access tools and data from any MCP-compatible service.

### Protocol details

The MCP server uses the following protocol specifications:

| Property | Value |
|---|---|
| **Transport** | MCP over streamable HTTP |
| **Endpoint** | `POST /edgeai/mcp` |
| **Port** | 8080 |
| **Message format** | JSON-RPC 2.0 |

All tool calls go through a single endpoint. The `method` field in the JSON-RPC body determines the operation (`initialize`, `tools/list`, `tools/call`).

## Built-in search tools

The built-in MCP server exposes six search tools that query your ingested collections:

| Search tool | Description | Best for |
|---|---|---|
| `search_hybrid` | Combined dense + sparse vector search with reranking | Default, general-purpose search (recommended) |
| `search_vector` | Pure semantic (dense) vector search | When query wording differs from document language |
| `search_text` | Sparse/keyword search using BGE-M3 learned representations | Exact terms, codes, product names |
| `search_image` | Image retrieval using vision embeddings | Finding images by text description |
| `search_multimodal` | Parallel hybrid + image search | Queries needing both text and image results |
| `get_available_collections` | Lists collections accessible to the authenticated user | Discovery, collection selection |

### Common parameters

All search tools (except `get_available_collections`) accept:

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | (required) | Search query text |
| `collection_names` | list[string] | `["edgeragapp"]` | Collections to search |
| `top_n` | integer | `5` | Number of results (1–50) |
| `strictness` | integer | `1` | Relevance threshold (0–5). `0` returns all results; higher values filter more aggressively |
| `filters` | string | `null` | Milvus boolean filter expression (for example, `file_path like "%manual%"`) |

### Multi-collection search

You can query multiple collections in a single request by passing multiple names in `collection_names`. Results are returned keyed by collection name:

```json
{
  "my-docs": { "results": [...], "metadata": {...} },
  "other-docs": { "results": [...], "metadata": {...} }
}
```

## Connect to the MCP server

You can connect to the MCP server using any MCP-compatible client. The connection process involves a session handshake where the client initializes a session, lists available tools, and then calls tools as needed.

### From an MCP-compatible client

Most MCP clients (for example, Claude Desktop, custom agents) handle the session lifecycle automatically. Configure them by using `URL: https://<cluster-domain>/edgeai/mcp`.

### Manual connection by using curl

MCP requires a session handshake with these three steps:

```bash
# 1. Initialize — capture the Mcp-Session-Id from response headers
curl -si -X POST https://<cluster-domain>/edgeai/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'

# 2. List available tools
curl -s -X POST https://<cluster-domain>/edgeai/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id-from-step-1>" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'

# 3. Call a tool
curl -s -X POST https://<cluster-domain>/edgeai/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id-from-step-1>" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"search_hybrid","arguments":{"query":"How do I reset my device?","top_n":5}}}'
```

### Port-forwarding (development)

You can also connect to the MCP server by port-forwarding to your local machine. This method is useful for development and testing with tools like Postman or curl without needing to manage authentication tokens.

```bash
kubectl port-forward deployment/indexed-sources-mcp-server-deployment 8080:8080 -n arc-rag
# Then use http://localhost:8080/edgeai/mcp (no Authorization header needed)
```

## Register external MCP servers

Agents and Tools with Foundry Local agents can connect to *external* MCP servers by registering them as knowledge sources with `kind: remote_mcp`. This registration allows agents to access tools and data from any MCP-compatible service.

Use the following command to create a knowledge source for the external MCP server:

```bash
curl -X POST https://<cluster-domain>/edgeai/knowledgesources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "my-external-mcp",
    "kind": "remote_mcp",
    "auth_type": "unauthenticated",
    "description": "External MCP server for custom tools",
    "remote_mcp_parameters": {
      "server_url": "https://external-mcp-server.example.com/mcp",
      "server_label": "my-external-server"
    }
  }'
```

<!-- For the full flow, see [Knowledge Bases Guide](knowledge-bases-guide.md). -->

## Authentication and security

You can connect to the MCP server using bearer token authentication or without authentication (for port-forwarding).

### Access control

| Context | Authentication |
|---|---|
| External access (ingress) | Bearer token required. Authentication enforced on `tools/call` only; `initialize` and `tools/list` pass through. |
| Port-forwarding | No authentication required. |
| Token forwarding | The bearer token is forwarded to downstream services for RBAC enforcement on collection access. |

### Supported authentication types for external MCP servers

| Auth type | Description |
|---|---|
| `microsoft_entra_id` | Forwards the caller's JWT token to the MCP server. |
| `unauthenticated` | No authentication required by the MCP server. |

> [!WARNING]
> **Security consideration (GAP-02):** When agents connect to external MCP servers, the caller's bearer token might be forwarded to the external server. Ensure you trust all registered external MCP servers. This behavior is under review for hardening.

## Related content

- [Agentic layer overview](agentic-overview.md)
<!-- - [Collections overview](collections-overview.md)
- [MCP server API reference](APIs/MCP-SERVER-API-REFERENCE.md)
- [Knowledge sources API reference](APIs/KNOWLEDGE-SOURCES-API-REFERENCE.md)
-->