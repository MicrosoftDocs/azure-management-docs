---
title: "Quickstart: Create Your First Agentic RAG Agent"
description: Learn how to create an Agentic RAG agent end-to-end by registering knowledge, creating an agent, starting a conversation, and streaming a response from your LLM.
author: cwatson-cat
ms.author: cwatson
ms.topic: quickstart
ms.date: 04/29/2026
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Quickstart: Create your first Agentic RAG agent

In this quickstart, you create an Agentic RAG agent end-to-end by registering knowledge, creating an agent, starting a conversation, and streaming a response from your LLM.

## Prerequisites

- Agentic RAG deployed in *combined* mode (default). If you deployed in *agentic* mode, skip Steps 2-4 and create a `remote_mcp` knowledge source that points to an external MCP server in Step 3 instead.
- A *bring-your-own-model (BYOM) endpoint*: an LLM that exposes an OpenAI-compatible chat completions API (for example, FoundryOnArc, KAITO, or Azure OpenAI) configured at the cluster level during deployment through Helm values (`byom.apiEndpoint`).
- Azure CLI installed and authenticated.
- **curl** and **jq** installed.
- NFS data source with documents to ingest.

> [!Note]
> The `endpoint_url` field on agents is stored but not currently used by the runtime. All agents share the cluster-level BYOM endpoint configured via Helm values.

## Step 1: Get an access token

Use the Azure CLI to get an access token for API authentication. Replace `<your-tenant-id>` and `<app-registration-client-id>` with your Entra ID tenant ID and app registration client ID that has the `EdgeRAGDeveloper` role.

```azurecli
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<app-registration-client-id>" \
  --query accessToken -o tsv)

# Set your cluster domain
CLUSTER="<your-cluster-domain>"
```

## Step 2: Create a collection and ingest data

Collections are how you organize and store your data in the Agentic RAG system. In this step, you create a collection and ingest documents from an NFS data source.

### Create a collection

Use the API to create a new collection:

```bash
curl -s -X POST "https://$CLUSTER/edgeai/collections?api-version=2024-10-01-preview" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "quickstart-docs",
    "description": "Quickstart tutorial collection"
  }'
```

### Ingest documents from NFS

Ingest documents from your NFS data source into the collection. Replace `<nfs-server-ip>:/exports/my-docs` with the actual NFS server path.

```bash
curl -s -X POST "https://$CLUSTER/edgeai/ingestion/jobs/quickstart-ingest?api-version=2024-10-01-preview" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "datasource": {
      "kind": "nfsv3",
      "connection": {
        "nfsServerPath": "<nfs-server-ip>:/exports/my-docs"
      },
      "chunking": {
        "chunkSize": "2000",
        "chunkOverlap": "200"
      }
    },
    "collectionName": "quickstart-docs",
    "parsingMode": "advanced"
  }'
```

Wait for the ingestion job to complete. Check status:

```bash
curl -s "https://$CLUSTER/edgeai/ingestion/jobs?api-version=2024-10-01-preview" \
  -H "Authorization: Bearer $TOKEN" | jq '.jobRuns[] | {jobId, status}'
```

## Step 3: Create a knowledge source

Create a knowledge source that points to your collection by using the built-in MCP server:

```bash
KS_RESPONSE=$(curl -s -X POST "https://$CLUSTER/edgeai/knowledgesources" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"name\": \"quickstart-search\",
    \"kind\": \"indexed_sources_mcp\",
    \"auth_type\": \"microsoft_entra_id\",
    \"description\": \"Search the quickstart-docs collection\",
    \"indexed_sources_parameters\": {
      \"server_url\": \"https://$CLUSTER/edgeai/mcp\",
      \"indexed_source_ref\": \"quickstart-docs\",
      \"server_label\": \"indexed-sources-mcp-server\"
    }
  }")

KS_ID=$(echo $KS_RESPONSE | jq -r '.id')
KS_STATUS=$(echo $KS_RESPONSE | jq -r '.validation_status')
echo "Knowledge Source: $KS_ID (status: $KS_STATUS)"
```

Verify the `validation_status` is `active`. If validation fails, the request is rejected with `400 Bad Request` and no record is created. Check the error response for details.

> [!Note]
> The `indexed_source_ref` value (`quickstart-docs`) must match the collection name created in Step 2.

## Step 4: Create a knowledge base

Group the knowledge source into a knowledge base:

```bash
KB_RESPONSE=$(curl -s -X POST "https://$CLUSTER/knowledge-bases" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"name\": \"Quickstart KB\",
    \"description\": \"Knowledge base for the quickstart tutorial\",
    \"knowledge_source_ids\": [\"$KS_ID\"]
  }")

KB_ID=$(echo $KB_RESPONSE | jq -r '.data.id')
echo "Knowledge Base: $KB_ID"
```

