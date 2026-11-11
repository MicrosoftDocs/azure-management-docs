---
title: Configure a Knowledge Source in Agents and Tools with Foundry Local
description: Learn how to create, update, and delete knowledge sources in Agents and Tools with Foundry Local so that knowledge bases can search your content.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer, I want to register my indexed collections and MCP servers as knowledge sources so that I can group them into knowledge bases for my agents.
---

# Configure a knowledge source in Agents and Tools with Foundry Local

A *knowledge source* registers one place that holds knowledge. It can be a collection that Agentic Retrieval indexed, an external Model Context Protocol (MCP) server, or an approved read-only SQL database slice. Agents don't use knowledge sources directly. Instead, you group knowledge sources into a [knowledge base](knowledge-bases-guide.md), and then add the knowledge base to an agent as a tool.

This article shows you how to create, list, update, and delete knowledge sources by using the Knowledge Sources API.

[!INCLUDE [sql-knowledge-source-preview-notice](includes/sql-knowledge-source-preview-notice.md)]

## Prerequisites

- An Agents and Tools with Foundry Local deployment.
- For indexed sources: a collection with ingested documents. See [Data collections](collections-overview.md) and [Add data source](add-data-source.md).
- For remote sources: an MCP server that the cluster can reach.
- For SQL sources: a supported SQL Server, PostgreSQL, or MySQL database. See [Connect a SQL database](connect-sql-database.md).
- A Microsoft Entra ID identity with **Cognitive Services Contributor** at the extension resource scope to create, change, or delete knowledge sources. **Reader** is enough to list and get sources and read exposed SQL schemas. In disconnected deployments, use **Contributor** for changes. All these operations support `EdgeRAGDeveloper` as an app-role fallback.

Azure roles are checked first. Previewing a SQL schema also needs **Cognitive Services Contributor**, even though it doesn't save a source. The extension's managed identity needs **Reader** at the extension scope to check caller roles. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles) and [Security and access](agent-service-security.md#permission-checks).

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

## Knowledge source kinds

The following table describes the kinds of knowledge sources that this article covers.

| Kind | Parameters field | Use it for |
|---|---|---|
| `indexed_sources_mcp` | `indexed_sources_parameters` | A collection that Agentic Retrieval indexed on this cluster. The built-in MCP server searches it. |
| `remote_mcp` | `remote_mcp_parameters` | An external MCP server. |
| `sql` | `sql_parameters` | An approved read-only set of SQL tables and columns. |

Microsoft 365 Local adds two more kinds, `sharepoint` and `exchange`, for on-premises SharePoint Server and Exchange Server. These kinds are in limited preview for approved customers. Microsoft 365 Local is supported only with Agentic Retrieval: add these sources to a knowledge base, not to an agent as direct tools. For setup, see [Connect Microsoft 365 Local](connect-microsoft-365-local.md).

## Create an indexed source

When you set `indexedSourcesEnabled=true`, the extension automatically registers a default knowledge source named `knowledge-agent`. It uses the built-in MCP server and points to the `edgeragapp` collection. You can use this source in a knowledge base, or create a source for another collection.

To make an indexed collection available to knowledge bases, send a `POST` request to `/knowledgesources`. Set `indexed_source_ref` to the collection name.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/knowledgesources" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "benefits-docs",
    "kind": "indexed_sources_mcp",
    "auth_type": "microsoft_entra_id",
    "description": "Benefits policy documents",
    "indexed_sources_parameters": {
      "server_url": "http://localhost:3500/v1.0/invoke/indexed-sources-mcp-server/method/mcp",
      "indexed_source_ref": "benefits"
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/knowledgesources", json={
    "name": "benefits-docs",
    "kind": "indexed_sources_mcp",
    "auth_type": "microsoft_entra_id",
    "description": "Benefits policy documents",
    "indexed_sources_parameters": {
        "server_url": "http://localhost:3500/v1.0/invoke/indexed-sources-mcp-server/method/mcp",
        "indexed_source_ref": "benefits",
    },
})
r.raise_for_status()
print(r.json()["provisioning_state"])  # Succeeded
```

---

The `server_url` in this example is the in-cluster address of the built-in MCP server. Use it as shown.

If the request succeeds, the service returns `201 Created`, and `provisioning_state` is `Succeeded`:

```json
{
  "name": "benefits-docs",
  "kind": "indexed_sources_mcp",
  "description": "Benefits policy documents",
  "auth_type": "microsoft_entra_id",
  "indexed_sources_parameters": {
    "server_url": "http://localhost:3500/v1.0/invoke/indexed-sources-mcp-server/method/mcp",
    "indexed_source_ref": "benefits",
    "server_label": "indexed-sources-mcp-server"
  },
  "provisioning_state": "Succeeded",
  "created_at": "2026-10-21T09:00:00Z"
}
```

When you set `auth_type` to `microsoft_entra_id`, the search runs with the end user's identity. Users only get results from collections that they have access to. For more information, see [Collections overview](collections-overview.md).

## Create a remote MCP source

To register an external MCP server as a knowledge source, use `remote_mcp`:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/knowledgesources" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "product-manuals",
    "kind": "remote_mcp",
    "auth_type": "unauthenticated",
    "description": "Product manuals search server",
    "remote_mcp_parameters": {
      "server_url": "https://manuals.contoso.com/mcp",
      "server_label": "manuals"
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/knowledgesources", json={
    "name": "product-manuals",
    "kind": "remote_mcp",
    "auth_type": "unauthenticated",
    "description": "Product manuals search server",
    "remote_mcp_parameters": {
        "server_url": "https://manuals.contoso.com/mcp",
        "server_label": "manuals",
    },
})
r.raise_for_status()
```

---

