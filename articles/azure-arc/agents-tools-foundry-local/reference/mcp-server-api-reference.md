---
title: MCP Server API reference - Agents and Tools with Foundry Local
description: API reference for the MCP server that exposes RAG retrieval capabilities as tools over Model Context Protocol.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
---

# Indexed Sources MCP API reference

Indexed Sources MCP exposes RAG retrieval capabilities as tools over Model Context Protocol (MCP) for any MCP-compatible AI agent or client.

## API information

| Property | Value |
|---|---|
| **Protocol** | MCP over Streamable HTTP |
| **Service** | `indexed-sources-mcp-server` |
| **Port** | 8080 |
| **MCP endpoint** | `/mcp` |
| **Health endpoint** | `GET /mcp/health` |

Unlike the other extension APIs, Indexed Sources MCP uses a single endpoint for all operations. Send an HTTP `POST` to `/mcp` with a JSON-RPC 2.0 body specifying the method (`initialize`, `tools/list`, `tools/call`).

Send `Accept: application/json, text/event-stream` and inspect the response's `Content-Type`. The JSON-RPC envelope can arrive as an `application/json` body or as JSON inside an SSE `data:` event. The JSON response examples below show the envelope or parsed tool content, not the transport framing. Check the JSON-RPC `error` and `result.isError` before consuming a tool result: HTTP `200` can contain a failed tool call.

## Access methods

### External access through ingress

```
https://<cluster-domain>/mcp
```

Requires a valid Bearer token (see [Authentication](#authentication)).

### Port forwarding (for development and testing)

```bash
kubectl port-forward --address 127.0.0.1 deployment/indexed-sources-mcp-server-deployment 8080:8080 -n arc-rag
```

Then call `http://127.0.0.1:8080/mcp`. Tool calls still require authentication and collection permissions when authentication is enabled. Keep the listener loopback-only.

---

## Authentication

Require authentication when `IS_AUTH_ENABLED=true` (the default). Authentication only applies to `tools/call` requests. Discovery methods (`initialize`, `tools/list`) don't require a token.

Pass the token as a Bearer token in the `Authorization` header. Dapr forwards the token to downstream services for RBAC enforcement on collection access.

Downstream services check permissions at the extension resource scope first:

| Required Azure role | Operations | App-role fallback |
|---|---|---|
| **Cognitive Services OpenAI User** | Search tools (`search_hybrid`, `search_vector`, `search_text`, `search_image`, `search_multimodal`) | `EdgeRAGDeveloper`, or `EdgeRAGEndUser` with a matching collection app role |
| **Reader** | `get_available_collections` | `EdgeRAGDeveloper`, or `EdgeRAGEndUser` with matching collection app roles |

In disconnected deployments, use **Contributor** for search tools and **Reader** (or **Contributor**) for `get_available_collections`.

### Get a token with Azure CLI

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive an authentication error, acquire a fresh token.

## Collection scope

Search calls are restricted by the `X-Allowed-Collections` request header. A managed knowledge-base caller sets this header from the indexed knowledge sources attached to the knowledge base.

- A direct MCP client must send `X-Allowed-Collections` on search tool calls.
- If you omit `collection_names`, the server searches all collections named in `X-Allowed-Collections`.
- If you provide `collection_names`, every requested collection must be present in `X-Allowed-Collections`.
- If the request doesn't have an allowed-collection scope, the search fails closed.

Call `get_available_collections` to list the collections available to the caller before you search.

## Connection

MCP requires a session handshake before tool calls: send an `initialize` request, capture the `Mcp-Session-Id` response header, then include it in all subsequent requests. MCP-compatible clients handle this automatically. For raw `curl` testing, follow the manual steps.

```bash
# Step 1 — Initialize
# The response body contains server capabilities. The Mcp-Session-Id is
# returned in the response headers — extract it and pass it to all
# subsequent requests.
curl -si -X POST http://127.0.0.1:8080/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'

# Example response headers:
#   HTTP/1.1 200 OK
#   Mcp-Session-Id: aaaa0000-bb11-2222-33cc-444444dddddd
#   Content-Type: text/event-stream

# Step 2 — List tools (pass the Mcp-Session-Id from Step 1)
curl -s -X POST http://127.0.0.1:8080/mcp \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Mcp-Session-Id: aaaa0000-bb11-2222-33cc-444444dddddd" \
  -d '{"jsonrpc":"2.0","id":2,"method":"tools/list"}'
```

## Tools

## search_hybrid

Hybrid retrieval combines dense (semantic) and sparse (learned keyword) vector search, with results merged and reranked. Both vectors are generated by the BGE-M3 model. This method is the recommended default search mode.

### Request

Send a `SearchToolRequest` object under `params.arguments.request`. The fields below belong inside that object. Inspect `tools/list` for the server's current input schema.

| Parameter | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `query` | `string` | Yes | — | — | Search query text |
| `collection_names` | `list[string] \| string \| null` | No | `null` | — | Collections to search (a list of names or a single collection name). Omit to use all collections in `X-Allowed-Collections`. |
| `top_n` | `integer` | No | `3` | 1–50 | Number of results to return |
| `strictness` | `integer` | No | `1` | 0–5 | Relevance threshold. `0` returns all results; higher values filter out lower-scoring chunks |
| `filters` | `string` | No | `null` | — | Reserved. The current search implementation doesn't apply this value. |

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_hybrid",
    "arguments": {
      "request": {
        "query": "How do I reset my device?",
        "collection_names": ["edgeragapp"],
        "top_n": 5,
        "strictness": 1
      }
    }
  }
}
```

#### Response

Tool responses are returned inside the standard MCP JSON-RPC envelope as a text content block. The tool output is a JSON string in the `text` field:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "result": {
    "content": [
      {
        "type": "text",
        "text": "{\"edgeragapp\": {\"results\": [...], \"metadata\": {...}}}"
      }
    ]
  }
}
```

