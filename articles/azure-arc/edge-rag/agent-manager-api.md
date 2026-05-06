---
title: Agent Management Service REST API - Azure Arc
description: Learn about the REST API for managing knowledge bases in the Azure Arc Agent Management Service.
ms.topic: reference
ms.date: 05/01/2026
author: cwatson-cat
ms.author: cwatson
ms.service: azure-arc
ms.subservice: edge-rag
---

# Agent Management Service REST API

Use this REST API to create, read, update, and delete knowledge bases in the Azure Arc Agent Management Service.

## Authorization

All endpoints require a valid Bearer token issued by Microsoft Entra ID. The MISE sidecar running alongside each service validates the token.

### Obtain a token

Acquire an access token from Entra ID by using the OAuth 2.0 client credentials flow or authorization code flow. The token must be issued for the Agents and Tools with Foundry Local app registration audience.

```azurecli
az account get-access-token --resource api://<app-registration-client-id> --query accessToken -o tsv
```

### Pass the token

Include the token in the `Authorization` header of every request:

```http
Authorization: Bearer <access-token>
```

### Roles

The API inspects the token's roles claim for role-based access control:

| Role | Access |
| --- | --- |
| **EdgeRAGDeveloper** | Full access: create, update, and delete knowledge bases. |
| *(any authenticated user)* | Read-only access: list and get knowledge bases. |

Write operations (create, update, and delete) require the **EdgeRAGDeveloper** role. If the caller lacks this role, the API returns `403 Forbidden`.

### Disable authentication (Development)

Set `IS_AUTH_ENABLED=false` in the environment to disable authentication. When disabled, all endpoints are accessible without a token and no ownership is enforced.

> [!NOTE]
> The Agent Management Service returns timestamps as ISO 8601 datetime strings (for example, "2025-01-15T10:30:00Z"). The Agents Runtime Service uses Unix epoch integers (for example, "1699012345").

## Create knowledge base

Creates a new knowledge base with optional knowledge source associations.

- Requires role: **EdgeRAGDeveloper**

### Request

```http
POST https://{manager_host}/knowledge-bases
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |

### POST example

```http
POST https://localhost:8000/knowledge-bases
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "name": "Product Documentation KB",
    "description": "Knowledge base containing product documentation and FAQs",
    "instructions": "You are a helpful assistant. Answer questions based on the knowledge base.",
    "temperature": 0.7,
    "top_p": 0.9,
    "knowledge_source_ids": [
        "ks_source1abc123",
        "ks_source2def456"
    ],
    "metadata": {
        "department": "engineering",
        "version": "2.0"
    }
}
```

### Body fields

| Name | Required | Description | Data type |
| --- | --- | --- | --- |
| name | **Yes** | Name of the knowledge base (1–256 characters, non-empty) | String |
| description | No | Description of the knowledge base (max 512 characters) | String |
| instructions | No | System instructions that guide the knowledge base's response behavior | String |
| temperature | No | Sampling temperature (0–2). Controls response randomness. Default: `1.0` | Float |
| top\_p | No | Nucleus sampling (0–1). Controls response diversity. Default: `1.0` | Float |
| knowledge\_source\_ids | No | List of Knowledge Source IDs to associate. Default: `[]` | Array\[String\] |
| metadata | No | Custom key-value metadata (max 16 pairs, 64-character keys, 512-character values) | Object |

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 201 | Created. The request was fulfilled and a new knowledge base was created. |
| 400 | Bad Request. Validation error. |
| 403 | Forbidden. The caller doesn't have the **EdgeRAGDeveloper** role. |
| 500 | Internal server error. |

#### Content-Type

`application/json`

#### Header

```http
HTTP/1.1 201 Created
```

#### Body

```json
{
    "data": {
        "id": "asst_abc123def456789012345678",
        "object": "knowledge_base",
        "created_at": "2025-01-15T10:30:00Z",
        "updated_at": null,
        "name": "Product Documentation KB",
        "description": "Knowledge base containing product documentation and FAQs",
        "knowledge_source_ids": [
            "ks_source1abc123",
            "ks_source2def456"
        ],
        "metadata": {
            "department": "engineering",
            "version": "2.0"
        }
    }
}
```

## List knowledge bases

Lists all knowledge bases with cursor-based pagination.

### Request

```http
GET https://{manager_host}/knowledge-bases?limit={limit}&order={order}&after={after}&before={before}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |

### Query parameters

| Name | Description | Data type |
| --- | --- | --- |
| limit | Optional. Maximum number of knowledge bases to return (1-100). Default is 20. | Integer |
| order | Optional. Sort order by created\_at. Values: `asc`, `desc`. Default is `desc`. | String |
| after | Optional. Cursor for forward pagination (KB ID). | String |
| before | Optional. Cursor for backward pagination (KB ID). | String |

### GET example

