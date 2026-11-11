---
title: MCP Server Overview for Agents and Tools with Foundry Local
description: Learn how the built-in MCP server provides search tools, connects to clients, and supports external MCP server registration.
ms.reviewer: assafakiva
ms.date: 11/08/2026
ms.topic: concept-article
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Indexed Sources MCP in Agents and Tools with Foundry Local

The Agents and Tools with Foundry Local platform includes Indexed Sources MCP, a built-in retrieval service in Agentic Retrieval that exposes RAG retrieval capabilities through Model Context Protocol (MCP) tools. Any MCP-compatible AI agent or client can connect to search your ingested collections.

## What is the Model Context Protocol?

MCP is an open standard for connecting AI agents to external tools and data sources. It defines a JSON-RPC 2.0-based protocol where:

- A *server* exposes tools (functions that agents can call) and resources (data endpoints).
- A *client* discovers and invokes those tools.

This modular architecture lets agents connect to new data sources without code changes. Register a new MCP server.

## Indexed Sources MCP implementation

Indexed Sources MCP (`indexed-sources-mcp-server`) runs on port 8080 and exposes search tools that query your ingested collections. Agents and Tools with Foundry Local agents can also consume external MCP servers registered as knowledge sources.

### Protocol details

Indexed Sources MCP uses the following protocol specifications:

| Property | Value |
|---|---|
| **Transport** | MCP over streamable HTTP |
| **Endpoint** | `POST /mcp` |
| **Port** | 8080 |
| **Message format** | JSON-RPC 2.0 |

All tool calls go through a single endpoint. The `method` field in the JSON-RPC body determines the operation (`initialize`, `tools/list`, `tools/call`). Both the gateway and the app expose `/mcp`.

Send `Accept: application/json, text/event-stream`. Inspect the response's `Content-Type`: a JSON-RPC response can arrive as JSON or inside an SSE `data:` event. Parse the JSON-RPC envelope first, then parse the tool's JSON text content. HTTP `200` alone doesn't mean a tool succeeded; check the JSON-RPC `error` and the tool result's `isError` flag.

## Indexed Sources MCP tools

Indexed Sources MCP exposes six search tools that query your ingested collections:

| Search tool | Description | Best for |
|---|---|---|
| `search_hybrid` | Combined dense + sparse vector search with reranking | Default, general-purpose search (recommended) |
| `search_vector` | Pure semantic (dense) vector search | When query wording differs from document language |
| `search_text` | Sparse/keyword search using BGE-M3 learned representations | Exact terms, codes, product names |
| `search_image` | Image retrieval using vision embeddings | Finding images by text description |
| `search_multimodal` | Parallel hybrid + image search | Queries needing both text and image results |
| `get_available_collections` | Lists collections accessible to the authenticated user | Discovery, collection selection |

### Common parameters

