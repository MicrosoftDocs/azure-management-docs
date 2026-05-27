---
title: Knowledge Base Manager REST API reference - Agents and Tools with Foundry Local
description: REST API reference for managing knowledge bases in Agents and Tools with Foundry Local.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/24/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
---

# Knowledge Base Manager REST API reference

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Authorization

All endpoints require a valid Bearer token issued by Microsoft Entra ID (Azure AD). The token is validated by the Entra Auth sidecar running alongside each service.

### Obtaining a Token

Acquire an access token from Microsoft Entra ID using the OAuth 2.0 client credentials flow or authorization code flow. The token must be issued for the Agentic RAG app registration audience.

Using the Azure CLI:

```bash
az account get-access-token --resource api://<app-registration-client-id> --query accessToken -o tsv
```

### Passing the Token

Include the token in the `Authorization` header of every request:

```http
Authorization: Bearer <access_token>
```

### Roles

The token's `roles` claim is inspected for role-based access control:

| Role | Access |
| --- | --- |
| `EdgeRAGDeveloper` | Full access — view and update knowledge bases. |
| *(any authenticated user)* | Read-only access — list and get knowledge bases. |

Write operations (update) require the **`EdgeRAGDeveloper`** role. If the caller lacks this role, the API returns `403 Forbidden`.

### Disabling Authentication (Development)

Set `IS_AUTH_ENABLED=false` in the environment to disable authentication. When disabled, all endpoints are accessible without a token and no ownership is enforced.

> **Note: Timestamps** — The Knowledge Base Manager returns timestamps as ISO 8601 datetime strings (e.g., `"2025-01-15T10:30:00Z"`). The Agents Runtime Service uses Unix epoch integers (e.g., `1699012345`).

## List Knowledge Bases

Lists all knowledge bases with cursor-based pagination.

### Request

```http
GET https://{manager_host}/knowledge-bases?limit={limit}&order={order}&after={after}&before={before}
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service. | String |

### Query parameters

| Name | Description | Data Type |
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
Authorization: Bearer eyJ0eX ... FWSXfwtQ
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

## Get Knowledge Base

Retrieves a knowledge base by ID.

### Request

```http
GET https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service. | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`). | String |

### GET example

```http
GET https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Authorization: Bearer eyJ0eX ... FWSXfwtQ
```

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The response contains the requested knowledge base. |
| 404 | Not Found. The specified knowledge base does not exist. |
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

## Update Knowledge Base

Updates an existing knowledge base (partial update). Only provided fields will be updated.

> **Important:** When `knowledge_source_ids` is provided in a PATCH request, it **replaces** the entire list — it does not append. Include all desired IDs in the array. To add a new source, first read the current list, then send the full updated array. Omitting the field entirely leaves the existing list unchanged.

**Requires role: `EdgeRAGDeveloper`**

### Request

```http
PATCH https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service. | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`). | String |

### PATCH example

```http
PATCH https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Content-Type: application/json
Authorization: Bearer eyJ0eX ... FWSXfwtQ
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
| 403 | Forbidden. The caller does not have the `EdgeRAGDeveloper` role. |
| 404 | Not Found. The specified knowledge base does not exist. |
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

## Replace Knowledge Base

Fully replaces an existing knowledge base. All fields are overwritten with the provided values. Fields not included are reset to their defaults (e.g., `knowledge_source_ids` becomes `[]`, `description` becomes `null`).

**Requires role: `EdgeRAGDeveloper`**

### Request

```http
PUT https://{manager_host}/knowledge-bases/{kb_id}
```

### Uri parameters

| Name | Description | Data Type |
| --- | --- | --- |
| manager\_host | Hostname of the Agent Management Service. | String |
| kb\_id | Unique knowledge base identifier (any string, typically `asst_{uuid}`). | String |

### PUT example

```http
PUT https://localhost:8000/knowledge-bases/asst_abc123def456789012345678
```

### Header

```http
Content-Type: application/json
Authorization: Bearer eyJ0eX ... FWSXfwtQ
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

| Name | Required | Description | Data Type |
| --- | --- | --- | --- |
| name | **Yes** | Name of the knowledge base (1–256 chars, non-empty). | String |
| description | No | Description of the knowledge base (max 512 chars). Default: `null`. | String |
| knowledge\_source\_ids | No | List of Knowledge Source IDs. Default: `[]`. | Array\[String\] |
| metadata | No | Custom key-value metadata. Default: `null`. | Object |

### Response

#### Status codes

| Code | Description |
| --- | --- |
| 200 | OK. The knowledge base was replaced successfully. |
| 400 | Bad Request. Validation error. |
| 403 | Forbidden. The caller does not have the `EdgeRAGDeveloper` role. |
| 404 | Not Found. The specified knowledge base does not exist. |
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