Parsed tool output:

```json
{
  "edgeragapp": {
    "results": [
      {
        "content": "To reset your device, press and hold the power button for 10 seconds...",
        "file_path": "10.244.3.70:/exports/manuals/device-guide.pdf",
        "chunk_id": "3",
        "score": 0.92,
        "page_numbers": [12],
        "vector_type": "dense",
        "title": "device-guide",
        "document_type": "PDF",
        "last_modified": 1792661000.0
      }
    ],
    "metadata": {
      "results_number": 1,
      "search_type": "hybrid_search",
      "collection": "edgeragapp"
    }
  }
}
```

The response is keyed by the collection name prefix you provided (for example, `edgeragapp`). When querying multiple collections, each appears as a separate key in the response.

Response model: `SearchChunksResponse`

## search_vector

Pure semantic (vector) retrieval. Best when query wording differs from document language.

### Request

Same as [search_hybrid](#request) - uses `SearchToolRequest`.

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_vector",
    "arguments": {
      "request": {
        "query": "power cycling procedure",
        "collection_names": ["edgeragapp"],
        "top_n": 3,
        "strictness": 2
      }
    }
  }
}
```

#### Response

Same structure as `search_hybrid`. `metadata.search_type` is `"vector_search"`.

Response model: `SearchChunksResponse`

## search_text

Sparse vector keyword search using BGE-M3 learned sparse representations. Best for exact terms, codes, and names.

### Request

Same as [search_hybrid](#request) - uses `SearchToolRequest`.

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_text",
    "arguments": {
      "request": {
        "query": "ERR-4012",
        "collection_names": ["edgeragapp"],
        "top_n": 10,
        "strictness": 0
      }
    }
  }
}
```

#### Response

Same structure as `search_hybrid`. `metadata.search_type` is `"full_text_search"`.

Response model: `SearchChunksResponse`

## search_image

Image retrieval by using vision embeddings.

### Request