The shared search schema for `search_hybrid`, `search_vector`, `search_text`, and `search_image` accepts the following parameters. For the separate text and image strictness fields in `search_multimodal`, see the [MCP API reference](reference/mcp-server-api-reference.md#search_multimodal).

Place these fields in `params.arguments.request`, not directly in `params.arguments`. Use `tools/list` to inspect each tool's input schema; collection discovery doesn't use this search-request wrapper.

| Parameter | Type | Default | Description |
|---|---|---|---|
| `query` | string | (required) | Search query text |
| `collection_names` | list[string] | `null` | Collections to search. Omit to use the collections in `X-Allowed-Collections`. |
| `top_n` | integer | `3` | Number of results (1–50) |
| `strictness` | integer | `1` | Relevance threshold (0–5). `0` returns all results; higher values filter more aggressively |
| `filters` | string | `null` | Reserved. The current search implementation doesn't apply this value. |

### Multi-collection search

You can query multiple collections in a single request by passing multiple names in `collection_names`. Results are returned keyed by collection name:

```json
{
  "my-docs": { "results": [], "metadata": {} },
  "other-docs": { "results": [], "metadata": {} }
}
```

## Connect to Indexed Sources MCP

You can connect to Indexed Sources MCP by using any MCP-compatible client. The connection process involves a session handshake where the client initializes a session, lists available tools, and then calls tools as needed.

### From an MCP-compatible client

Most MCP clients handle the session lifecycle automatically. Configure them by using `URL: https://<cluster-domain>/mcp`. For search calls, configure the `X-Allowed-Collections` header with the collection names that the client is allowed to request. A managed knowledge-base caller sets this scope from its indexed knowledge sources.

### Connect manually by using curl

MCP requires a session handshake with these three steps:

```bash
# 1. Initialize — capture the Mcp-Session-Id from response headers
curl -si -X POST https://<cluster-domain>/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'

# 2. List available tools
curl -s -X POST https://<cluster-domain>/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id-from-step-1>" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'

# 3. Call a tool
curl -s -X POST https://<cluster-domain>/mcp \
  -H "Content-Type: application/json" \
  -H "X-Allowed-Collections: <collection-name>" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: <session-id-from-step-1>" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{"jsonrpc":"2.0","id":3,"method":"tools/call","params":{"name":"search_hybrid","arguments":{"request":{"query":"How do I reset my device?","top_n":5}}}}'
```

### Port forwarding (development)

You can also connect to Indexed Sources MCP by port-forwarding to your local machine. Authentication for `tools/call` still applies when `IS_AUTH_ENABLED=true`. Discovery methods don't require a token.

```bash
kubectl port-forward --address 127.0.0.1 deployment/indexed-sources-mcp-server-deployment 8080:8080 -n arc-rag
# Then use http://127.0.0.1:8080/mcp with the required token and collection scope.
```

## Register external MCP servers

Agents can use an external MCP server directly as a tool, or a knowledge base can use the server as a `remote_mcp` knowledge source, which is a remote source in Agentic Retrieval.

- To add the server directly to an agent version, see [Add tools to an agent](add-agent-tools.md).
- To combine the server with other sources in a knowledge base, register it as a knowledge source. The caller needs **Cognitive Services Contributor** at the extension scope (**Contributor** in disconnected deployments) or the `EdgeRAGDeveloper` app role as a fallback. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

Use the following command to create a named knowledge source for the external MCP server:

```bash
curl -X POST https://<cluster-domain>/knowledgesources \
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

External destinations must use HTTPS and pass the API's address checks. With `unauthenticated`, no caller authentication header is forwarded. Use `microsoft_entra_id` only for a trusted destination approved by the deployment that validates the caller's Microsoft Entra token; forwarding to an untrusted external server is rejected.

Successful connection validation creates the source and returns `201 Created`. A validation failure returns `400 Bad Request` and creates nothing. [Check readiness](knowledge-sources-guide.md#check-readiness), then [create a named knowledge base](knowledge-bases-guide.md) that references `my-external-mcp`.

## Register an indexed collection

Register Indexed Sources MCP as an `indexed_sources_mcp` knowledge source to make an ingested collection available to a knowledge base. Use a token for a caller with **Cognitive Services Contributor** at the extension scope (**Contributor** in disconnected deployments), or the `EdgeRAGDeveloper` app role as a fallback.

The following example connects the `my-docs` collection. Replace it with a collection you prepared, and use the trusted internal Dapr invocation destination:

```bash
curl -X POST https://<cluster-domain>/knowledgesources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "my-docs-search",
    "kind": "indexed_sources_mcp",
    "auth_type": "microsoft_entra_id",
    "description": "Search the my-docs collection",
    "indexed_sources_parameters": {
      "server_url": "http://localhost:3500/v1.0/invoke/indexed-sources-mcp-server/method/mcp",
      "indexed_source_ref": "my-docs",
      "server_label": "indexed-sources-mcp-server"
    }
  }'
```

`indexed_source_ref` is the collection name. The HTTPS URL in the command is the client-facing gateway. The request body's `server_url` is the destination Agents and Tools with Foundry Local calls inside the cluster; `localhost` refers to its Dapr sidecar, not your computer.

This Indexed Sources MCP destination is explicitly trusted for credential forwarding. Don't substitute the client-facing `https://<cluster-domain>/mcp` URL: gateway access doesn't make that hostname a trusted server-side MCP destination, and validation can reject it with `400`. If your deployment uses a different internal endpoint, use the destination approved by its trust configuration. Don't disable Entra authentication to bypass validation.

Successful validation returns `201 Created`. [Check readiness](knowledge-sources-guide.md#check-readiness), then [associate the source with a knowledge base](knowledge-bases-guide.md).

## Authentication and security

When authentication is enabled, Indexed Sources MCP requires a bearer token for `tools/call`. This requirement also applies when you use port forwarding. Discovery methods such as `initialize` and `tools/list` don't require a token.

### Access control

| Context | Authentication |
|---|---|
| External access (ingress) | Bearer token required. Authentication enforced on `tools/call` only; `initialize` and `tools/list` pass through. |
| Port-forwarding | Uses the same `tools/call` authentication setting as ingress access. |
| Token forwarding | The bearer token is forwarded to downstream services for RBAC enforcement on collection access. |

### Supported authentication types for registered MCP servers

| Auth type | Description |
|---|---|
| `microsoft_entra_id` | Forwards the caller's token only to a trusted MCP destination approved by the deployment. |
| `unauthenticated` | Doesn't forward auth headers to the MCP server. |

> [!IMPORTANT]
> Agents and Tools with Foundry Local rejects credential forwarding to untrusted external MCP servers. Use `unauthenticated` for the external-server registration example.

You must specify `auth_type` when you create a registration through REST; the API doesn't provide a default. This setting controls downstream MCP authentication, separately from the bearer token used to authorize the registration request.

Treat external MCP tool descriptions and results as untrusted input. Register only servers that you operate or trust, don't put credentials in tool arguments, and don't rely on returned content to override your agent or security instructions.

## Related content

- [Agent Service overview](agentic-overview.md)
- [Collections overview](collections-overview.md)
- [Indexed Sources MCP API reference](reference/mcp-server-api-reference.md)
- [Configure a knowledge source](knowledge-sources-guide.md)
- [Knowledge sources REST API reference](reference/knowledge-sources-api-reference.md)