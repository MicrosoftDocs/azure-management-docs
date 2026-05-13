---
title: "Quickstart: Query Your Data with Agents and Tools with Foundry Local"
description: Learn how to query your data with Agents and Tools with Foundry Local by registering knowledge, linking it to your default knowledge base, starting a conversation, and streaming a response from your LLM.
author: cwatson-cat
ms.author: cwatson
ms.topic: quickstart
ms.date: 04/29/2026
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Quickstart: Query your data with Agents and Tools with Foundry Local

In this quickstart, you query your data with Agents and Tools with Foundry Local by ingesting data, registering it as a knowledge source, linking it to your default knowledge base, and running a conversation with streaming.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you have:

- Agents and Tools with Foundry Local deployed in *combined* mode (default). If you deployed in *agentic* mode, skip Steps 2-3 and create a `remote_mcp` knowledge source that points to an external MCP server in Step 3 instead. See [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md).
- A *bring-your-own-model (BYOM) endpoint*: an LLM that exposes an OpenAI-compatible chat completions API (for example, Foundry Local on Azure Local or Microsoft Foundry) configured at the cluster level during deployment through Helm values (`byom.apiEndpoint`).
- Azure CLI installed and authenticated.
- **curl** and **jq** installed.
- NFS data source with documents to ingest.

Each deployment includes a default knowledge base that is automatically provisioned. You link knowledge sources to this default knowledge base.

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

Collections are how you organize and store your data in the Agents and Tools with Foundry Local system. In this step, you create a collection and ingest documents from an NFS data source.

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

## Step 4: Link the knowledge source to your default knowledge base

Get the default knowledge base ID and link the knowledge source to it:

```bash
# Get default knowledge base ID
KB_ID=$(curl -s "https://$CLUSTER/knowledge-bases?limit=1" \
  -H "Authorization: Bearer $TOKEN" | jq -r '.data[0].id')
echo "Knowledge Base: $KB_ID"

# Link the knowledge source
curl -s -X PATCH "https://$CLUSTER/knowledge-bases/$KB_ID" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"knowledge_source_ids\": [\"$KS_ID\"]
  }"
```

## Step 5: Create a thread

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

## Step 6: Send a message

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

## Step 7: Execute a run (streaming)

Execute a run against the thread with streaming enabled, using the knowledge base ID:

```bash
curl -N -X POST "https://$CLUSTER/threads/$THREAD_ID/runs?stream=true" \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d "{
    \"agent_id\": \"$KB_ID\"
  }"
```

You see server-sent events (SSE) streaming back:

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
| Knowledge source validation status is `failed` | Check `validation_error`. Verify the MCP server URL is reachable. Update via PATCH to trigger revalidation. |
| Run status is `failed` | Check `last_error` on the run object. Common issues: BYOM endpoint unreachable, model timeout. |
| `401 Unauthorized` | Token expired. Reacquire: `TOKEN=$(az account get-access-token ...)` |
| `403 Forbidden` | Missing `EdgeRAGDeveloper` role on token. Check Entra ID app role assignments. |
| `404 Not Found` on thread/message | Thread ownership is enforced; you can only access threads you created. |
| Ingestion job stuck in `PENDING` | Check ingestion service logs. Verify NFS path is accessible from the cluster. |

## Next step

> [!div class="nextstepaction"]
> [Add more knowledge sources](add-data-source.md)

## Related articles

- [Agentic layer overview](agentic-overview.md)
- [MCP server in Agents and Tools with Foundry Local](mcp-server-overview.md)
<!-- - [Collections overview](collections-overview.md)
- [Knowledge Base Manager API reference](APIs/knowledge-base-manager-api.md)
- [Agents Runtime API reference](APIs/agents-runtime-api.md) -->
