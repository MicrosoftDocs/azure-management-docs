---
title: Inference REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for querying collections using RAG or querying the language model directly.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/05/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
---

# Inference REST API reference

Query collections by using retrieval-augmented generation (RAG) or query the language model directly. The inference API searches a collection's vector data, retrieves relevant chunks, and generates a response grounded in the retrieved content.

> [!IMPORTANT]
> The inference API is the classic chat path, and it's deprecated. Its chat completions and feedback routes accept external requests only from the developer portal in the browser. Other external callers get `403 Forbidden` with the message `Classic chat and evaluation are available only in the Developer Portal.`, even with a valid token. For new applications, use agents and the [Responses API](responses-api-reference.md). This API uses the `byom.*` settings and the `byom-api-key` Secret. Agents don't use them.

## API information

The RAG endpoint requires the `api-version=2024-10-01-preview` query parameter. The model-only endpoint doesn't require this parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Inference API (`inferencing-flow`) |
| **Port** | 3001 |
| **Dapr App ID** | `inferencing-flow` |

## Access methods

You can reach the Inference API through three access methods. The access method determines whether authentication and RBAC are enforced.

### External access (via ingress)

```
https://<cluster-domain>/chat/...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header, which triggers token validation, the Azure role check, and any supported app-role fallback.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/inferencingflow-deployment 3001:3001 -n arc-rag
```

Then call `http://localhost:3001/chat/...`. Without the ingress external-request marker, this classic service skips authentication and collection checks. Limit direct cluster access to trusted administrators.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: inferencing-flow" http://localhost:3500/chat/...
```

Without the ingress external-request marker, this classic service skips authentication and collection checks. This behavior doesn't apply to protected Agent Service APIs.

---

## Authentication

Require authentication only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

Azure role-based access control (Azure RBAC) checks permissions at the extension resource scope first. For external inference calls, assign **Cognitive Services OpenAI User**. The routes use the data action `Microsoft.CognitiveServices/accounts/OpenAI/deployments/chat/completions/action`. In disconnected deployments, this data permission maps to a control-plane permission; use **Contributor**.

Callers allowed by an Azure role can access every collection without a per-collection filter. The extension's managed identity needs **Reader** at the extension scope to check Azure roles. See [Assign Azure roles](../prepare-authentication.md#assign-azure-roles).

If the Azure role check doesn't allow a request, these classic inference routes accept the following app-role fallback:

| App role | Description |
|---|---|
| `EdgeRAGDeveloper` | Fallback query access to all collections. |
| `EdgeRAGEndUser` | Access only to collections where the user has a matching app role (for example, role `finance-docs` grants access to collection `finance-docs`). |

This fallback doesn't apply to the Agent Service Responses or Conversations APIs. Those APIs require an Azure role.

### Getting a token via Azure CLI

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

## Chat completions (RAG)

Query a collection by using Retrieval-Augmented Generation. Search the collection's vector data, retrieve relevant chunks, and generate an LLM response that's grounded in the retrieved content.

### Request

```http
POST /chat/completions?api-version=2024-10-01-preview
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

Collection access depends on the Azure role decision or, for app-role fallback, the token's app roles. The endpoint doesn't use an `x-user-role` header.

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `messages` | True | Array | Chat messages. Include `system` (optional), prior `user`/`assistant` turns for history, and final `user` message with the question. |
| `temperature` | False | Float | LLM temperature (0.0–1.0). Default: `0.0`. |
| `top_p` | False | Float | LLM top-p sampling. Default: `1.0`. |
| `isMarkdown` | False | Boolean | If `true`, LLM response is converted to markdown. Default: `false`. |
| `data_sources[0].type` | False | String | Data source type. Use `"milvus"`. |
| `data_sources[0].parameters.endpoint` | False | String | Milvus endpoint. Use `""` (empty string). |
| `data_sources[0].parameters.index_name` | False | String | Collection to query. Aliases `collectionName` and `collection_name` are also accepted. Default: `"edgeragapp"`. |
| `data_sources[0].parameters.text_strictness` | False | Integer | Text relevance threshold (0–5). Higher = stricter filtering. Default: `3`. |
| `data_sources[0].parameters.image_strictness` | False | Integer | Image relevance threshold (0–5). Only used with multimodal search. Default: `1`. |
| `data_sources[0].parameters.top_n_documents` | False | Integer | Max number of chunks to retrieve. Default: `2`. |
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
POST /chat/completions?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
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
| 400 | Bad Request. The collection exists but has no ingested data, the API version is invalid, or the request body is unsupported. |
| 401 | Unauthorized. Authentication context missing. |
| 403 | Forbidden. The request didn't come from the developer portal, neither Azure roles nor app-role fallback allow the request, or the fallback caller lacks a collection role. |
| 503 | Authentication or authorization service unavailable, with no successful fallback. |
| 404 | Not Found. Collection not found. |

## Chat completions (model only)

Queries the LLM directly without retrieval. No collection or `api-version` parameter is needed. External calls still require authentication and an Azure role or supported app-role fallback.

### Request

```http
POST /slm/chat/completions
```

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `messages` | True | Array | Chat messages array. |
| `temperature` | False | Float | LLM temperature (0.0–1.0). Default: `0.0`. |
| `top_p` | False | Float | LLM top-p sampling. Default: `1.0`. |

> [!NOTE]
> No `data_sources` field, `api-version` parameter, or collection check is needed. External API authorization still applies.

### Example

#### Request

```http
POST /slm/chat/completions

Content-Type: application/json
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
| 400 | Bad Request. The request body is invalid or unsupported. |
| 401 | Unauthorized. Authentication context missing. |
| 403 | Forbidden. The request didn't come from the developer portal, or neither Azure roles nor app-role fallback allow the request. |
| 503 | Authentication, authorization, or model service unavailable, with no successful fallback. |

---

## RBAC (Role-Based Access Control)

Azure RBAC authorizes the API at the extension scope. When Azure roles allow the request, the caller can query every collection. Azure RBAC doesn't filter individual collections.

*App-role collection access* applies only when a classic route uses app-role fallback:

1. `EdgeRAGDeveloper` allows access to all collections.
1. `EdgeRAGEndUser` also needs an app role that matches the collection name.
1. Without matching collection access, the query returns `403`.

**Example:** With app-role fallback, querying `finance-docs` needs JWT roles `["EdgeRAGEndUser", "finance-docs"]`.

> [!IMPORTANT]
> With app-role fallback, the `edgeragapp` default collection also requires an `edgeragapp` role for end users. Azure-role-authorized callers don't need collection app roles.

### Managing roles

For Azure role assignments, see [Assign Azure roles](../prepare-authentication.md#assign-azure-roles). For app-role fallback, manage app roles on the **Agents and Tools with Foundry Local** app registration. Each collection role's **Value** must match the collection name.

## Related content

- [Collections API Reference](collections-api-reference.md) — Create and manage collections
- [Ingestion API Reference](ingestion-api-reference.md) — Ingest documents into collections before querying
