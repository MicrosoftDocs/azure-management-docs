---
title: Inference REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for querying collections using RAG or querying the language model directly.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/27/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
---

# Inference REST API reference

Query collections by using Retrieval-Augmented Generation (RAG) or query the language model directly. The inference API searches a collection's vector data, retrieves relevant chunks, and generates a response grounded in the retrieved content.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## API information

The RAG endpoint requires the `api-version=2024-10-01-preview` query parameter. The model-only endpoint doesn't require this parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Inference API (`inferencing-flow`) |
| **Port** | 3001 |
| **Dapr App ID** | `inferencing-flow` |

## Access Methods

You can reach the Inference API through three access methods. The access method determines whether authentication and RBAC are enforced.

### External access (via ingress)

```
https://<cluster-domain>/edgeai/chat/...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header which triggers Entra Auth sidecar validation and RBAC enforcement.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/inferencingflow-deployment 3001:3001 -n arc-rag
```

Then call `http://localhost:3001/edgeai/chat/...`. No `Authorization` header needed. RBAC is bypassed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: inferencing-flow" http://localhost:3500/edgeai/chat/...
```

No `Authorization` header needed. RBAC is bypassed.

---

## Authentication

Required only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

Unlike the Collections and Ingestion APIs (which require `EdgeRAGDeveloper` only), the inference API accepts two roles:

| Role | Description |
|---|---|
| `EdgeRAGDeveloper` | Full access to all collections (bypasses RBAC). Required for Collections and Ingestion APIs. |
| `EdgeRAGEndUser` | Access only to collections where the user has a matching app role (for example, role `finance-docs` grants access to collection `finance-docs`). |

### Getting a token via Azure CLI

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

## Chat Completions (RAG)

Queries a collection by using Retrieval-Augmented Generation. Searches the collection's vector data, retrieves relevant chunks, and generates an LLM response that's grounded in the retrieved content.

### Request

```http
POST /edgeai/chat/completions?api-version=2024-10-01-preview
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
| `x-user-role` | True | String | `"dev"` — uses request body parameters directly and saves them to state store. `"user"` — ignores most request body parameters and loads developer-configured settings from state store. |

