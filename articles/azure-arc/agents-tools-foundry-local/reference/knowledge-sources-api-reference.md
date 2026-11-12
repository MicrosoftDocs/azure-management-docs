---
title: Knowledge Sources REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for managing knowledge sources by using the Knowledge Sources API in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
---

# Knowledge sources REST API reference

Use this reference to manage knowledge sources in Agents and Tools with Foundry Local. A knowledge source registers an indexed collection, an external MCP server, or a SQL database. Knowledge bases refer to knowledge sources by name. For step-by-step guidance, see [Configure a knowledge source](../knowledge-sources-guide.md).

[!INCLUDE [sql-knowledge-source-preview-notice](../includes/sql-knowledge-source-preview-notice.md)]

## Base address and authorization

- **Base address:** `https://<cluster-domain>/knowledgesources`.
- **Authorization:** Send a Microsoft Entra ID access token for the extension's app registration in the Authorization header, using the Bearer scheme.

Azure role-based access control (Azure RBAC) checks permissions at the extension resource scope first:

| Required Azure role | Operations | Permission |
|---|---|---|
| **Reader** | List and get sources; get exposed SQL schema | `Microsoft.CognitiveServices/accounts/knowledgeSources/read` |
| **Cognitive Services Contributor** | Create or update sources; preview a SQL schema; replace exposed schema; rotate SQL credentials | `Microsoft.CognitiveServices/accounts/knowledgeSources/write` |
| **Cognitive Services Contributor** | Delete one or several sources | `Microsoft.CognitiveServices/accounts/knowledgeSources/delete` |

If the Azure role check doesn't allow a request, all operations accept `EdgeRAGDeveloper` as an app-role fallback. `EdgeRAGEndUser` isn't a fallback for these routes. **Contributor** and **Owner** also allow these management operations. In disconnected deployments, use **Reader** for reads and **Contributor** for changes, including schema previews.

The extension's managed identity needs **Reader** at the extension scope to check role assignments. See [Assign Azure roles](../prepare-authentication.md#assign-azure-roles).

> [!IMPORTANT]
> The Knowledge Sources API path is `/knowledgesources`. Earlier builds used `/knowledgesources/v2` or `/edgeai/knowledgesources/v2`. Those URLs no longer work, and there's no redirect. For example, `POST /knowledgesources/v2` returns `405 Method Not Allowed`, and `GET /knowledgesources/v2` reads a source named `v2`. Update your clients to `/knowledgesources`, and don't send old requests again. `/edgeai/knowledgesources` still works as an alias of `/knowledgesources`.

