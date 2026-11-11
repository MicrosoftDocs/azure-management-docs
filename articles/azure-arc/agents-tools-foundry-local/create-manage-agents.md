---
title: Create and Manage Agents in Agents and Tools with Foundry Local
description: Learn how to create model connections, agents, and agent versions in Agents and Tools with Foundry Local, and how to select, disable, or delete them.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer, I want to create and version agents in Agents and Tools with Foundry Local so that I can control which instructions, model, and tools answer my users.
---

# Create and manage agents in Agents and Tools with Foundry Local

This article shows you how to create an agent in Agents and Tools with Foundry Local and keep it up to date. You create a model connection, create the first agent version, then learn how to add versions, pick the version that answers users, turn the agent on or off, and delete it.

If you're new to agents, model connections, and agent versions, read [Agent Service overview](agentic-overview.md#key-concepts) first.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

- An Agents and Tools with Foundry Local deployment. See [Deployment overview](deploy-overview.md).
- A language model endpoint that offers an OpenAI-compatible API, such as Foundry Local on Azure Local. See [Set up language model endpoint](prepare-model-endpoint.md).
- A Microsoft Entra ID user with **Cognitive Services Contributor** at the extension scope, or **Contributor** in disconnected deployments (**Reader** is sufficient for read-only operations). The `EdgeRAGDeveloper` app role also works as a fallback. To read full agent definitions, the user also needs the `EdgeRAGDeveloper` app role. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

## Create a model connection

An agent needs a language model. You tell the platform how to reach the model by creating a *model connection*. You create it once, and many agents can use it.

Send a `PUT` request to `/connections/<connection-name>`. The connection name must use lowercase letters, digits, and hyphens, and be 1 to 63 characters long.

The following example creates a model connection named `local-model` for a model served by Foundry Local on the same cluster. It signs in by using the extension's managed identity.

# [curl](#tab/curl)

```bash
curl -X PUT "$BASE/connections/local-model" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "type": "ModelConnection",
    "kind": "FoundryOnArc",
    "endpoint": "https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/",
    "model": "gpt-oss-20b",
    "authentication": { "type": "ManagedIdentity" },
    "maxContextTokens": 131072
  }'
```

# [Python](#tab/python)

```python
r = api.put(f"{BASE}/connections/local-model", json={
    "type": "ModelConnection",
    "kind": "FoundryOnArc",
    "endpoint": "https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/",
    "model": "gpt-oss-20b",
    "authentication": {"type": "ManagedIdentity"},
    "maxContextTokens": 131072,
})
r.raise_for_status()
```

---

The following table describes the fields of a model connection.

| Field | Required | Description |
|---|---|---|
| `type` | Yes | Always `ModelConnection`. |
| `kind` | Yes | `FoundryOnArc` for Foundry Local on Azure Local, `FoundryCloud` for Microsoft Foundry, or `OpenAICompatible` for any other OpenAI-compatible endpoint. |
| `endpoint` | Yes | The HTTPS base address of the model API. It must end with `/v1/`. |
| `model` | Yes | The model or deployment name, such as `gpt-oss-20b`. It can't contain `/`. |
| `authentication.type` | Yes | `ManagedIdentity` or `ApiKey`. `OpenAICompatible` supports `ApiKey` only. |
| `authentication.audience` | No | For `ManagedIdentity` only. The token audience. If you leave it out, the platform uses a default: the Foundry Local client ID that you set at install time (`foundryClientId`) for `FoundryOnArc`, or the matching Azure audience for `FoundryCloud`. |
| `authentication.apiKey` | For `ApiKey` | The API key. The platform stores it as a secret and never returns it. |
| `maxContextTokens` | Yes | The model's context window, in tokens, from 4,096 to 10,000,000. |
| `isReasoningModel` | No | Set to `true` for reasoning models. Default `false`. |
| `reasoningEffort` | No | The reasoning effort to request. Only valid when `isReasoningModel` is `true`. |

To use an API key instead of managed identity, set `authentication` like this:

```json
"authentication": { "type": "ApiKey", "apiKey": "<your-api-key>" }
```

> [!IMPORTANT]
> To use `ManagedIdentity`, the endpoint must use a DNS name, not an IP address, and a cluster administrator must approve it. If the endpoint isn't approved, the request fails with `400 Bad Request` and the message "An administrator must approve the managed identity destination". See the next section.

