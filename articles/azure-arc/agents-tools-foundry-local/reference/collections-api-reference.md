---
title: Collections REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for managing data collections in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
---

# Collections REST API reference

Manage collections in Agents and Tools with Foundry Local. Collections are logical groupings that organize ingested data. Each collection maps to a set of vector collections and database tables.

A collection is identified by its **name** (there's no separate ID). The name is immutable after creation.

## API information

All endpoints require the `api-version=2024-10-01-preview` query parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Collections API (`vectordb-api-server`) |
| **Port** | 3002 |
| **Dapr App ID** | `vectordb-api-server` |


## Access methods

You can reach the Collections API through three access methods. The access method determines whether authentication is required.

### External access (via ingress)

```
https://<cluster-domain>/collections...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header which triggers Entra Auth sidecar validation.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/vectordb-api-server-deployment 3002:3002 -n arc-rag
```

Then call `http://localhost:3002/collections...`. No `Authorization` header needed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: vectordb-api-server" http://localhost:3500/collections...
```

No `Authorization` header needed for internal Dapr calls.

---

## Authentication

Require authentication only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

External calls require a valid Microsoft Entra ID access token for the extension's app registration. Azure role-based access control (Azure RBAC) checks permissions at the extension resource scope first:

| Azure role | Operation | App-role fallback |
|---|---|---|
| **Reader** | List or get collections | `EdgeRAGDeveloper` or `EdgeRAGEndUser`; collection filters apply below |
| **Cognitive Services Contributor** | Create, update, or delete collections | `EdgeRAGDeveloper` |

The actions are `Microsoft.CognitiveServices/accounts/collections/read`, `write`, and `delete`. **Contributor** and **Owner** also allow management operations. In disconnected deployments, use **Reader** for reads and **Contributor** for changes.

Azure-role-authorized callers can read every collection. App-role fallback keeps the per-collection filter: `EdgeRAGDeveloper` can access all collections, while `EdgeRAGEndUser` needs a role matching each collection name.

The extension's managed identity needs **Reader** at the extension scope to read role assignments. App-only callers can use Azure roles without a `roles` claim. See [Assign Azure roles](../prepare-authentication.md#assign-azure-roles).

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

Error responses from the Collections API use the envelope `{"error": {"message": "...", "type": "...", "param": "...", "code": "..."}}`.

---

## Collection name rules

- Use only lowercase letters, digits, and hyphens.
- Start with a lowercase letter and end with a lowercase letter or digit.
- Use 2 to 49 characters.
- The names `default`, `system`, `edgeragenduser`, and `edgeragdeveloper` are reserved.
- The default collection `edgeragapp` is auto-created on startup and can't be deleted.

Internally, the system replaces hyphens in collection names with underscores to form the `storage_prefix`. This prefix is used for Milvus collection names and Postgres table names. For example, the collection `my-docs` uses the storage prefix `my_docs`, which results in Milvus collections like `my_docs_BGE_M3`.

## Create

Creates a new collection.

### Request

```http
POST /collections?api-version=2024-10-01-preview
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
POST /collections?api-version=2024-10-01-preview

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
| 201 | Created. The collection was successfully created. The service creates the text Milvus collections (`{prefix}_BGE_M3` and `{prefix}_filemetadata_BGE_M3`), creates the image Milvus collections (`{prefix}_CLIP_ViT_L_14` and `{prefix}_filemetadata_CLIP_ViT_L_14`) only when GPU image embedding is enabled, and saves metadata to the Dapr state store. |
| 400 | Bad Request. Invalid name (bad characters, too short or long, or reserved name). |
| 409 | Conflict. A collection with this name already exists. |

---

## List

Returns all collections, sorted by `created_at` descending (newest first).

### Request

```http
GET /collections?api-version=2024-10-01-preview
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
GET /collections?api-version=2024-10-01-preview

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
GET /collections/{name}?api-version=2024-10-01-preview
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
GET /collections/my-collection?api-version=2024-10-01-preview

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
| 403 | Forbidden. Neither Azure roles nor app-role fallback allow access to this collection. |
| 404 | Not Found. No collection with the specified name exists. |

## Update

Updates a collection. You can only update the `description` field. The `name` is immutable after creation.

### Request

```http
PATCH /collections/{name}?api-version=2024-10-01-preview
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
| `description` | False | String | The updated description. Maximum 500 characters. |

### Example

#### Request

```http
PATCH /collections/my-collection?api-version=2024-10-01-preview

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
| 409 | Conflict. The collection is being deleted or updated by another operation. |

---

## Delete

Deletes a collection and all associated resources (Milvus collections, Postgres tables, and metadata).

### Request

```http
DELETE /collections/{name}?api-version=2024-10-01-preview
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
DELETE /collections/my-collection?api-version=2024-10-01-preview

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
| 404 | Not Found. No collection with the specified name exists. |
| 409 | Conflict. The collection is `edgeragapp` (default, can't be deleted) or active ingestion jobs are running against this collection. |

## Related content

- [Collections Overview](../collections-overview.md) — Concepts, architecture, and when to use multiple collections
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG
- [MCP Server API Reference](mcp-server-api-reference.md) — Search tools that query collections
- [Agent Service overview](../agentic-overview.md) — How agents connect to collections via knowledge sources