```http
GET https://localhost:8000/knowledge-bases?limit=10&order=desc
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. A successful operation with the knowledge base list. |
| 400 | Bad Request. Invalid cursor or query parameters. |
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
            "id": "asst_abc123def456789012345678",
            "object": "knowledge_base",
            "created_at": "2025-01-15T10:30:00Z",
            "updated_at": null,
            "name": "Product Documentation KB",
            "description": "Knowledge base containing product documentation and FAQs",
            "knowledge_source_ids": [
                "ks_source1abc123",
                "ks_source2def456"
            ],
            "metadata": {
                "department": "engineering"
            }
        }
    ],
    "first_id": "asst_abc123def456789012345678",
    "last_id": "asst_abc123def456789012345678",
    "has_more": false
}
```

## Get knowledge base

Retrieves a knowledge base by ID.

### Request

```http
GET https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`) | String |

### GET example

```http
GET https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested knowledge base. |
| 404 | Not Found. The specified knowledge base doesn't exist. |
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
    "data": {
        "id": "asst_abc123def456789012345678",
        "object": "knowledge_base",
        "created_at": "2025-01-15T10:30:00Z",
        "updated_at": null,
        "name": "Product Documentation KB",
        "description": "Knowledge base containing product documentation and FAQs",
        "knowledge_source_ids": [
            "ks_source1abc123",
            "ks_source2def456"
        ],
        "metadata": {
            "department": "engineering"
        }
    }
}
```

## Update knowledge base

Updates an existing knowledge base (partial update). Only the provided fields are updated.

> [!IMPORTANT]
> When you provide `knowledge_source_ids` in a PATCH request, you **replace** the entire list; you don't append. Include all desired IDs in the array. To add a new source, first read the current list, then send the full updated array. If you omit the field entirely, the existing list stays unchanged.

- Requires role: **EdgeRAGDeveloper**

### Request

```http
PATCH https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`) | String |

### PATCH example

```http
PATCH https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "name": "Updated Product KB",
    "description": "Updated knowledge base with new sources",
    "knowledge_source_ids": [
        "ks_source1abc123",
        "ks_source2def456",
        "ks_source3ghi789"
    ]
}
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The knowledge base was updated successfully. |
| 400 | Bad Request. Validation error. |
| 403 | Forbidden. The caller doesn't have the **EdgeRAGDeveloper** role. |
| 404 | Not Found. The specified knowledge base doesn't exist. |
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
    "data": {
        "id": "asst_abc123def456789012345678",
        "object": "knowledge_base",
        "created_at": "2025-01-15T10:30:00Z",
        "updated_at": "2025-01-15T11:00:00Z",
        "name": "Updated Product KB",
        "description": "Updated knowledge base with new sources",
        "knowledge_source_ids": [
            "ks_source1abc123",
            "ks_source2def456",
            "ks_source3ghi789"
        ],
        "metadata": {
            "department": "engineering"
        }
    }
}
```

## Replace knowledge base

Fully replaces an existing knowledge base. This operation overwrites all fields with the values you provide. Fields you don't include are reset to their default values (for example, `knowledge_source_ids` becomes `[]`, and `description` becomes `null`).

- Requires role: **EdgeRAGDeveloper**

### Request

```http
PUT https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`) | String |

### PUT example

```http
PUT https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

### Body example

```json
{
    "name": "Replaced Product KB",
    "description": "Completely replaced knowledge base",
    "knowledge_source_ids": [
        "ks_newSource1",
        "ks_newSource2"
    ],
    "metadata": {
        "department": "product"
    }
}
```

### Body fields

| Name | Required | Description | Data type |
| --- | --- | --- | --- |
| name | **Yes** | Name of the knowledge base (1–256 characters, non-empty). | String |
| description | No | Description of the knowledge base (max 512 chars). Default: `null`. | String |
| knowledge\_source\_ids | No | List of Knowledge Source IDs. Default: `[]`. | Array\[String\] |
| metadata | No | Custom key-value metadata. Default: `null`. | Object |

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The knowledge base was replaced successfully. |
| 400 | Bad Request. Validation error. |
| 403 | Forbidden. The caller doesn't have the **EdgeRAGDeveloper** role. |
| 404 | Not Found. The specified knowledge base doesn't exist. |
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
    "data": {
        "id": "asst_abc123def456789012345678",
        "object": "knowledge_base",
        "created_at": "2025-01-15T10:30:00Z",
        "updated_at": "2025-01-15T12:00:00Z",
        "name": "Replaced Product KB",
        "description": "Completely replaced knowledge base",
        "knowledge_source_ids": [
            "ks_newSource1",
            "ks_newSource2"
        ],
        "metadata": {
            "department": "product"
        }
    }
}
```

## Delete knowledge base

Deletes a knowledge base by ID.

- Requires role: **EdgeRAGDeveloper**

### Request

```http
DELETE https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`) | String |

### DELETE example

```http
DELETE https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Authorization: Bearer <access-token>
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The knowledge base was deleted successfully. |
| 403 | Forbidden. The caller doesn't have the **EdgeRAGDeveloper** role. |
| 404 | Not Found. The specified knowledge base doesn't exist. |
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
    "id": "asst_abc123def456789012345678",
    "object": "knowledge_base.deleted",
    "deleted": true
}
```