Same as [search_hybrid](#request) - uses `SearchToolRequest`.

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_image",
    "arguments": {
      "request": {
        "query": "wiring diagram for sensor module",
        "collection_names": ["edgeragapp"],
        "top_n": 5,
        "strictness": 1
      }
    }
  }
}
```

#### Response

Same structure as `search_hybrid`. `metadata.search_type` is `"image_search"` and `vector_type` is `"image"`.

Response model: `SearchChunksResponse`

## search_multimodal

Runs hybrid search (dense and sparse) and image search in parallel. Returns separate `text_chunks` and `image_chunks`.

### Request

Send a `MultimodalSearchRequest` object under `params.arguments.request`.

| Parameter | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `query` | `string` | Yes | — | — | Search query |
| `collection_names` | `list[string] \| string \| null` | No | `null` | — | Collections to search (a list of names or a single collection name). Omit to use all collections in `X-Allowed-Collections`. |
| `top_n` | `integer` | No | `3` | 1–50 | Number of results per modality |
| `text_strictness` | `integer` | No | `1` | 0–5 | Text relevance threshold. `0` returns all results; higher values filter out lower-scoring chunks |
| `image_strictness` | `integer` | No | `1` | 0–5 | Image relevance threshold. Same scale as `text_strictness` |
| `filters` | `string` | No | `null` | — | Reserved. The current search implementation doesn't apply this value. |

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search_multimodal",
    "arguments": {
      "request": {
        "query": "installation steps with diagrams",
        "collection_names": ["edgeragapp"],
        "top_n": 5,
        "text_strictness": 2,
        "image_strictness": 1
      }
    }
  }
}
```

#### Response

```json
{
  "edgeragapp": {
    "text_chunks": [
      {
        "content": "Step 1: Mount the bracket to the wall...",
        "file_path": "10.244.3.70:/exports/manuals/install-guide.pdf",
        "chunk_id": "7",
        "score": 0.88,
        "page_numbers": [3, 4],
        "vector_type": "dense"
      }
    ],
    "image_chunks": [
      {
        "content": "",
        "file_path": "10.244.3.70:/exports/manuals/install-guide.pdf",
        "chunk_id": "img-2",
        "score": 0.76,
        "page_numbers": [4],
        "vector_type": "image"
      }
    ],
    "metadata": {
      "results_number": 2,
      "search_type": "multimodal_search",
      "collection": "edgeragapp"
    }
  }
}
```

Response model: `MultimodalSearchResponse`

## get_available_collections

Lists all collections the authenticated user has access to, restricted by `X-Allowed-Collections` when the header is present.

### Request

The tool accepts an optional `random_string` parameter. The value is ignored and exists for compatibility with clients that don't reliably emit an empty argument object.

### Example