For API testing, always use `x-user-role: dev` so you control the parameters directly. The `x-user-role` header controls parameter behavior but doesn't affect RBAC. The JWT token's `roles` claim solely determines RBAC.

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `messages` | True | Array | Chat messages. Include `system` (optional), prior `user`/`assistant` turns for history, and final `user` message with the question. |
| `temperature` | False | Float | LLM temperature (0.0–1.0). Default: `0.7`. |
| `top_p` | False | Float | LLM top-p sampling. Default: `0.1`. |
| `isMarkdown` | False | Boolean | If `true`, LLM response is converted to markdown. Default: `false`. |
| `data_sources[0].type` | False | String | Data source type. Use `"milvus"`. |
| `data_sources[0].parameters.endpoint` | False | String | Milvus endpoint. Use `""` (empty string). |
| `data_sources[0].parameters.index_name` | False | String | Collection to query. Alias `collectionName` also accepted. Default: `"edgeragapp"`. |
| `data_sources[0].parameters.text_strictness` | False | Integer | Text relevance threshold (1–5). Higher = stricter filtering. Default: `2`. |
| `data_sources[0].parameters.image_strictness` | False | Integer | Image relevance threshold (1–5). Only used with multimodal search. Default: `1`. |
| `data_sources[0].parameters.top_n_documents` | False | Integer | Max number of chunks to retrieve. Default: `5`. |
| `data_sources[0].parameters.query_type` | False | String | Search mode. See [Query types](#query-types). Default: `"hybrid_search"`. |

### Query types

| Value | Description |
|---|---|
| `hybrid_search` | Combined dense + sparse vector search (default, recommended). |
| `vector_search` | Dense vector search only. |
| `full_text_search` | Sparse/keyword search only. |
| `hybrid_multimodal_search` | Hybrid search including image vectors. Returns text and image results. |

### Example

#### Request

```http
POST /edgeai/chat/completions?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
x-user-role: dev
```

```json
{
  "messages": [
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "What is absinthe?"}
  ],
  "temperature": 0.7,
  "top_p": 0.1,
  "data_sources": [
    {
      "type": "milvus",
      "parameters": {
        "endpoint": "",
        "index_name": "edgeragapp",
        "text_strictness": 2,
        "image_strictness": 1,
        "top_n_documents": 5,
        "query_type": "hybrid_search"
      }
    }
  ]
}
```

#### Response

**Status code:** 200 OK

```json
{
  "id": "chatcmpl-<unique-id>",
  "created": 1772357617,
  "choices": [
    {
      "content_filter_results": {},
      "message": {
        "content": "Absinthe is a distilled, highly alcoholic beverage...",
        "role": "assistant",
        "context": {
          "citations": [
            {
              "filepath": "10.244.3.70:/exports/basic/Absinthe.txt",
              "chunk_id": "0",
              "score": 4.95,
              "content": "Source text of the chunk...",
              "page_numbers": []
            }
          ]
        },
        "search_type": "hybrid_search",
        "top_n_documents": 5,
        "text_strictness": 2,
        "search_results": {
          "dense": [],
          "sparse": [],
          "image": []
        }
      }
    }
  ]
}
```

**Key response fields:**

| Field | Description |
|---|---|
| `choices[0].message.content` | The LLM's generated answer. |
| `choices[0].message.context.citations` | Retrieved chunks with file paths, scores, and content. |
| `choices[0].message.search_results` | Raw search results grouped by type (`dense`, `sparse`, `image`). |

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the generated answer and citations. |
| 400 | Bad Request. Collection exists but has no ingested data, invalid API version, or unsupported `x-user-role` value. |
| 401 | Unauthorized. Authentication context missing. |
| 403 | Forbidden. RBAC denied - missing collection role. |
| 404 | Not Found. Collection not found. |

## Chat Completions (Model-Only)

Queries the LLM directly without retrieval. No collection needed, no RBAC check, no `api-version` parameter required.

### Request

```http
POST /edgeai/slm/chat/completions
```

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |
| `x-user-role` | False | String | `"dev"` or `"user"`. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `messages` | True | Array | Chat messages array. |
| `temperature` | False | Float | LLM temperature (0.0–1.0). Default: `0.7`. |
| `top_p` | False | Float | LLM top-p sampling. Default: `0.1`. |

> [!NOTE]
> No `data_sources` field, no `api-version` parameter, no collection validation, no RBAC.

### Example

#### Request

```http
POST /edgeai/slm/chat/completions

Content-Type: application/json
x-user-role: user
```

```json
{
  "messages": [
    {"role": "user", "content": "Explain quantum computing in simple terms"}
  ],
  "temperature": 0.7,
  "top_p": 0.1
}
```

#### Response

**Status code:** 200 OK

Same structure as the RAG response but without citations or search results.

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the generated answer. |

---

## RBAC (Role-Based Access Control)

Every collection is protected when accessed through the external endpoint. When querying through ingress, the system checks the JWT token's `roles` claim:

1. **Collection exists?** → No → `404`
1. **Auth bypassed?** (port-forward / internal Dapr) → Skip RBAC
1. **User has `EdgeRAGDeveloper` role?** → Access granted to all collections
1. **User has a role matching the collection name?** → Access granted
1. **Otherwise** → `403 Access denied`

**Example:** To query collection `finance-docs`, an end user needs JWT roles: `["EdgeRAGEndUser", "finance-docs"]`.

> [!IMPORTANT]
> The `edgeragapp` default collection also requires an `edgeragapp` role for end users. This requirement is a breaking change from previous behavior where any authenticated user could query it.

### Managing roles

Manage roles in Entra ID (Azure AD) as app role assignments on the Agentic RAG application registration. Each collection needs a corresponding app role with a **value** matching the collection name.

## Related content

- [Collections API Reference](collections-api-reference.md) — Create and manage collections
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections before querying