## Step 5: Create an agent

Create an agent with your BYOM endpoint and knowledge base:

```bash
AGENT_RESPONSE=$(curl -s -X POST "https://$CLUSTER/agents" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"name\": \"Quickstart Agent\",
    \"instructions\": \"You are a helpful assistant. Answer questions using your knowledge base. Always cite your sources.\",
    \"endpoint_url\": \"<your-byom-endpoint-url>\",
    \"knowledge_base_id\": \"$KB_ID\",
    \"temperature\": 0.7,
    \"top_p\": 0.9
  }")

AGENT_ID=$(echo $AGENT_RESPONSE | jq -r '.data.id')
echo "Agent: $AGENT_ID"
```

Replace `<your-byom-endpoint-url>` with your LLM endpoint (e.g., `https://my-model.openai.azure.com/v1`).

## Step 6: Create a thread

Start a new conversation:

```bash
THREAD_RESPONSE=$(curl -s -X POST "https://$CLUSTER/threads" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "title": "Quickstart conversation"
  }')

THREAD_ID=$(echo $THREAD_RESPONSE | jq -r '.id')
echo "Thread: $THREAD_ID"
```

## Step 7: Send a message

Add a user message to the thread:

```bash
curl -s -X POST "https://$CLUSTER/threads/$THREAD_ID/messages" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "role": "user",
    "content": "What information do you have in the knowledge base? Summarize the key topics."
  }'
```

## Step 8: Execute a run (streaming)

Execute the agent against the thread with streaming enabled:

```bash
curl -N -X POST "https://$CLUSTER/threads/$THREAD_ID/runs?stream=true" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"agent_id\": \"$AGENT_ID\"
  }"
```

You'll see server-sent events (SSE) streaming back:

```output
event: thread.run.created
data: {"id": "run_...", "status": "queued", ...}

event: thread.run.in_progress
data: {"id": "run_...", "status": "in_progress", ...}

event: thread.message.delta
data: {"id": "msg_...", "delta": {"content": [{"type": "text", "text": {"value": "Based on the documents "}}]}}

event: thread.message.delta
data: {"id": "msg_...", "delta": {"content": [{"type": "text", "text": {"value": "in your knowledge base, the key topics include..."}}]}}

event: thread.run.completed
data: {"id": "run_...", "status": "completed", ...}

event: done
data: [DONE]
```

### View the complete response

After the run completes, retrieve all messages:

```bash
curl -s "https://$CLUSTER/threads/$THREAD_ID/messages?order=asc" \
  -H "Authorization: Bearer $TOKEN" | jq '.data[] | {role, content: .content[0].text.value}'
```

### Check run steps (optional)

See what the agent did during the run (tool calls, MCP calls, message creation):

```bash
# Get the run ID
RUN_ID=$(curl -s "https://$CLUSTER/threads/$THREAD_ID/runs?limit=1" \
  -H "Authorization: Bearer $TOKEN" | jq -r '.data[0].id')

# List steps
curl -s "https://$CLUSTER/threads/$THREAD_ID/runs/$RUN_ID/steps?order=asc" \
  -H "Authorization: Bearer $TOKEN" | jq '.data[] | {type, step_details}'
```

## Troubleshooting

If you encounter issues, check the following common problems and solutions:

| Problem | Solution |
|---|---|
| Knowledge source validation status is `failed` | Check `validation_error`. Verify the MCP server URL is reachable. Update via PATCH to trigger re-validation. |
| Run status is `failed` | Check `last_error` on the run object. Common issues: BYOM endpoint unreachable, model timeout, invalid endpoint URL. |
| `401 Unauthorized` | Token expired. Re-acquire: `TOKEN=$(az account get-access-token ...)` |
| `403 Forbidden` | Missing `EdgeRAGDeveloper` role on token. Check Entra ID app role assignments. |
| `404 Not Found` on thread/message | Thread ownership is enforced — you can only access threads you created. |
| Ingestion job stuck in `PENDING` | Check ingestion service logs. Verify NFS path is accessible from the cluster. |

## Next step

> [!div class="nextstepaction"]
> [Add more knowledge sources](add-data-source.md)

## Related articles

- [Agentic layer overview](agentic-overview.md)
- [MCP server in Agentic RAG](mcp-server-overview.md)
<!-- - [Collections overview](collections-overview.md)
- [Agent Manager API reference](APIs/agent-manager-api.md)
- [Agents Runtime API reference](APIs/agents-runtime-api.md) -->
