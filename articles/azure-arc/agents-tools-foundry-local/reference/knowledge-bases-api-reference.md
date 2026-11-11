---
title: Knowledge Bases API Reference - Agents and Tools with Foundry Local
description: REST API reference for knowledge bases in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/05/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Knowledge bases REST API reference

Use this reference to manage knowledge bases in Agents and Tools with Foundry Local. For step-by-step guidance, see [Manage knowledge bases](../knowledge-bases-guide.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

> [!NOTE]
> The earlier `/knowledge-bases` API is retired and returns `410 Gone`. Use `/knowledgebases`.

## Base address and authorization

- **Base address:** `https://<cluster-domain>`.
- **API version:** Required. Add `api-version=2026-08-01-preview` to every request. Other values return `400`.
- **Authorization:** Send a Microsoft Entra ID access token for the extension's app registration in the Authorization header, using the Bearer scheme.

Azure role-based access control (Azure RBAC) checks permissions at the extension resource scope first:

| Required Azure role | Operations | Permission |
|---|---|---|
| **Reader** | List and get knowledge bases | `Microsoft.CognitiveServices/accounts/knowledgeBases/read` |
| **Cognitive Services Contributor** | Create or replace knowledge bases | `Microsoft.CognitiveServices/accounts/knowledgeBases/write` |
| **Cognitive Services Contributor** | Delete knowledge bases | `Microsoft.CognitiveServices/accounts/knowledgeBases/delete` |

The unsupported REST retrieval route also checks `knowledgeBases/write` before returning `501`.

If the Azure role check doesn't allow a request, all operations accept `EdgeRAGDeveloper` as an app-role fallback. `EdgeRAGEndUser` isn't a fallback for these routes. **Contributor** and **Owner** also allow these management operations. In disconnected deployments, use **Reader** for reads and **Contributor** for changes.

The extension's managed identity needs **Reader** at the extension scope to check role assignments. See [Assign Azure roles](../prepare-authentication.md#assign-azure-roles). These management permissions don't allow agent execution through Responses or Conversations. Those APIs require an Azure role and have no app-role fallback.

## Operations

| Operation | Method and path | Success |
|---|---|---|
| List knowledge bases | `GET /knowledgebases` | `200` |
| Create a knowledge base | `POST /knowledgebases` | `201` |
| Get a knowledge base | `GET /knowledgebases('{name}')` | `200` |
| Create or replace a knowledge base | `PUT /knowledgebases('{name}')` | `201` when created, `200` when replaced |
| Delete a knowledge base | `DELETE /knowledgebases('{name}')` | `204` |
| Retrieve from a knowledge base | `POST /knowledgebases('{name}')/retrieve` | Not supported. Returns `501`. Agents retrieve through MCP. |

`GET /knowledgebases` returns `{"value": [...]}` ordered by `name`.

### Request headers

| Header | Operations | Description |
|---|---|---|
| `If-Match` | `PUT`, `DELETE` | Run the request only if the knowledge base still has this ETag. Otherwise returns `412`. |
| `If-None-Match` | `PUT` | Set to `*` to create only. Returns `412` if the knowledge base exists. |

Successful create and replace responses include an `ETag` header. Read the `@odata.etag` property in the knowledge base body after a `GET`; don't depend on `GET` returning an `ETag` header. Send that value in `If-Match` when updating or deleting.

### Error status codes

API handler errors use `{"error": {"code": "...", "message": "...", "param": "..."}}` (`param` is included when a specific field caused the error). Authentication and authorization errors use `{"detail": "..."}`.

| Status code | Cause |
|---|---|
| `400` | The request isn't valid, for example a missing knowledge source or model connection. |
| `401` | Authentication failed. |
| `403` | Neither Azure roles nor the `EdgeRAGDeveloper` app-role fallback allow the operation. |
| `404` | The knowledge base doesn't exist. |
| `409` | The knowledge base is in use by an agent version, or its managed connection name is already taken. |
| `412` | The `If-Match` or `If-None-Match` condition failed. |
| `500` | The service or Kubernetes operation failed. |
| `502` | A Kubernetes store or dependency call failed. |
| `503` | Authentication is unavailable, or the Azure role check is unavailable and no app-role fallback allows the request. |
| `504` | A Kubernetes store or dependency call timed out. |

## Knowledge base object

`POST` and `PUT` requests use the same properties for the request body and the response body. For `PUT`, the body `name` must match the name in the route. The response also includes `@odata.etag`.

| Property | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | 1 to 40 lowercase letters, digits, and hyphens. Starts and ends with a letter or digit. |
| `description` | string | Yes | 1 to 512 characters. Agents see it as the tool description. |
| `knowledgeSources` | array | Yes | 1 to 100 items of `{"name": "<knowledge-source-name>"}`. |
| `models` | array | Yes | Exactly one model. See the next table. |
| `retrievalInstructions` | string | No | Guidance for source selection and searching. Up to 8,192 characters. |
| `answerInstructions` | string | No | Guidance for composing the grounded answer. Up to 8,192 characters. |
| `outputMode` | string | No | `answerSynthesis` (default and only value). Accepted and stored for compatibility; current execution doesn't use this field. |
| `retrievalReasoningEffort` | object | No | `{"kind": "low"}` (default) or `{"kind": "medium"}`. Accepted and stored for compatibility; current execution doesn't use this field. |
| `encryptionKey` | object | No | Not supported. Returns `400`. |
| `corsOptions` | object | No | Not supported. Returns `400`. |

### Model properties

| Property | Type | Required | Description |
|---|---|---|---|
| `kind` | string | Yes | `azureOpenAI`. |
| `azureOpenAIParameters.resourceUri` | string | Yes | `connection://<model-connection-name>`. |
| `azureOpenAIParameters.deploymentId` | string | Yes | The model name in the model connection. |
| `azureOpenAIParameters.modelName` | string | No | If set, must equal `deploymentId`. |

The model connection must exist, be ready, and contain the model identified by `deploymentId`.

## Managed connection

When you create a knowledge base, the service creates an MCP connection named `kb-<name>`. Add it to an agent version as an `mcp` tool with `project_connection_id` set to `kb-<name>`. The connection offers one tool, `knowledge_base_retrieve`. When you delete the knowledge base, the service deletes the connection.

## Related content

- [Agents REST API reference](agents-api-reference.md)
- [Responses and Conversations REST API reference](responses-api-reference.md)
- [Manage knowledge bases](../knowledge-bases-guide.md)
- [Knowledge sources REST API reference](knowledge-sources-api-reference.md)
- [KnowledgeBase CRD reference](knowledgebase-crd-reference.md)
