---
title: Knowledge Sources REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for managing knowledge sources in Agents and Tools with Foundry Local.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/27/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
---

# Knowledge Sources REST API reference

Manage knowledge sources in Agents and Tools with Foundry Local. A knowledge source is a self-contained registration of an MCP server connection, optionally bound to a specific indexed source reference. Knowledge sources are the units that agents and knowledge bases consume to access external knowledge.

Two kinds of knowledge sources exist:

- **`remote_mcp`** — An arbitrary external MCP server (any MCP-compliant endpoint).
- **`indexed_sources_mcp`** — An `indexed-source-mcp` server connected to a specific indexed source reference.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## API information

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Knowledge Sources API (`knowledge-sources`) |
| **Port** | 3005 |
| **Dapr App ID** | `knowledge-sources` |

All endpoints use the `/edgeai/knowledgesources` prefix.

## Access Methods

You can reach the Knowledge Sources API through three access methods. The access method determines whether authentication is required.

### External access (via ingress)

```
https://<cluster-domain>/edgeai/knowledgesources/...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header which triggers Entra Auth sidecar validation.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/knowledgesources-deployment 3005:3005 -n arc-rag
```

Then call `http://localhost:3005/edgeai/knowledgesources/...`. No `Authorization` header needed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: knowledge-sources" http://localhost:3500/edgeai/knowledgesources/...
```

No `Authorization` header needed for internal Dapr calls.

## Authentication

Required only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

All external API calls require a valid Entra ID JWT token with the `EdgeRAGDeveloper` app role.

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "<your-resource-app-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

## Concepts

### Knowledge source

A knowledge source is a **registration of an MCP server**. It holds the connection configuration (server URL, auth type) and, for indexed-source MCPs, the reference to the specific indexed source.

### Kinds

| `kind` | Required parameters field | Use case |
|---|---|---|
| `remote_mcp` | `remote_mcp_parameters` | Register any external MCP server. |
| `indexed_sources_mcp` | `indexed_sources_parameters` | Register an `indexed-source-mcp` server pointing at a specific indexed source. |

### Auth Types

### Validation Status
| Value | Description |
|---|---|
| `microsoft_entra_id` | Forwards the caller's `Authorization` header to the MCP server. |
| `unauthenticated` | No auth headers are forwarded to the MCP server. |

### MCP validation

When you **create** a knowledge source or **update** its connection parameters, the service validates the MCP connection before saving it:

1. Extracts `server_url` from the parameters.
1. Performs an SSRF/DNS safety check (blocks private/internal IPs).
1. Sends a JSON-RPC `initialize` request to the MCP server (connect timeout: 5 seconds, read timeout: 30 seconds).
1. Retries up to three times with exponential backoff (1 second, 2 seconds).
1. If validation **succeeds**: sets `validation_status` to `active` and saves the record.
1. If validation **fails**: rejects the request with `400 Bad Request` - doesn't save anything (on create) or change anything (on update).

| Status | Description |
|---|---|
| `unknown` | Default status. Not validated yet. |
| `active` | MCP `initialize` succeeded. `validated_at` is set. |
| `failed` | Validation failed. Check `validation_error` for details. `validated_at` is set. |

## Create

Creates a new knowledge source. The service validates the MCP server before saving the record.

### Request

```http
POST /edgeai/knowledgesources
```

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `name` | True | String | Unique human-readable name. 1-256 characters. |
| `kind` | True | String | `"remote_mcp"` or `"indexed_sources_mcp"`. |
| `auth_type` | False | String | `"microsoft_entra_id"` (default) or `"unauthenticated"`. |
| `description` | False | String | Optional description. |
| `created_by` | False | String | Optional creator identifier. |
| `remote_mcp_parameters` | Conditional | Object | Required when `kind` is `"remote_mcp"`. See below. |
| `indexed_sources_parameters` | Conditional | Object | Required when `kind` is `"indexed_sources_mcp"`. See below. |

#### `remote_mcp_parameters`

| Name | Required | Type | Description |
|---|---|---|---|
| `server_url` | True | String | URL of the MCP server endpoint. |
| `server_label` | False | String | Human-readable label for the server. |

#### `indexed_sources_parameters`

| Name | Required | Type | Description |
|---|---|---|---|
| `server_url` | True | String | URL of the indexed-source-mcp server endpoint. |
| `indexed_source_ref` | True | String | The name of the indexed source to connect to. |
| `server_label` | False | String | Human-readable label. Defaults to `"indexed-sources-mcp-server"`. |

> [!NOTE]
> Provide only one of `remote_mcp_parameters` or `indexed_sources_parameters`. The provided parameter must match the `kind`.

### Example: Remote MCP

#### Request

```http
POST /edgeai/knowledgesources

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "name": "my-external-mcp",
  "kind": "remote_mcp",
  "auth_type": "unauthenticated",
  "description": "External MCP server for document search",
  "remote_mcp_parameters": {
    "server_url": "https://my-mcp-server.example.com/mcp",
    "server_label": "doc-search"
  }
}
```

#### Response

**Status code:** 201 Created

```json
{
  "id": "f1e2d3c4-b5a6-7890-abcd-ef1234567890",
  "name": "my-external-mcp",
  "kind": "remote_mcp",
  "description": "External MCP server for document search",
  "auth_type": "unauthenticated",
  "remote_mcp_parameters": {
    "server_url": "https://my-mcp-server.example.com/mcp",
    "server_label": "doc-search"
  },
  "indexed_sources_parameters": null,
  "validation_status": "active",
  "validation_error": null,
  "validated_at": "2026-03-29T12:00:00.000000+00:00",
  "created_by": null,
  "created_at": "2026-03-29T12:00:00.000000+00:00",
  "updated_at": "2026-03-29T12:00:00.000000+00:00"
}
```

### Example: Indexed Sources MCP

#### Request

```http
POST /edgeai/knowledgesources

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "name": "my-indexed-source",
  "kind": "indexed_sources_mcp",
  "auth_type": "microsoft_entra_id",
  "description": "Indexed source for product documentation",
  "indexed_sources_parameters": {
    "server_url": "https://indexed-sources-mcp.example.com/mcp",
    "indexed_source_ref": "product-docs-v2",
    "server_label": "indexed-sources-mcp-server"
  }
}
```

#### Response

**Status code:** 201 Created

```json
{
  "id": "a2b3c4d5-e6f7-8901-bcde-f23456789012",
  "name": "my-indexed-source",
  "kind": "indexed_sources_mcp",
  "description": "Indexed source for product documentation",
  "auth_type": "microsoft_entra_id",
  "remote_mcp_parameters": null,
  "indexed_sources_parameters": {
    "server_url": "https://indexed-sources-mcp.example.com/mcp",
    "indexed_source_ref": "product-docs-v2",
    "server_label": "indexed-sources-mcp-server"
  },
  "validation_status": "active",
  "validation_error": null,
  "validated_at": "2026-03-29T12:00:00.000000+00:00",
  "created_by": null,
  "created_at": "2026-03-29T12:00:00.000000+00:00",
  "updated_at": "2026-03-29T12:00:00.000000+00:00"
}
```

### Response codes

| Code | Description |
|---|---|
| 201 | Created. The knowledge source was created and MCP validation succeeded. |
| 400 | Bad Request. MCP validation failed (server unreachable or protocol error). |
| 409 | Conflict. A knowledge source with the same `name` already exists. |
| 422 | Unprocessable Entity. Missing required fields, invalid field values, or parameters/kind mismatch. |

---

## List

Returns knowledge sources with pagination and optional filtering.

### Request

```http
GET /edgeai/knowledgesources
```

### Query parameters

| Name | Required | Type | Description |
|---|---|---|---|
| `name` | False | String | Filter by exact name. |
| `kind` | False | String | Filter by kind (`remote_mcp` or `indexed_sources_mcp`). |
| `limit` | False | Integer | Maximum number of results. Default: `10`. Range: 1-100. |
| `offset` | False | Integer | Number of results to skip. Default: `0`. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
GET /edgeai/knowledgesources?kind=indexed_sources_mcp&limit=10&offset=0

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "items": [
    {
      "id": "a2b3c4d5-e6f7-8901-bcde-f23456789012",
      "name": "my-indexed-source",
      "kind": "indexed_sources_mcp",
      "description": "Indexed source for product documentation",
      "auth_type": "microsoft_entra_id",
      "remote_mcp_parameters": null,
      "indexed_sources_parameters": {
        "server_url": "https://indexed-sources-mcp.example.com/mcp",
        "indexed_source_ref": "product-docs-v2",
        "server_label": "indexed-sources-mcp-server"
      },
      "validation_status": "active",
      "validation_error": null,
      "validated_at": "2026-03-29T12:00:00.000000+00:00",
      "created_by": null,
      "created_at": "2026-03-29T12:00:00.000000+00:00",
      "updated_at": "2026-03-29T12:00:00.000000+00:00"
    }
  ],
  "total": 1,
  "limit": 10,
  "offset": 0
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the paginated list of knowledge sources. |

---

## Get

Retrieves a single knowledge source by ID.

### Request

```http
GET /edgeai/knowledgesources/{ks_id}
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `ks_id` | path | True | UUID | The knowledge source identifier. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
GET /edgeai/knowledgesources/a2b3c4d5-e6f7-8901-bcde-f23456789012

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "id": "a2b3c4d5-e6f7-8901-bcde-f23456789012",
  "name": "my-indexed-source",
  "kind": "indexed_sources_mcp",
  "description": "Indexed source for product documentation",
  "auth_type": "microsoft_entra_id",
  "remote_mcp_parameters": null,
  "indexed_sources_parameters": {
    "server_url": "https://indexed-sources-mcp.example.com/mcp",
    "indexed_source_ref": "product-docs-v2",
    "server_label": "indexed-sources-mcp-server"
  },
  "validation_status": "active",
  "validation_error": null,
  "validated_at": "2026-03-29T12:00:00.000000+00:00",
  "created_by": null,
  "created_at": "2026-03-29T12:00:00.000000+00:00",
  "updated_at": "2026-03-29T12:00:00.000000+00:00"
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the requested knowledge source. |
| 404 | Not Found. No knowledge source with the specified ID exists. |