Before the service saves the knowledge source, it connects to the MCP server to check that it works. If the check fails, the service returns `400 Bad Request` and saves nothing.

Authenticated external MCP destinations must use HTTPS and be trusted by the deployment before the service forwards a caller token. Use `unauthenticated` when the server doesn't require delegated authentication.

> [!TIP]
> To let an agent call all the tools of an external MCP server, you don't need a knowledge source. Add the server to the agent as an MCP tool instead. See [Add tools to an agent](add-agent-tools.md). Use a knowledge source when you want a knowledge base to combine the server with other sources.

## Check readiness

Get the source by name:

```bash
curl "$BASE/knowledgesources/benefits-docs" \
  -H "Authorization: ******"
```

MCP source creation normally returns `Succeeded`. SQL source creation returns `202 Accepted` and can report `Provisioning` while the operator creates the serving workload. Poll a source that reports `Provisioning` until `provisioning_state` is `Succeeded`. If the state is `Failed`, inspect the custom resource's `status.message` and correct the cause.

## Request fields

The following table describes the fields that you send when you create a knowledge source.

| Field | Required | Description |
|---|---|---|
| `name` | Yes | The name and ID of the knowledge source. Use lowercase letters, digits, and hyphens, and start and end with a letter or digit. You can't change it later. Knowledge bases refer to the knowledge source by this name. |
| `kind` | Yes | `indexed_sources_mcp`, `remote_mcp`, or `sql`. You can't change it later. |
| `auth_type` | Yes | `microsoft_entra_id` sends the end user's token only to a trusted MCP destination. `unauthenticated` sends no credentials. |
| `description` | No | A short description of the content. |
| `indexed_sources_parameters.server_url` | For `indexed_sources_mcp` | The address of the built-in MCP server. |
| `indexed_sources_parameters.indexed_source_ref` | For `indexed_sources_mcp` | The name of the collection. |
| `indexed_sources_parameters.server_label` | No | A label for the server. Default `indexed-sources-mcp-server`. |
| `remote_mcp_parameters.server_url` | For `remote_mcp` | The address of the MCP server. |
| `remote_mcp_parameters.server_label` | No | A label for the server. |
| `sql_parameters` | For `sql` | SQL connection, schema, and credential configuration. See [Connect a SQL database](connect-sql-database.md). |

> [!WARNING]
> With `microsoft_entra_id`, the end user's token goes to the MCP server. Register only MCP servers that you operate or trust.

## List and get knowledge sources

To list knowledge sources, use `limit` (1 to 100, default 10) and `offset` for paging. You can also filter by `kind`.

# [curl](#tab/curl)

```bash
curl "$BASE/knowledgesources?kind=indexed_sources_mcp&limit=20" \
  -H "Authorization: Bearer $TOKEN"

curl "$BASE/knowledgesources/benefits-docs" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
page = api.get(f"{BASE}/knowledgesources",
               params={"kind": "indexed_sources_mcp", "limit": 20}).json()
for source in page["items"]:
    print(source["name"], source["provisioning_state"])

source = api.get(f"{BASE}/knowledgesources/benefits-docs").json()
```

---

The list response has `items`, `total`, `limit`, and `offset`.

## Update a knowledge source

To change a knowledge source, send a `PATCH` request with only the fields that you want to change. You can change `description`, `auth_type`, and the parameters object that matches the kind. When you send a parameters object, it replaces the old one completely.

# [curl](#tab/curl)

```bash
curl -X PATCH "$BASE/knowledgesources/product-manuals" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{ "description": "Product manuals, 2026 edition" }'
```

# [Python](#tab/python)

```python
api.patch(f"{BASE}/knowledgesources/product-manuals",
          json={"description": "Product manuals, 2026 edition"}).raise_for_status()
```

---

For `indexed_sources_mcp`, `remote_mcp`, and `sql` sources, `PATCH` returns `200 OK`. For MCP sources, connection changes are validated synchronously before returning `200 OK`. For SQL `sql_parameters` changes, the `200 OK` response reports `provisioning_state` as `Provisioning` while reconciliation continues, whereas `PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`.

## Delete knowledge sources

Before you delete a knowledge source, remove it from every knowledge base that uses it. See [Manage knowledge bases](knowledge-bases-guide.md).

# [curl](#tab/curl)

```bash
# Delete one knowledge source
curl -X DELETE "$BASE/knowledgesources/product-manuals" \
  -H "Authorization: Bearer $TOKEN"

# Delete several knowledge sources
curl -X POST "$BASE/knowledgesources/batch-delete" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{ "names": ["product-manuals", "benefits-docs"] }'
```

# [Python](#tab/python)

```python
api.delete(f"{BASE}/knowledgesources/product-manuals").raise_for_status()

result = api.post(f"{BASE}/knowledgesources/batch-delete",
                  json={"names": ["product-manuals", "benefits-docs"]}).json()
for item in result["results"]:
    print(item["name"], item["status"])
```

---

The batch response lists the result for each name: `deleted`, `not_found`, or `error`.

> [!NOTE]
> The Knowledge Sources API path is `/knowledgesources`. URLs from earlier builds that end in `/knowledgesources/v2` no longer work. For details, see the [Knowledge Sources REST API reference](reference/knowledge-sources-api-reference.md#base-address-and-authorization).

## Next step

> [!div class="nextstepaction"]
> [Manage knowledge bases](knowledge-bases-guide.md)

## Related content

- [Agent Service overview](agentic-overview.md)
- [MCP server in Agents and Tools with Foundry Local](mcp-server-overview.md)
- [Connect a SQL database as a knowledge source](connect-sql-database.md)
- [Add tools to an agent](add-agent-tools.md)
- [Knowledge Sources REST API reference](reference/knowledge-sources-api-reference.md)