## Approve endpoints for managed identity

When the platform calls a model or an MCP server by using the extension's managed identity, it sends the token only to addresses that a cluster administrator approves. This rule keeps the identity's tokens from going to the wrong place. Endpoints that use an API key or no authentication don't need approval.

The ConfigMap `mi-destinations` in the namespace `model-access-<extension-namespace>` (for example, `model-access-arc-rag`) stores the approved addresses. The value `approvedMiDestinations` holds a JSON array of HTTPS base addresses. An address approves itself and every path under it.

Set the list in one of these ways:

- **At install or update time.** Pass the extension setting `serviceUrls` as a JSON array in a string. The setting applies to the whole extension, so you set it once for all agents and connections. The following example approves one Foundry Local model:

  ```bash
  --config 'serviceUrls=["https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/"]'
  ```

  Each Foundry Local model has its own host name, and one address can't approve more than one host name. To approve every Foundry Local model that your agents use, list the address of each model in the same setting:

  ```bash
  --config 'serviceUrls=["https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/","https://gpt-oss-120b.foundry-local-operator.svc.cluster.local:5000/v1/"]'
  ```

  Use the first example if your agents use only one model. Use the second example if they use more than one model. When you deploy another model later, add its address to the list.

- **After installation.** A Kubernetes user with write access to the ConfigMap can edit it directly:

  ```bash
  kubectl -n model-access-arc-rag edit configmap mi-destinations
  ```

Keep the following rules in mind:

- Approved addresses must use HTTPS and a DNS name. IP addresses aren't accepted for managed identity, even if they're on the list.
- The host name and port of an endpoint must match an approved address exactly. Wildcards such as `*` aren't supported.
- Don't include a query string in an approved address.
- When you change `serviceUrls`, the new value replaces the whole list, including changes that you made directly in the ConfigMap. Include every address that you still need.
- A list that isn't valid JSON, or that has an address that isn't valid, blocks all managed identity calls to models and MCP servers. Check the whole list after you edit it.

## Create an agent

You create an agent by creating its first *agent version*. Send a `POST` request to `/agents/<agent-name>/versions`. If the agent doesn't exist yet, the platform creates it.

Agent names use letters, digits, and hyphens, are 1 to 63 characters long, and must start and end with a letter or digit. The platform stores names in lowercase.

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "description": "Answers questions about HR policies.",
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "You are an HR assistant. Answer briefly and politely.",
      "temperature": 0.2
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/hr-helper/versions", json={
    "description": "Answers questions about HR policies.",
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "You are an HR assistant. Answer briefly and politely.",
        "temperature": 0.2,
    },
})
r.raise_for_status()
version = r.json()
print(version["id"])  # hr-helper:1
```

---

The response contains the new version:

```json
{
  "object": "agent.version",
  "id": "hr-helper:1",
  "name": "hr-helper",
  "version": "1",
  "description": "Answers questions about HR policies.",
  "created_at": 1792575600,
  "definition": {
    "kind": "prompt",
    "model": "local-model/gpt-oss-20b",
    "instructions": "You are an HR assistant. Answer briefly and politely.",
    "temperature": 0.2,
    "tools": []
  },
  "metadata": {},
  "status": "active"
}
```

A new agent is **enabled** and uses its latest version, so users can chat with it right away.

### Agent version fields

The following table describes the fields you can send when you create a version.

| Field | Required | Description |
|---|---|---|
| `description` | No | A short description, up to 512 characters. End users see it when they list agents. |
| `metadata` | No | Up to 16 key-value pairs. Keys can be up to 64 characters, and values up to 512 characters. |
| `definition.kind` | Yes | `prompt`. |
| `definition.model` | Yes | The model connection name and the model name, separated by `/`. For example, `local-model/gpt-oss-20b`. |
| `definition.instructions` | No | The instructions that tell the agent how to behave. |
| `definition.temperature` | No | How creative the answers are, from `0` to `2`. Lower values give more focused answers. |
| `definition.top_p` | No | Another way to control variety, from `0` to `1`. Change `temperature` or `top_p`, not both. |
| `definition.tools` | No | Up to 32 tools that the agent can call. See [Add tools to an agent](add-agent-tools.md). |
| `definition.tool_choice` | No | `auto` (default) lets the agent decide when to call tools. `none` stops it from calling tools. |
| `definition.text.format` | No | The output format: `{"type": "text"}`, `{"type": "json_object"}`, or `{"type": "json_schema", "name": "...", "schema": {...}}`. |

## Change an agent

You can't edit an agent version. To change an agent, create a new version with the full new settings:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "description": "Answers questions about HR policies.",
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "You are an HR assistant. Answer in three sentences or fewer. Say when you are not sure."
    }
  }'
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/hr-helper/versions", json={
    "description": "Answers questions about HR policies.",
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "You are an HR assistant. Answer in three sentences or fewer. Say when you are not sure.",
    },
})
r.raise_for_status()
```

