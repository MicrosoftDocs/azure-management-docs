---
title: "Quickstart: Create an Agent in Agents and Tools with Foundry Local"
description: In this quickstart, you create a model connection and an agent in Agents and Tools with Foundry Local, then chat with it through the API and the browser.
ms.reviewer: assafakiva
ms.topic: quickstart
ms.date: 11/08/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-generated
# Customer intent: As an AI application developer, I want to create my first agent and talk to it so that I can see how Agent Service works.
---

# Quickstart: Create an agent and chat with it

In this quickstart, you create your first agent in **Agents and Tools with Foundry Local**. You connect a language model, create an agent with instructions, ask it a question by using the Responses API, and then chat with it in the browser.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

- A deployment of **Agents and Tools with Foundry Local** that you can reach at `https://<cluster-domain>`. To deploy one, see [Quickstart: Deploy Agents and Tools with Foundry Local](quickstart-edge-rag.md).
- A model served by Foundry Local on the same cluster, or another OpenAI-compatible endpoint. See [Set up language model endpoint](prepare-model-endpoint.md).
- A Microsoft Entra ID identity with **Cognitive Services Contributor** to create resources and **Cognitive Services OpenAI User** to run the agent, assigned at the extension scope. In disconnected deployments, use **Contributor**. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- **Reader** assigned to the extension's managed identity at the extension scope. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles).
- The Azure CLI, and Bash or Python 3.10 or later.

[!INCLUDE [agent-service-setup](includes/agent-service-setup.md)]

## Connect a language model

Create a model connection named `local-model`. This example uses a Foundry Local model that accepts an API key. Replace the endpoint, model name, and key with your values.

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
    "authentication": { "type": "ApiKey", "apiKey": "<your-api-key>" },
    "maxContextTokens": 131072
  }'
```

The service returns the connection. It never returns your API key:

```json
{
  "type": "ModelConnection",
  "kind": "FoundryOnArc",
  "endpoint": "https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/",
  "model": "gpt-oss-20b",
  "authentication": { "type": "ApiKey" },
  "maxContextTokens": 131072,
  "isReasoningModel": false,
  "name": "local-model",
  "id": "local-model"
}
```

# [Python](#tab/python)

```python
r = api.put(f"{BASE}/connections/local-model", json={
    "type": "ModelConnection",
    "kind": "FoundryOnArc",
    "endpoint": "https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/",
    "model": "gpt-oss-20b",
    "authentication": {"type": "ApiKey", "apiKey": "<your-api-key>"},
    "maxContextTokens": 131072,
})
r.raise_for_status()
print(r.json()["name"], r.json()["authentication"])
```

The service never returns your API key:

```output
local-model {'type': 'ApiKey'}
```

---

If your Foundry Local endpoint uses Microsoft Entra ID authentication instead of an API key, set `"authentication": { "type": "ManagedIdentity" }`. When you use managed identity, a cluster administrator must also approve the model endpoint URL (`https://gpt-oss-20b.foundry-local-operator.svc.cluster.local:5000/v1/`) in `serviceUrls` (`model-access-arc-rag/mi-destinations`). See [Create a model connection](create-manage-agents.md#create-a-model-connection) and [Approve endpoints for managed identity](create-manage-agents.md#approve-endpoints-for-managed-identity).

## Create an agent

Create the first version of an agent named `quickstart-helper`:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/agents/quickstart-helper/versions" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "description": "A friendly helper for the quickstart.",
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "You are a friendly helper. Answer in two sentences or fewer."
    }
  }'
```

The service returns version `1` of the agent:

```json
{
  "object": "agent.version",
  "id": "quickstart-helper:1",
  "name": "quickstart-helper",
  "version": "1",
  "description": "A friendly helper for the quickstart.",
  "created_at": 1792656119,
  "definition": {
    "kind": "prompt",
    "model": "local-model/gpt-oss-20b",
    "instructions": "You are a friendly helper. Answer in two sentences or fewer.",
    "tools": []
  },
  "metadata": {},
  "status": "active"
}
```

# [Python](#tab/python)

```python
r = api.post(f"{BASE}/agents/quickstart-helper/versions", json={
    "description": "A friendly helper for the quickstart.",
    "definition": {
        "kind": "prompt",
        "model": "local-model/gpt-oss-20b",
        "instructions": "You are a friendly helper. Answer in two sentences or fewer.",
    },
})
r.raise_for_status()
print(r.json()["id"])
```

```output
quickstart-helper:1
```

---

The agent is enabled right away. Each later change creates a new version with the next number.

## Ask the agent a question

Send a question by using the Responses API:

# [curl](#tab/curl)

```bash
curl -X POST "$BASE/openai/v1/responses" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "agent_reference": { "type": "agent_reference", "name": "quickstart-helper" },
    "input": "What can you help me with?"
  }'
```

The answer is in the `text` field of the `message` item in `output`. The following output is shortened, and your answer text will be different:

```json
{
  "id": "resp_35ce0dd241a44aafb754ec2d6d671955",
  "object": "response",
  "status": "completed",
  "model": "local-model/gpt-oss-20b",
  "output": [
    {
      "type": "message",
      "role": "assistant",
      "status": "completed",
      "content": [
        {
          "type": "output_text",
          "text": "I can answer questions and help you summarize or draft text.",
          "annotations": []
        }
      ]
    }
  ],
  "warnings": [],
  "agent_reference": {
    "type": "agent_reference",
    "name": "quickstart-helper",
    "version": "1"
  }
}
```

# [Python](#tab/python)

```python
response = client.responses.create(
    input="What can you help me with?",
    extra_body={
        "agent_reference": {"type": "agent_reference", "name": "quickstart-helper"}
    },
)
print(response.status)
print(response.output_text)
```

Your answer text will be different:

```output
completed
I can answer questions and help you summarize or draft text.
```

---

## Chat with the agent in the browser

1. Open the Local chat UI at `https://<cluster-domain>/chat` and sign in.
1. Select **quickstart-helper** from the list of agents.
1. Ask a question.

The agent streams its answer into the chat window. The Local chat UI saves the conversation in the sidebar, so you can come back to it later.

For connected deployments, users who only chat need **Reader** to list agents and **Cognitive Services OpenAI User** to run them. In disconnected deployments, use **Contributor**. `EdgeRAGEndUser` alone doesn't grant Responses or Conversations access. The `EdgeRAGDeveloper` app role remains a management fallback and is needed to read full definitions from agent list and get operations.

## Clean up resources

To delete the agent and the model connection resources:

# [curl](#tab/curl)

```bash
curl -X DELETE "$BASE/agents/quickstart-helper" -H "Authorization: Bearer $TOKEN"
curl -X DELETE "$BASE/connections/local-model" -H "Authorization: Bearer $TOKEN"
```

# [Python](#tab/python)

```python
api.delete(f"{BASE}/agents/quickstart-helper").raise_for_status()
api.delete(f"{BASE}/connections/local-model").raise_for_status()
```

---

## Next step

You created an agent and talked to it. Next, give your agent tools, such as a search tool that works over your own documents:

> [!div class="nextstepaction"]
> [Add tools to an agent](add-agent-tools.md)
