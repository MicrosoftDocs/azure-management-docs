---
title: Manage Knowledge Bases in Agents and Tools with Foundry Local
description: Learn how to create, update, and delete knowledge bases in Agents and Tools with Foundry Local, and how agents use them to search your knowledge sources.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer, I want to group my knowledge sources into knowledge bases so that my agents can search the right content and cite it.
---

# Manage knowledge bases in Agents and Tools with Foundry Local

A *knowledge base* groups one or more [knowledge sources](knowledge-sources-guide.md) so that an agent can search them as one tool. It also holds instructions that guide how the platform searches and how it writes the grounded content that it returns to the agent.

This article shows you how to create a knowledge base, connect it to an agent, and update or delete it. The Knowledge Bases API follows the shape of the knowledge bases API in Azure AI Search.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## How knowledge bases work

1. You create knowledge sources, for example one for each collection.
1. You create a knowledge base that lists those knowledge sources and names a model connection.
1. The platform creates an MCP connection named `kb-<knowledge-base-name>`. This connection is the knowledge base's MCP endpoint, and offers one tool, `knowledge_base_retrieve`.
1. You add the `kb-<knowledge-base-name>` connection to an agent version as a tool.
1. When a user asks a question, the agent calls `knowledge_base_retrieve`. The platform searches the knowledge sources with the user's identity and returns content with references. The agent uses it in the answer and shows citations.

Each knowledge base is independent. You can create several knowledge bases, and you can add several knowledge bases to one agent.

The knowledge base MCP endpoint is internal to the cluster. Agents on the cluster use it. Applications outside the cluster use [Indexed Sources MCP](mcp-server-overview.md) at `/mcp` to search collections directly.

## Prerequisites