---

The platform numbers the new version `2`. If the agent follows its latest version, new conversations use version `2` right away.

To see all versions of an agent, send `GET /agents/hr-helper/versions?limit=20&order=desc`.

## Choose the version that answers users

By default, an agent uses its latest version (`@latest`). To test a new version before users see it, you can pin the agent to a specific version. Send a `PATCH` request to `/agents/<agent-name>`:

# [curl](#tab/curl)

```bash
curl -X PATCH "$BASE/agents/hr-helper" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "agent_endpoint": {
      "version_selector": {
        "version_selection_rules": [
          { "type": "FixedRatio", "agent_version": "1", "traffic_percentage": 100 }
        ]
      }
    }
  }'
```

# [Python](#tab/python)

```python
r = api.patch(f"{BASE}/agents/hr-helper", json={
    "agent_endpoint": {"version_selector": {"version_selection_rules": [
        {"type": "FixedRatio", "agent_version": "1", "traffic_percentage": 100},
    ]}},
})
r.raise_for_status()
```

---

Now users get version `1`, even after you create newer versions. To follow the latest version again, set `agent_version` to `@latest`. To roll back after a bad change, pin the agent to the last good version.

Developers can still test any version directly. In the Responses API, set `agent_reference.version`. See [Run agents with the Responses API](run-agents-responses-api.md).

## Turn an agent off or on

To stop users from starting new responses with an agent, disable it:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/hr-helper:disable" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
api.post(f"{BASE}/agents/hr-helper:disable").raise_for_status()
```

---

Responses that are already running finish normally. New requests to the agent fail with `403 Forbidden`, and end users no longer see the agent when they list agents. To turn the agent on again, send the same request to `/agents/hr-helper:enable`. Both requests return `204 No Content`.

## Delete an agent version or an agent

To delete one version, send `DELETE /agents/hr-helper/versions/1`. To delete an agent and all its versions, send `DELETE /agents/hr-helper`.

# [curl](#tab/curl)

```bash
curl -X DELETE "$BASE/agents/hr-helper" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
api.delete(f"{BASE}/agents/hr-helper").raise_for_status()
```

---

The following rules apply:

- You can't delete a version or an agent while a response is running with it. The request fails with `409 Conflict`. Disable the agent and wait for running responses to finish first.
- You can't delete the newest version while older versions exist, because version numbers are never reused. To replace the newest version, create a new version instead.
- If you delete the version that the agent is pinned to, the agent goes back to `@latest`.
- If you delete the only version, the platform deletes the agent too.
- Deleting an agent doesn't delete its model connection, or MCP connections that were created inline. Delete those separately if you don't need them.

## List agents

Developers see the full definition of each agent:

# [curl](#tab/curl)

```bash
curl "$BASE/agents?limit=20" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
for agent in api.get(f"{BASE}/agents", params={"limit": 20}).json()["data"]:
    print(agent["name"], agent["state"], agent["versions"]["latest"]["version"])
```

---

Callers with **Reader** at the extension scope (or the `EdgeRAGEndUser` app-role fallback) whose token doesn't include `EdgeRAGDeveloper` see enabled agents only, with their `id`, `name`, and `description`.

## Related content

- [Add tools to an agent](add-agent-tools.md)
- [Run agents with the Responses API](run-agents-responses-api.md)
- [Agents REST API reference](reference/agents-api-reference.md)
- [Agent Service overview](agentic-overview.md)