---

## Update

Partially updates a knowledge source. If you change connection parameters or `auth_type`, the system re-runs MCP validation. If revalidation fails, the system doesn't update any fields.

### Request

```http
PATCH /edgeai/knowledgesources/{ks_id}
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `ks_id` | path | True | UUID | The knowledge source identifier. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

All fields are optional. Only the fields you provide are updated.

| Name | Required | Type | Description |
|---|---|---|---|
| `name` | False | String | Updated name. 1-256 characters. |
| `description` | False | String | Updated description. |
| `auth_type` | False | String | Updated auth type. Triggers MCP revalidation. |
| `remote_mcp_parameters` | False | Object | Updated parameters. Only valid for `kind=remote_mcp`. Triggers MCP revalidation. |
| `indexed_sources_parameters` | False | Object | Updated parameters. Only valid for `kind=indexed_sources_mcp`. Triggers MCP revalidation. |

The `kind` field is immutable after creation. Supplying a parameters field that doesn't match the knowledge source's `kind` returns `400 Bad Request`.

### Example

#### Request

```http
PATCH /edgeai/knowledgesources/a2b3c4d5-e6f7-8901-bcde-f23456789012

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "name": "updated-indexed-source",
  "description": "Updated description"
}
```

#### Response

**Status code:** 200 OK

Returns the full updated knowledge source object.

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The knowledge source was updated. |
| 400 | Bad Request. MCP revalidation failed, or parameters/kind mismatch. |
| 404 | Not Found. No knowledge source with the specified ID exists. |
| 409 | Conflict. Updated `name` conflicts with an existing knowledge source. |
| 422 | Unprocessable Entity. Invalid field values. |

## Delete

Deletes a single knowledge source by ID.

### Request

```http
DELETE /edgeai/knowledgesources/{ks_id}
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `ks_id` | path | True | UUID | The knowledge source identifier. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
DELETE /edgeai/knowledgesources/a2b3c4d5-e6f7-8901-bcde-f23456789012

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 204 No Content