These permissions manage knowledge sources. Running agents through Responses or Conversations requires an Azure role and has no app-role fallback. See [Permission checks](../agent-service-security.md#permission-checks).

## Operations

| Operation | Method and path | Success |
|---|---|---|
| Create a knowledge source | `POST /knowledgesources` | `201` for MCP sources; `202` for SQL |
| List knowledge sources | `GET /knowledgesources` | `200` |
| Get a knowledge source | `GET /knowledgesources/{name}` | `200` |
| Update a knowledge source | `PATCH /knowledgesources/{name}` | `200` |
| Delete a knowledge source | `DELETE /knowledgesources/{name}` | `204` |
| Delete several knowledge sources | `POST /knowledgesources/batch-delete` | `200` |
| Preview a SQL schema | `POST /knowledgesources/schema-preview` | `200` |
| Get exposed SQL schema | `GET /knowledgesources/{name}/schema` | `200` |
| Replace exposed SQL schema | `PUT /knowledgesources/{name}/schema` | `202` |
| Rotate SQL credential | `PUT /knowledgesources/{name}/credential` | `202` |

## Create request body

| Property | Type | Required | Description |
|---|---|---|---|
| `name` | string | Yes | The name and ID. Use 1 to 40 lowercase letters, digits, and hyphens. Start and end with a letter or digit. Can't look like a GUID. Can't change later. |
| `kind` | string | Yes | `indexed_sources_mcp`, `remote_mcp`, or `sql`. Can't change later. |
| `auth_type` | string | Yes | `microsoft_entra_id` forwards the caller's token only to a trusted MCP destination. `unauthenticated` sends no credentials (also use `unauthenticated` for `sql`). |
| `description` | string | No | Up to 500 characters. |
| `indexed_sources_parameters` | object | For `indexed_sources_mcp` | See the next table. |
| `remote_mcp_parameters` | object | For `remote_mcp` | See the next table. |
| `sql_parameters` | object | For `sql` | See [Connect a SQL database](../connect-sql-database.md#sql-parameters). |

Send only the parameters object that matches `kind`.

### indexed_sources_parameters

| Property | Type | Required | Description |
|---|---|---|---|
| `server_url` | string | Yes | The trusted in-cluster address of Indexed Sources MCP: `http://localhost:3500/v1.0/invoke/indexed-sources-mcp-server/method/mcp`. |
| `indexed_source_ref` | string | Yes | The collection name, up to 256 characters. |
| `server_label` | string | No | Up to 64 characters. Default `indexed-sources-mcp-server`. |

### remote_mcp_parameters

| Property | Type | Required | Description |
|---|---|---|---|
| `server_url` | string | Yes | The HTTPS MCP server address, up to 2,048 characters. |
| `server_label` | string | No | Up to 64 characters. |

When you create a knowledge source or change its connection settings, the service connects to the MCP server to check it. If the check fails, the service returns `400` and saves nothing.

With `microsoft_entra_id`, the deployment must trust the destination before the service forwards the caller's token. Use `unauthenticated` when delegated authentication isn't required.

## Knowledge source object

| Property | Type | Description |
|---|---|---|
| `name` | string | The name and ID. |
| `kind` | string | The kind. |
| `description` | string | The description. |
| `auth_type` | string | The authentication type. |
| `indexed_sources_parameters` | object | The settings for `indexed_sources_mcp`, or `null`. |
| `remote_mcp_parameters` | object | The settings for `remote_mcp`, or `null`. |
| `uid` | string | Kubernetes resource UID, or `null`. |
| `resource_version` | string | Opaque resource version, or `null`. |
| `generation` | integer | Current desired generation, or `null`. |
| `observed_generation` | integer | Most recently completed generation, or `null`. |
| `provisioning_state` | string | `Provisioning`, `Succeeded`, or `Failed`. |
| `provisioning_message` | string | Reason for the current provisioning state, or `null`. |
| `created_by` | string | Who created it. |
| `created_at` | string | When it was created, in ISO 8601 format. |

SQL connection strings and `sql_parameters` aren't returned. Use the SQL schema operation to read the exposed entities.

## Readiness

`indexed_sources_mcp` and `remote_mcp` sources return `201 Created` on create and `200 OK` on `PATCH`, with `provisioning_state` set to `Succeeded`. Creating a `sql` source returns `202 Accepted`. Updating a `sql` source with `PATCH` returns `200 OK` with `provisioning_state` set to `Provisioning`, while `PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`. Whenever `provisioning_state` is `Provisioning` or the response is `202`, poll `GET /knowledgesources/{name}` until `provisioning_state` is `Succeeded` or `Failed`.

`generation` is the desired configuration. `observed_generation` is the generation most recently completed by the status writer or operator. A pending generation isn't reported as ready because an earlier generation succeeded.

## List query parameters

| Name | Type | Description |
|---|---|---|
| `kind` | string | Return only `indexed_sources_mcp`, `remote_mcp`, or `sql`. |
| `limit` | integer | 1 to 100. Default 10. |
| `offset` | integer | Number of items to skip. Default 0. |

The response is `{"items": [...], "total": <number>, "limit": <number>, "offset": <number>}`.

## Update request body

Send only the fields that you want to change: `description`, `auth_type`, or the parameters object that matches the kind. A parameters object replaces the old one completely. You can't change `name` or `kind`. Omit a field to preserve it. Send `null` only for `description`; explicit `null` is rejected for `auth_type` and parameter blocks.

`PATCH /knowledgesources/{name}` behavior depends on the source kind:

- `indexed_sources_mcp` and `remote_mcp` updates revalidate the MCP server synchronously and return `200 OK` with `provisioning_state: "Succeeded"`.
- `sql` `PATCH` updates replace the exposed schema (while retaining the stored credential; use `PUT /knowledgesources/{name}/credential` to rotate it) and return `200 OK` with `provisioning_state: "Provisioning"`. (`PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`.)
- Description-only or no-op updates return `200 OK`.

## Batch delete

Request body:

```json
{ "names": ["product-manuals", "benefits-docs"] }
```

You can send up to 100 names. The response has one result for each name, in the same order:

```json
{
  "results": [
    { "name": "product-manuals", "status": "deleted" },
    { "name": "benefits-docs", "status": "not_found" }
  ],
  "deleted_count": 1
}
```

`status` is `deleted`, `not_found`, or `error`. For `error`, `message` explains why.

## Related content

- [Configure a knowledge source](../knowledge-sources-guide.md)
- [Knowledge Bases REST API reference](knowledge-bases-api-reference.md)
- [MCP Server API reference](mcp-server-api-reference.md)
- [Connect a SQL database as a knowledge source](../connect-sql-database.md)
