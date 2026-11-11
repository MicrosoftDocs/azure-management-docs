---
title: Add Tools to an Agent in Agents and Tools with Foundry Local
description: Learn how to give an agent in Agents and Tools with Foundry Local tools such as MCP servers, Microsoft Foundry toolboxes, and knowledge bases.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer, I want to connect my agent to MCP servers, toolboxes, and knowledge bases so that it can look up information and act for my users.
---

# Add tools to an agent in Agents and Tools with Foundry Local

An agent becomes useful when it can look things up and take action. In Agents and Tools with Foundry Local, you give an agent *tools*. The agent decides when to call a tool, reads the result, and uses it in its answer.

All tools use the Model Context Protocol (MCP). MCP is an open standard that lets an AI agent discover and call tools on a server. In this article, you learn how to:

- Register an MCP server as a *connection*, and add it to an agent.
- Create the connection inline, as part of the agent version.
- Add a Microsoft Foundry toolbox to an agent.
- Add a knowledge base to an agent.
- Limit which tools the agent can call.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

> [!WARNING]
> Connect only to MCP servers that you trust. Treat tool descriptions and results as untrusted input. Limit the tools an agent can call, review actions that change data, and monitor tool calls. See [Authentication and security](mcp-server-overview.md#authentication-and-security).

## Prerequisites

- An agent and a model connection. See [Create and manage agents](create-manage-agents.md).
- A Microsoft Entra ID identity with **Cognitive Services Contributor** at the extension resource scope to change connections and agent versions. **Reader** is enough for read-only access. In disconnected deployments, use **Contributor** for changes. These management APIs check Azure roles first and use `EdgeRAGDeveloper` as a fallback.
- For MCP tools: an MCP server that the cluster can reach over HTTPS.
- For knowledge base tools: a knowledge base. See [Manage knowledge bases](knowledge-bases-guide.md).

To run an agent after you add tools, also assign **Cognitive Services OpenAI User** in connected deployments, or **Contributor** in disconnected deployments. Responses and Conversations have no app-role fallback.

The extension's managed identity needs **Reader** at the extension scope to check caller roles. For setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles). For separate model and tool permissions, see [Security and access](agent-service-security.md#permission-checks).

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

## Tool types

The following table lists the tools that you can add to an agent version. Each one is an `mcp` tool in the agent version. They differ in where the MCP server runs and how you connect to it.

| Tool | What the agent can do | How you connect it |
|---|---|---|
| **MCP server** | Call the tools that an MCP server offers, such as a search service or a line-of-business API. | An MCP connection that you create first, or inline in the agent version. |
| **Microsoft Foundry toolbox** | Call the tools in a toolbox that you set up in a Microsoft Foundry project. | An MCP connection that points to the toolbox address. Needs network access to Azure. |
| **Knowledge base** | Search your knowledge sources, and get grounded content with citations. | The connection `kb-<knowledge-base-name>`, which the platform creates with each knowledge base. |

An agent version can have up to 32 tools. Each tool needs a unique `server_label`.

## Register an MCP server as a connection

An *MCP connection* stores the address of an MCP server and how to sign in to it. Create it once, and then add it to as many agents as you need. Send a `PUT` request to `/connections/<connection-name>`.

# [curl](#tab/curl)

```bash
curl -X PUT "$BASE/connections/inventory-mcp" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "type": "RemoteTool_Preview",
    "target": "https://inventory.contoso.com/mcp",
    "authType": "ApiKey",
    "apiKey": "<your-api-key>",
    "apiKeyHeaderName": "x-api-key",
    "apiKeyValueFormat": "Raw"
  }'
```

# [Python](#tab/python)

```python
r = api.put(f"{BASE}/connections/inventory-mcp", json={
    "type": "RemoteTool_Preview",
    "target": "https://inventory.contoso.com/mcp",
    "authType": "ApiKey",
    "apiKey": "<your-api-key>",
    "apiKeyHeaderName": "x-api-key",
    "apiKeyValueFormat": "Raw",
})
r.raise_for_status()
```

---

The following table describes the fields of an MCP connection.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | `RemoteTool_Preview`. |
| `target` | Yes | The HTTPS address of the MCP server. |
| `authType` | Yes | How the platform signs in to the MCP server. See the next table. |
| `tokenScope` | For `AAD` | The Microsoft Entra ID scope to request, ending with `/.default`. |
| `apiKey` | For `ApiKey` | The API key. The platform stores it as a secret and never returns it. |
| `apiKeyHeaderName` | For `ApiKey` | The HTTP header that carries the key, for example `x-api-key` or `Authorization`. |
| `apiKeyValueFormat` | For `ApiKey` | `Raw` sends the key as is. `Bearer` sends `Bearer <key>`. |
| `metadata` | No | Up to 16 key-value pairs for your own use. |

Choose the authentication type that your MCP server expects:

| `authType` | What happens when the agent calls the server |
|---|---|
| `ApiKey` | The platform sends the stored API key in the header that you set in `apiKeyHeaderName`. |
| `AAD` | The platform gets a Microsoft Entra ID token for `tokenScope` with the extension's managed identity, and sends it to the server. The end user's token isn't sent. A cluster administrator must approve the server address first. See [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity). |
| `None` | The platform sends no credentials. Use it only for servers inside a trusted network. |

To check a connection, send `GET /connections/<connection-name>`. The response never includes the API key. To change a connection, send `PUT` again with the full settings, or `PATCH` with only the fields you want to change.

## Add an MCP server to an agent

Create a new agent version and add an `mcp` tool. Set `project_connection_id` to the name of the MCP connection.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "Use the inventory tools to check stock before you answer.",
      "tools": [
        {
          "type": "mcp",
          "server_label": "inventory",
          "server_description": "Looks up product stock levels and locations.",
          "project_connection_id": "inventory-mcp",
          "allowed_tools": { "tool_names": ["get_stock", "find_location"] }
        }
      ]
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/hr-helper/versions", json={
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "Use the inventory tools to check stock before you answer.",
        "tools": [{
            "type": "mcp",
            "server_label": "inventory",
            "server_description": "Looks up product stock levels and locations.",
            "project_connection_id": "inventory-mcp",
            "allowed_tools": {"tool_names": ["get_stock", "find_location"]},
        }],
    },
})
r.raise_for_status()
print(r.json()["version"])
```

---

The following table describes the fields of an `mcp` tool.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | `mcp`. |
| `server_label` | Yes | A short, unique name for this tool in the agent, up to 64 characters. It appears in tool calls and citations. |
| `server_description` | No | Tells the model what the server is for, up to 512 characters. A clear description helps the agent pick the right tool. |
| `project_connection_id` | One of these two | The name of an existing MCP connection. |
| `connection` | One of these two | The settings of a new MCP connection. The platform creates the connection for you. See the next section. |
| `allowed_tools` | No | `{"tool_names": [...]}` limits the agent to the listed tools on the server. If you leave it out, the agent can call every tool that the server offers. |

> [!TIP]
> Use `allowed_tools` to give the agent only the tools it needs. Fewer tools make it easier for the model to choose correctly, and they reduce risk.

## Create the connection inline

If you don't want to create the connection first, put the connection settings in the tool's `connection` field. The platform creates the MCP connection when it saves the version, and replaces `connection` with `project_connection_id` in the stored version.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "Use the weather tools to answer.",
      "tools": [
        {
          "type": "mcp",
          "server_label": "weather",
          "server_description": "Current weather and forecasts.",
          "connection": {
            "type": "RemoteTool_Preview",
            "target": "https://weather.contoso.com/mcp",
            "authType": "None"
          }
        }
      ]
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/hr-helper/versions", json={
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "Use the weather tools to answer.",
        "tools": [{
            "type": "mcp",
            "server_label": "weather",
            "server_description": "Current weather and forecasts.",
            "connection": {
                "type": "RemoteTool_Preview",
                "target": "https://weather.contoso.com/mcp",
                "authType": "None",
            },
        }],
    },
})
r.raise_for_status()
print(r.json()["definition"]["tools"][0]["project_connection_id"])
```

---

The created connection has a name that starts with `mcp-inline-`. Keep the following points in mind:

- When you create a later version with the same connection settings, the platform reuses the same connection.
- The platform doesn't delete inline connections when you delete the agent. To remove one, delete it with `DELETE /connections/<name>` after no agent version uses it.
- Use separate connections, created with `PUT /connections`, when several agents share the same server. They're easier to find and update.

## Add a Microsoft Foundry toolbox to an agent

A *toolbox* is a set of tools that you build and manage in a [Microsoft Foundry](/azure/foundry/what-is-foundry) project. Each toolbox has an MCP address, so an agent on your cluster can call its tools. Toolboxes run in Azure, so the cluster needs network access to Azure.

Before you start, ensure that:

- The extension's managed identity has access to the toolbox in your Foundry project. Without access, creating the agent version fails with `400 Bad Request` and the message `missing RBAC permissions for toolbox`.
- A cluster administrator approved the toolbox address. Approve the toolbox prefix `https://<account>.services.ai.azure.com/api/projects/<project>/toolboxes/<toolbox>/`, so all versions of the toolbox are allowed. See [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity).

Create a connection to the toolbox. Toolbox connections must use `authType` `AAD` with the scope `https://ai.azure.com/.default`.

# [curl](#tab/curl)