No response body.

### Response codes

| Code | Description |
|---|---|
| 204 | No Content. The knowledge source was deleted. |
| 404 | Not Found. No knowledge source with the specified ID exists. |

## Batch Delete

Deletes multiple knowledge sources by their IDs.

### Request

```http
POST /edgeai/knowledgesources/batch-delete
```

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `ks_ids` | True | Array of UUID | Knowledge source IDs to delete. At least one required. |

### Example

#### Request

```http
POST /edgeai/knowledgesources/batch-delete

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "ks_ids": [
    "f1e2d3c4-b5a6-7890-abcd-ef1234567890",
    "a2b3c4d5-e6f7-8901-bcde-f23456789012"
  ]
}
```

#### Response

**Status code:** 200 OK

```json
{
  "deleted_count": 2
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The matching knowledge sources are deleted. |
| 400 | Bad Request. No `ks_ids` provided. |

## Related content

- [Knowledge Sources Guide](../knowledge-sources-guide.md) — How-to guide for configuring knowledge sources
- [Knowledge Bases Guide](../knowledge-bases-guide.md) — Group knowledge sources into knowledge bases
- [Collections API Reference](collections-api-reference.md) — Create and manage collections
- [Collections Overview](../collections-overview.md) — Understanding the relationship between collections and knowledge sources
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG
