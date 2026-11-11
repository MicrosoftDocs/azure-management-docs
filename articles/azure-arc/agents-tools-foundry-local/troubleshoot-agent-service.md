---
title: Troubleshoot Agent Service in Agents and Tools with Foundry Local
description: Find and fix common problems with agents, connections, tools, knowledge bases, and responses in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: troubleshooting-general
ms.date: 11/09/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer or operator, I want to understand and fix Agent Service errors so that my agents keep working.
---

# Troubleshoot Agent Service in Agents and Tools with Foundry Local

This article helps you find and fix common problems with Agent Service in Agents and Tools with Foundry Local. Start with the error code and message that the API returns. Then use the tables to find the cause.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Read the error

Errors from API handlers use this format:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "Model must use the <connection>/<model> format",
    "param": "model"
  }
}
```

- `code` is a short, stable error code.
- `message` explains the problem.
- `param` names the request field that caused it, when there is one.

Authentication and authorization middleware checks (`401`, `403 Insufficient permissions`, `403 Route is not authorized`, `403 Unable to resolve request owner`, and `503`) run before the API handler and return `{"detail": "<message>"}`. API handler and gateway errors in the following table (`403 Agent '...' is disabled`, `404`, and `410`) return `{"error": {"code": "...", "message": "..."}}`.

## Sign-in and access errors

| Status | Message or symptom | Cause | Fix |
|---|---|---|---|
| `401` | `Authentication required` or `Authentication failed` | The bearer token is missing, invalid, expired, or for the wrong app. The response includes `WWW-Authenticate: Bearer`. | Get a fresh token for `api://<app-registration-client-id>` and send it in the Authorization header. |
| `403` | `Insufficient permissions` | Azure roles don't allow the operation, and no supported app-role fallback allows it. | Assign **Reader** for reads, **Cognitive Services Contributor** for changes, or **Cognitive Services OpenAI User** for Responses and Conversations at the extension scope. In disconnected deployments, use **Contributor** for changes and chat. App roles alone don't allow Responses or Conversations. |
| `403` | `Route is not authorized` | The requested route has no authorization policy. | Check the URL and method against the current API reference. If a documented route fails, contact the operator. Adding a role doesn't authorize an unknown route. |
| `403` | `Unable to resolve request owner` | The service can't determine the owner required by the route. | Check the caller's identity and the service's owner-resolution configuration with the operator. |
| `503` | `Authorization service unavailable` | The Azure Resource Manager role check is unavailable and no app-role fallback allows the request. | Assign **Reader** to the extension's managed identity at the extension resource scope. Check connectivity to Azure Resource Manager and avoid conditional role assignments, which don't grant access. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles). |
| `503` | `Authentication service unavailable` | The service can't validate the token through its authentication dependency. | Ask the operator to check the authentication service and identity-provider connectivity. Assigning a caller role doesn't repair token validation. |
| `403` | Agent '...' is disabled and cannot start new Responses | The agent is disabled. | Enable it with `POST /agents/<name>:enable`. |
| `404` | Conversation or response not found | It doesn't exist, another caller owns it, or a stored response belongs to a different tenant. | Use the same caller and tenant that created the resource. |
| `410` | The API is retired | The app calls a retired API, such as `/threads`, `/knowledge-bases`, or `POST /agents` with the old request body. | Move to the current APIs. See [Agent APIs from earlier previews](agentic-overview.md#agent-apis-from-earlier-previews). |

For role setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles). Review inherited roles and deny assignments at subscription, resource-group, and cluster scopes. Allow time for Azure propagation after changes. The service also caches allows for 300 seconds and denies for 30 seconds.

## Model connection errors