```bash
curl -X PUT "$BASE/connections/support-toolbox" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "type": "RemoteTool_Preview",
    "target": "https://<account>.services.ai.azure.com/api/projects/<project>/toolboxes/<toolbox>/mcp?api-version=v1",
    "authType": "AAD",
    "tokenScope": "https://ai.azure.com/.default"
  }'
```

# [Python](#tab/python)

```python
r = api.put(f"{BASE}/connections/support-toolbox", json={
    "type": "RemoteTool_Preview",
    "target": "https://<account>.services.ai.azure.com/api/projects/<project>/toolboxes/<toolbox>/mcp?api-version=v1",
    "authType": "AAD",
    "tokenScope": "https://ai.azure.com/.default",
})
r.raise_for_status()
```

---

The toolbox address uses one of these forms. The query must be exactly `api-version=v1`.

| Form | Use it to |
|---|---|
| `.../toolboxes/<toolbox>/mcp?api-version=v1` | Always use the toolbox's current default version. |
| `.../toolboxes/<toolbox>/versions/<version>/mcp?api-version=v1` | Pin the agent to one toolbox version. |

Then add the connection to an agent version as an `mcp` tool with `project_connection_id` set to `support-toolbox`, like any other MCP connection.

When the agent runs, the platform checks which tools the toolbox offers. It hides tool types that it can't run, and the agent uses the rest. If the platform can't check the toolbox, the response includes the warning `mcp_toolbox_inspection_failed`.

## Add a knowledge base to an agent

When you create a knowledge base, the platform creates a matching MCP connection named `kb-<knowledge-base-name>`. To let an agent search the knowledge base, add that connection as an `mcp` tool.

For a knowledge base named `benefits`:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "Answer HR questions. Always search the benefits knowledge base first, and cite your sources.",
      "tools": [
        {
          "type": "mcp",
          "server_label": "benefits",
          "project_connection_id": "kb-benefits"
        }
      ]
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/hr-helper/versions", json={
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "Answer HR questions. Always search the benefits knowledge base first, and cite your sources.",
        "tools": [{
            "type": "mcp",
            "server_label": "benefits",
            "project_connection_id": "kb-benefits",
        }],
    },
})
r.raise_for_status()
```

---

For a knowledge base tool, set only `type`, `server_label`, and `project_connection_id`. The platform fills in the rest when it saves the version:

- `server_description` becomes the knowledge base's `description`. Write a clear description for the knowledge base, because the agent uses it to decide when to search.
- `allowed_tools` becomes `{"tool_names": ["knowledge_base_retrieve"]}`, the one tool that a knowledge base offers. If you set `allowed_tools` yourself, it must have this exact value.

The knowledge base must exist and be ready when you create the version. Otherwise the request fails with `400 Bad Request`.

When the agent calls the knowledge base, the platform passes the end user's identity to it. Users only get content from collections that they have access to. The agent's answer includes citations that point to the source documents.

You can add several knowledge bases and MCP servers to the same agent. Give each one a clear description so the agent knows which one to use.

> [!NOTE]
> Microsoft 365 Local connects to on-premises SharePoint Server and Exchange Server. It's in limited preview for approved customers, and it's supported only with Agentic Retrieval. It isn't supported as a general Agent Service integration, so you don't add SharePoint or Exchange tools directly to an agent. Instead, create `sharepoint` or `exchange` knowledge sources, add them to a knowledge base, and then add the knowledge base to the agent as shown in this section. For setup, see [Connect Microsoft 365 Local](connect-microsoft-365-local.md).

## Control when the agent calls tools

By default, the agent decides on its own when to call a tool (`"tool_choice": "auto"`). To stop an agent version from calling tools, for example to test the model alone, set `"tool_choice": "none"` in the definition.

Instructions also guide tool use. For example: "Always search the benefits knowledge base before you answer an HR question."

## See which tools the agent called

When you run an agent with the Responses API, the response output shows each step:

- An `mcp_list_tools` item lists the tools that the agent found on a server.
- An `mcp_call` item shows each tool call, with its arguments and output.
- A `message` item holds the final answer.

For more information, see [Run agents with the Responses API](run-agents-responses-api.md).

## Remove a tool or a connection

To remove a tool from an agent, create a new agent version without it.

To delete an MCP connection:

# [curl](#tab/curl)

```bash
curl -X DELETE "$BASE/connections/inventory-mcp" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
api.delete(f"{BASE}/connections/inventory-mcp").raise_for_status()
```

---

If an agent version still uses the MCP connection, the request fails with `409 Conflict`. Don't change or delete the `kb-<name>` connections directly. The platform creates, updates, and deletes them with their knowledge bases.

## Related content

- [Create and manage agents](create-manage-agents.md)
- [Manage knowledge bases](knowledge-bases-guide.md)
- [MCP server in Agents and Tools with Foundry Local](mcp-server-overview.md)
- [Agents REST API reference](reference/agents-api-reference.md)