#### Request

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "get_available_collections",
    "arguments": {
      "random_string": ""
    }
  }
}
```

#### Response

```json
{
  "collections": [
    {
      "name": "edgeragapp",
      "description": "Default system collection (auto-created)",
      "status": "active",
      "created_at": "2026-02-25T15:32:19.097957+00:00"
    },
    {
      "name": "my-docs",
      "description": "Technical manuals",
      "status": "active",
      "created_at": "2026-03-04T12:00:00.000000+00:00"
    }
  ],
  "total_count": 2
}
```

Response model: `CollectionsResponse`

## Request schemas

### `SearchToolRequest`

Used by: `search_hybrid`, `search_vector`, `search_text`, `search_image`.

This schema is the value of `params.arguments.request`, not the schema of the complete `arguments` object.

| Field | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `query` | `string` | Yes | — | — | Search query text |
| `collection_names` | `list[string] \| string \| null` | No | `null` | — | Collections to search (a list of names or a single collection name). Omit to use all collections in `X-Allowed-Collections`. |
| `top_n` | `integer` | No | `3` | `ge=1`, `le=50` | Number of results (1–50) |
| `strictness` | `integer` | No | `1` | `ge=0`, `le=5` | Relevance threshold (0–5). `0` returns all results; higher values filter out lower-scoring chunks |
| `filters` | `string \| null` | No | `null` | — | Reserved. The current search implementation doesn't apply this value. |

### `MultimodalSearchRequest`

Used by: `search_multimodal`.

This schema is the value of `params.arguments.request`. Collection discovery uses its own schema and doesn't use this wrapper.

| Field | Type | Required | Default | Constraints | Description |
|---|---|---|---|---|---|
| `query` | `string` | Yes | — | — | Search query |
| `collection_names` | `list[string] \| string \| null` | No | `null` | — | Collections to search (a list of names or a single collection name). Omit to use all collections in `X-Allowed-Collections`. |
| `top_n` | `integer` | No | `3` | `ge=1`, `le=50` | Number of results per modality (1–50) |
| `text_strictness` | `integer` | No | `1` | `ge=0`, `le=5` | Text relevance threshold (0–5). `0` returns all results; higher values filter out lower-scoring chunks |
| `image_strictness` | `integer` | No | `1` | `ge=0`, `le=5` | Image relevance threshold (0–5). Same scale as `text_strictness` |
| `filters` | `string \| null` | No | `null` | — | Reserved. The current search implementation doesn't apply this value. |

## Response schemas

### `ToolChunkResult`

A single chunk result from a search operation.

| Field | Type | Description |
|---|---|---|
| `content` | `string` | Chunk text content |
| `file_path` | `string` | Source document path |
| `chunk_id` | `string` | Chunk position in document |
| `score` | `float` | Relevance score (0.0–1.0) |
| `page_numbers` | `list[integer]` | Pages where chunk appears |
| `vector_type` | `string` | Embedding type (`dense`, `sparse`, or `image`). Default: `"dense"` |
| `title` | `string` | Display title derived from the source file name |
| `document_type` | `string` | Display document type derived from the source file extension |
| `last_modified` | `float \| null` | Source last-modified timestamp in Unix epoch seconds, when available |

### `SearchMetadata`

Metadata about a search operation.

| Field | Type | Description |
|---|---|---|
| `results_number` | `integer` | Total number of results returned |
| `search_type` | `string` | Type of search performed (for example, `hybrid_search`, `vector_search`, `full_text_search`, `image_search`, `multimodal_search`) |
| `collection` | `string` | Milvus collection name used |

### `SearchChunksResponse`

Returned by: `search_hybrid`, `search_vector`, `search_text`, `search_image`.

| Field | Type | Description |
|---|---|---|
| `results` | `list[ToolChunkResult]` | List of matching chunks |
| `metadata` | `SearchMetadata` | Search operation metadata |

### `MultimodalSearchResponse`

Returned by: `search_multimodal`.

| Field | Type | Description |
|---|---|---|
| `text_chunks` | `list[ToolChunkResult]` | Text search results |
| `image_chunks` | `list[ToolChunkResult]` | Image search results (IDs only) |
| `metadata` | `SearchMetadata` | Search metadata |

### `CollectionInfo`

| Field | Type | Description |
|---|---|---|
| `name` | `string` | Collection name |
| `description` | `string \| null` | Collection description |
| `status` | `string` | Collection status, such as `"active"` or `"deleting"` |
| `created_at` | `string` | ISO 8601 creation timestamp |

### `CollectionsResponse`

Returned by: `get_available_collections`.

| Field | Type | Description |
|---|---|---|
| `collections` | `list[CollectionInfo]` | List of accessible collections |
| `total_count` | `integer` | Total number of collections returned |

---

## Response codes

The JSON-RPC response returns errors in the `error` field. These codes apply to all tools.

| Code | Description |
|---|---|
| `-32600` | Invalid Request — the JSON-RPC request is malformed. |
| `-32601` | Method not found — the requested MCP method doesn't exist. |
| `-32602` | Invalid params — tool arguments failed schema validation (for example, `top_n` out of range). |
| `-32603` | Internal error — the server or tool call failed unexpectedly. |

## Related content

- [Indexed Sources MCP overview](../mcp-server-overview.md) — Concepts, connecting clients, and registering external MCP servers
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG
- [Collections API Reference](collections-api-reference.md) — Create and manage collections
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections
- [Knowledge sources API reference](knowledge-sources-api-reference.md) — Register and manage knowledge sources
- [Agent Service overview](../agentic-overview.md) — How agents use MCP tools