| Status | Message | Cause | Fix |
|---|---|---|---|
| `400` | An administrator must approve the managed identity destination | The model endpoint isn't on the approved list for managed identity. | Approve the endpoint. See [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity). |
| `400` | Model Connection settings are invalid | A field is missing or not valid. Common causes: `endpoint` doesn't end with `/v1/`, using an IP address with managed identity, or `OpenAICompatible` doesn't use an API key. | Check the fields in [Create a model connection](create-manage-agents.md#create-a-model-connection). |
| `400` | Model binding must contain one connection and one model | The agent's `model` value has the wrong format. | Use `<model-connection-name>/<model-name>`, for example `local-model/gpt-oss-20b`. |
| `200` with `status: failed` | `error.code` is `server_error` | The model endpoint returned an error, or the platform couldn't get a token. | Check that the model is running, and that the managed identity or API key is valid. Check the agents execution logs. |
| `200` with `status: failed` | `error.code` is `rate_limit_exceeded` | The model endpoint limited the requests. | Retry later, or raise the limit on the model endpoint. |

## Tool and MCP errors

| Status | Message or symptom | Cause | Fix |
|---|---|---|---|
| `400` | missing RBAC permissions for toolbox: `<label>` | The extension's managed identity can't access the Microsoft Foundry toolbox. | Give the managed identity access to the toolbox in the Foundry project, and try again. |
| `400` | Managed Knowledge Base Connection is invalid | A knowledge base tool has the wrong `allowed_tools`, or the connection isn't a `kb-` connection. | Set only `type`, `server_label`, and `project_connection_id: kb-<name>`. |
| `400` | Knowledge Base '...' was not found, or is not ready | The knowledge base doesn't exist yet, or is still being set up. | Check the name, wait until the knowledge base is ready, and try again. |
| `400` | MCP server labels must be unique | Two tools in the version have the same `server_label`. | Give each tool its own label. |
| `409` | Conflict when you delete a connection | An agent version uses the MCP connection. | Remove the connection from all agent versions first. To remove a `kb-` connection, delete its knowledge base. |
| Answer without tool use | The agent answers but doesn't call the tool. | The tool or knowledge base description isn't clear, the instructions don't ask for it, or the collection has no documents. | Improve the knowledge base `description`, add instructions such as "Always search the knowledge base first", and check that the collection has ingested documents. |
| Answer without citations | The answer has no `[cite:...]` markers. | The search returned no content that the user can see. | Check the connected knowledge sources and ingested content. If the search uses app-role fallback, check the collection app role. Azure-role-authorized callers don't have a per-collection filter. See [Collections overview](collections-overview.md). |

### Tool warnings in a response

When a tool isn't available but the agent can still answer, the response includes a `warnings` list. Each warning has a `code` and the `server_label` of the tool.

| Warning code | Meaning |
|---|---|
| `mcp_initialization_failed` | The platform couldn't start a session with the MCP server. |
| `mcp_authentication_failed` | The MCP server didn't accept the credentials, or the platform couldn't get a token. |
| `mcp_tools_unavailable` | The MCP server offered no tools that the agent can use. |
| `mcp_tool_validation_failed` | A tool's definition from the server isn't valid. |
| `mcp_call_failed` | A tool call failed. |
| `mcp_toolbox_inspection_failed` | The platform couldn't check the tools in a Microsoft Foundry toolbox. |

Check that the MCP server is running and reachable from the cluster, and that the connection's authentication settings are correct.

## Agent and version errors

| Status | Message | Cause | Fix |
|---|---|---|---|
| `409` | `AgentVersion '...' is the newest version and cannot be deleted` | Version numbers are never reused, so you can't delete the newest version while older ones exist. | Create a new version instead, or delete the whole agent. |
| `409` | Agent '...' has an active Response | A response is still running with this agent or version. | Disable the agent, wait for running responses to finish, and try again. |

## Conversation and response errors

| Status | Message | Cause | Fix |
|---|---|---|---|
| `400` | previous_response_id cannot reference a Conversation-backed Response | You chained a response that belongs to a conversation. | Use `conversation` to continue a conversation, or chain only responses outside conversations. |
| `400` | Invalid $filter | The filter uses a field or operator that isn't supported. | See [List responses query parameters](reference/responses-api-reference.md#list-responses-query-parameters). |
| `409` | A Conversation cannot contain more than 2000 Items | The conversation is full. | Start a new conversation. |
| `400` | `Response '...' cannot be cancelled; current status is ...` | The response already finished, so there's nothing to cancel. | No action needed. |

## Knowledge base and knowledge source errors

| Status | Message | Cause | Fix |
|---|---|---|---|
| `400` | Field required (`param` is `api-version`), or api-version must be 2026-08-01-preview | The Knowledge Bases API needs `api-version=2026-08-01-preview`. | Add the query parameter to every request. |
| `400` | Knowledge source validation failed | The service couldn't connect to the MCP server. | Check the `server_url`, and that the server is reachable from the cluster. |
| `409` | Knowledge Base is in use | An agent version uses the knowledge base. | Remove it from the agents, and delete the old versions that use it. |
| `412` | `Knowledge Base ETag does not match` or `Knowledge Base already exists` | The `If-Match` ETag is out of date, or the knowledge base already exists with `If-None-Match: *`. | Get the knowledge base again, and retry with the new ETag. |

## Check service logs

If you have access to the cluster, check the logs of the Agent Service workloads in the extension namespace, such as `arc-rag`:

```bash
kubectl get pods -n arc-rag | grep -E "agents-management|agents-execution|kb-execution"

kubectl logs -n arc-rag deployment/agents-management-deployment --tail=100
kubectl logs -n arc-rag deployment/agents-execution-deployment --tail=100
kubectl logs -n arc-rag deployment/kb-execution-deployment --tail=100
```

| Service | Check its logs when |
|---|---|
| `agents-management` | Creating agents, versions, connections, or knowledge bases fails. |
| `agents-execution` | Responses fail, are slow, or tools don't run. |
| `kb-execution` | Knowledge base searches return nothing or fail. |

For metrics and dashboards, see [Monitor Agents and Tools with Foundry Local](observability.md).

## Related content

- [Agent Service limits](reference/agent-service-limits.md)
- [Security and access for Agent Service](agent-service-security.md)
- [Known issues](known-issues.md)