- At least one knowledge source. See [Configure a knowledge source](knowledge-sources-guide.md).
- A model connection. See [Create a model connection](create-manage-agents.md#create-a-model-connection).
- A Microsoft Entra ID identity with **Cognitive Services Contributor** at the extension resource scope to create, replace, or delete knowledge bases. **Reader** is enough to list and get knowledge bases. In disconnected deployments, use **Contributor** for changes. These management operations support `EdgeRAGDeveloper` as an app-role fallback.

Azure roles are checked first. The extension's managed identity needs **Reader** at the extension scope to check caller roles. To run an agent that uses the knowledge base, also assign **Cognitive Services OpenAI User** in connected deployments, or **Contributor** in disconnected deployments. Responses and Conversations have no app-role fallback.

For role setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles). For separate model and tool permissions, see [Security and access](agent-service-security.md#permission-checks).

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

Every request to the Knowledge Bases API must include the query parameter `api-version=2026-08-01-preview`.

## Create a knowledge base

Send a `PUT` request to `/knowledgebases('<name>')`. The following example creates a knowledge base named `benefits` that searches two knowledge sources and uses the model connection `local-model`.

# [curl](#tab/curl)

```bash
curl -X PUT "$BASE/knowledgebases('benefits')?api-version=2026-08-01-preview" \
  -H "Content-Type: application/json" \
  -H "If-None-Match: *" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "benefits",
    "description": "Employee health benefits, leave, and enrollment policies.",
    "retrievalInstructions": "Prefer the newest policy documents.",
    "answerInstructions": "Quote the policy section that supports each statement.",
    "outputMode": "answerSynthesis",
    "knowledgeSources": [
      { "name": "benefits-docs" },
      { "name": "hr-faq" }
    ],
    "models": [
      {
        "kind": "azureOpenAI",
        "azureOpenAIParameters": {
          "resourceUri": "connection://local-model",
          "deploymentId": "gpt-oss-20b"
        }
      }
    ],
    "retrievalReasoningEffort": { "kind": "low" }
  }'
```

# [Python](#tab/python)

```python
KB_API = {"api-version": "2026-08-01-preview"}

r = api.put(
    f"{BASE}/knowledgebases('benefits')",
    params=KB_API,
    headers={"If-None-Match": "*"},
    json={
        "name": "benefits",
        "description": "Employee health benefits, leave, and enrollment policies.",
        "retrievalInstructions": "Prefer the newest policy documents.",
        "answerInstructions": "Quote the policy section that supports each statement.",
        "outputMode": "answerSynthesis",
        "knowledgeSources": [{"name": "benefits-docs"}, {"name": "hr-faq"}],
        "models": [{
            "kind": "azureOpenAI",
            "azureOpenAIParameters": {
                "resourceUri": "connection://local-model",
                "deploymentId": "gpt-oss-20b",
            },
        }],
        "retrievalReasoningEffort": {"kind": "low"},
    },
)
r.raise_for_status()
etag = r.headers["ETag"]
```

---

The `If-None-Match: *` header ensures that you don't replace an existing knowledge base by mistake. The service returns `201 Created` and the knowledge base, with an `@odata.etag` value and an `ETag` header.

You can also create a knowledge base with `POST /knowledgebases?api-version=2026-08-01-preview` and the same body. `POST` fails if the knowledge base already exists.

### Knowledge base fields

| Field | Required | Description |
|---|---|---|
| `name` | Yes | 1 to 40 lowercase letters, digits, and hyphens. It must start and end with a letter or digit, and match the name in the URL. |
| `description` | Yes | What the knowledge base contains, up to 512 characters. Agents read this description to decide when to search, so make it clear and specific. |
| `knowledgeSources` | Yes | 1 to 100 knowledge sources, by name. |
| `models` | Yes | Exactly one model. Set `kind` to `azureOpenAI`. In `azureOpenAIParameters`, set `resourceUri` to `connection://<model-connection-name>`, and set `deploymentId` to the model name in that connection. Any model connection kind works. |
| `retrievalInstructions` | No | Guidance for source selection and searching, up to 8,192 characters. |
| `answerInstructions` | No | Guidance for composing the grounded answer, up to 8,192 characters. |
| `outputMode` | No | Reserved compatibility field. The accepted value is `answerSynthesis`, but current execution doesn't use this field. |
| `retrievalReasoningEffort` | No | Reserved compatibility field. Accepts `{"kind": "low"}` or `{"kind": "medium"}`, but current execution doesn't use this field. |

The `encryptionKey` and `corsOptions` fields aren't supported in this version.

## Add a knowledge base to an agent

Add the connection `kb-<knowledge-base-name>` to a new agent version as an `mcp` tool:

```json
{
  "type": "mcp",
  "server_label": "benefits",
  "project_connection_id": "kb-benefits"
}
```

For the full steps, see [Add a knowledge base to an agent](add-agent-tools.md#add-a-knowledge-base-to-an-agent).

## Get and list knowledge bases

# [curl](#tab/curl)

```bash
curl "$BASE/knowledgebases('benefits')?api-version=2026-08-01-preview" \
  -H "Authorization: Bearer $TOKEN"

curl "$BASE/knowledgebases?api-version=2026-08-01-preview" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
kb = api.get(f"{BASE}/knowledgebases('benefits')", params=KB_API).json()

for kb in api.get(f"{BASE}/knowledgebases", params=KB_API).json()["value"]:
    print(kb["name"], [s["name"] for s in kb["knowledgeSources"]])
```

---

The list response returns the knowledge bases in a `value` array.

## Update a knowledge base

To change a knowledge base, send a `PUT` request with the full new body. `PUT` replaces the whole knowledge base, so include every field that you want to keep, including all knowledge sources.

To ensure that nobody changed the knowledge base since you read it, send the `@odata.etag` value in an `If-Match` header:

# [curl](#tab/curl)

```bash
curl -X PUT "$BASE/knowledgebases('benefits')?api-version=2026-08-01-preview" \
  -H "Content-Type: application/json" \
  -H "If-Match: <etag-value>" \
  -H "Authorization: Bearer $TOKEN" \
  -d @benefits-kb.json
```

# [Python](#tab/python)

```python
kb = api.get(f"{BASE}/knowledgebases('benefits')", params=KB_API).json()
etag = kb.pop("@odata.etag")
kb["knowledgeSources"].append({"name": "product-manuals"})

r = api.put(f"{BASE}/knowledgebases('benefits')", params=KB_API,
            headers={"If-Match": etag}, json=kb)
r.raise_for_status()
```

---

If the knowledge base changed in the meantime, the service returns `412 Precondition Failed`. Read it again and retry.

Agents that use the knowledge base use its updated source and model associations, retrieval instructions, and answer instructions. The reserved output-mode and reasoning-effort fields don't currently change retrieval or answer composition. The platform copies the knowledge base `description` into the agent version when you create the version. If you change the description, create a new agent version so the agent sees it.

## Delete a knowledge base

# [curl](#tab/curl)

```bash
curl -X DELETE "$BASE/knowledgebases('benefits')?api-version=2026-08-01-preview" \
  -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
api.delete(f"{BASE}/knowledgebases('benefits')", params=KB_API).raise_for_status()
```

---

The platform also deletes the `kb-<knowledge-base-name>` connection. If an agent version still uses the knowledge base, the request fails with `409 Conflict`. Remove the knowledge base from your agents first by creating new versions without it, and delete the old versions that use it.

## Search a knowledge base

Agents search knowledge bases through MCP. The REST endpoint `POST /knowledgebases('<name>')/retrieve` isn't supported in this version and returns `501 Not Implemented`. To test a knowledge base, add it to an agent and ask the agent a question. See [Run agents with the Responses API](run-agents-responses-api.md).

## Related content

- [Configure a knowledge source](knowledge-sources-guide.md)
- [Add tools to an agent](add-agent-tools.md)
- [Agent Service overview](agentic-overview.md)
- [Knowledge Bases REST API reference](reference/knowledge-bases-api-reference.md)
