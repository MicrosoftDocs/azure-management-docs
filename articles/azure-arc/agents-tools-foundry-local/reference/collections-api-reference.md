---
title: Collections REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for managing data collections in Agents and Tools with Foundry Local.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/27/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
---

# Collections REST API reference

Manage collections in Agents and Tools with Foundry Local. Collections are logical groupings that organize ingested data. Each collection maps to a set of vector collections and database tables.

A collection is identified by its **name** (there is no separate ID). The name is immutable after creation.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## API information

All endpoints require the `api-version=2024-10-01-preview` query parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Collections API (`vectordb-api-server`) |
| **Port** | 3002 |
| **Dapr App ID** | `vectordb-api-server` |


## Access Methods

You can reach the Collections API through three access methods. The access method determines whether authentication is required.

### External access (via ingress)

```
https://<cluster-domain>/edgeai/collections...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header which triggers Entra Auth sidecar validation.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/vectordb-api-server-deployment 3002:3002 -n arc-rag
```

Then call `http://localhost:3002/edgeai/collections...`. No `Authorization` header needed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: vectordb-api-server" http://localhost:3500/edgeai/collections...
```

No `Authorization` header needed for internal Dapr calls.

---

## Authentication

Required only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

All external API calls require a valid Entra ID JWT token with the `EdgeRAGDeveloper` app role.

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

---

## Collection name rules

- Use only lowercase letters, digits, and hyphens.
- Start and end with an alphanumeric character.
- Use 2 to 49 characters.
- The names `default` and `system` are reserved.
- The default collection `edgeragapp` is auto-created on startup and can't be deleted.

Internally, the system replaces hyphens in collection names with underscores to form the `storage_prefix`. This prefix is used for Milvus collection names and Postgres table names. For example, the collection `my-docs` uses the storage prefix `my_docs`, which results in Milvus collections like `my_docs_BGE_M3`.


## Create

Creates a new collection.

### Request

```http
POST /edgeai/collections?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `name` | True | String | Collection name. See [Collection Name Rules](#collection-name-rules). |
| `description` | False | String | Optional description. Maximum 500 characters. |

### Example

#### Request

```http
POST /edgeai/collections?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "name": "my-collection",
  "description": "Optional description, max 500 chars"
}
```

#### Response

**Status code:** 201 Created

```json
{
  "name": "my-collection",
  "description": "Optional description, max 500 chars",
  "created_at": "2026-03-04T12:00:00.000000+00:00",
  "created_by": "<your-display-name>",
  "status": "active",
  "last_ingestion_at": null
}
```

### Response codes

| Code | Description |
|---|---|
| 201 | Created. The collection was successfully created. Four Milvus collections are provisioned (`{prefix}_BGE_M3`, `{prefix}_CLIP_ViT_L_14`, `{prefix}_filemetadata_BGE_M3`, `{prefix}_filemetadata_CLIP_ViT_L_14`) and metadata is saved to the Dapr state store. |
| 400 | Bad Request. Invalid name (bad characters, too short or long, or reserved name). |
| 409 | Conflict. A collection with this name already exists. |

---

## List

Returns all collections, sorted by `created_at` descending (newest first).

### Request

```http
GET /edgeai/collections?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
GET /edgeai/collections?api-version=2024-10-01-preview

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "collections": [
    {
      "name": "my-collection",
      "description": "My documents",
      "created_at": "2026-03-04T12:00:00.000000+00:00",
      "created_by": "<your-display-name>",
      "status": "active",
      "last_ingestion_at": "2026-03-04T13:00:00.000000+00:00"
    },
    {
      "name": "edgeragapp",
      "description": "Default system collection (auto-created)",
      "created_at": "2026-02-25T15:32:19.097957+00:00",
      "created_by": null,
      "status": "active",
      "last_ingestion_at": null
    }
  ],
  "total_count": 2
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the list of collections. |

---

## Get

Gets a single collection by name.

### Request

```http
GET /edgeai/collections/{name}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `name` | path | True | String | The collection name. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
GET /edgeai/collections/my-collection?api-version=2024-10-01-preview

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "name": "my-collection",
  "description": "My documents",
  "created_at": "2026-03-04T12:00:00.000000+00:00",
  "created_by": "<your-display-name>",
  "status": "active",
  "last_ingestion_at": "2026-03-04T13:00:00.000000+00:00"
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the requested collection. |
| 404 | Not Found. No collection with the specified name exists. |

## Update

Updates a collection. You can only update the `description` field. The `name` is immutable after creation.

### Request

```http
PATCH /edgeai/collections/{name}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `name` | path | True | String | The collection name. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `description` | True | String | The updated description. Maximum 500 characters. |

### Example

#### Request

```http
PATCH /edgeai/collections/my-collection?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "description": "Updated description"
}
```

#### Response

**Status code:** 200 OK

Returns the updated collection object.

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The collection was updated successfully. |
| 404 | Not Found. No collection with the specified name exists. |

---

## Delete

Deletes a collection and all associated resources (Milvus collections, Postgres tables, and metadata).

### Request

```http
DELETE /edgeai/collections/{name}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `name` | path | True | String | The collection name. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
DELETE /edgeai/collections/my-collection?api-version=2024-10-01-preview

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "status": "deleted",
  "deleted_resources": {
    "milvus_collections": [
      "my_collection_BGE_M3",
      "my_collection_CLIP_ViT_L_14",
      "my_collection_filemetadata_BGE_M3",
      "my_collection_filemetadata_CLIP_ViT_L_14"
    ],
    "postgres_tables": ["my_collection_*"]
  }
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The collection and all associated resources were deleted. |
| 409 | Conflict. The collection is `edgeragapp` (default, can't be deleted) or active ingestion jobs are running against this collection. |

## Related content

- [Collections Overview](../collections-overview.md) — Concepts, architecture, and when to use multiple collections
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG
- [MCP Server API Reference](mcp-server-api-reference.md) — Search tools that query collections
- [Agentic Layer Overview](../agentic-overview.md) — How agents connect to collections via knowledge sources
