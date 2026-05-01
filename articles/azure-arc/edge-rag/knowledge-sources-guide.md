---
title: Configure a Knowledge Source in Agentic RAG
description: Learn how to configure knowledge sources in Agentic RAG, which define MCP server connections for agent access.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 04/30/2026
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Configure a knowledge source in Agentic RAG

This guide explains how to configure knowledge sources in Agentic RAG. A *knowledge source* is a self-contained registration of an MCP server connection, optionally bound to a specific indexed source reference. Each knowledge source includes all MCP server connection details (URL, auth type). 

## Prerequisites

- Deploy Agentic RAG in combined or agentic mode.

- MCP server. Either the built-in Agentic RAG MCP server or an external one.

- Bearer token with **EdgeRAGDeveloper** role for write operations.

  ```azurecli
  TOKEN=$(az account get-access-token \
    --resource "api://<app-registration-client-id>" \
    --query accessToken -o tsv)
  ```

## Knowledge source kinds

The following table describes the two types of knowledge sources you can create:

| Kind | Parameters field | Use case |
|---|---|---|
| `remote_mcp` | `remote_mcp_parameters` | Register any external MCP server. |
| `indexed_sources_mcp` | `indexed_sources_parameters` | Register the built-in `indexed-source-mcp` server pointing at a specific indexed source (for example, a collection). |

## Step 1: Create a remote MCP knowledge source

To register an external MCP server directly as a knowledge source, send a POST request with the server connection details:

```bash
curl -X POST https://<cluster-domain>/edgeai/knowledgesources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "my-external-mcp",
    "kind": "remote_mcp",
    "auth_type": "unauthenticated",
    "description": "External MCP server for document search",
    "remote_mcp_parameters": {
      "server_url": "https://my-mcp-server.example.com/mcp",
      "server_label": "doc-search"
    }
  }'
```

You can verify the successful response (201 Created):

```json
{
  "id": "f1e2d3c4-b5a6-7890-abcd-ef1234567890",
  "name": "my-external-mcp",
  "kind": "remote_mcp",
  "description": "External MCP server for document search",
  "auth_type": "unauthenticated",
  "remote_mcp_parameters": {
    "server_url": "https://my-mcp-server.example.com/mcp",
    "server_label": "doc-search"
  },
  "indexed_sources_parameters": null,
  "validation_status": "active",
  "validation_error": null,
  "validated_at": "2026-03-29T12:00:00.000000+00:00",
  "created_by": null,
  "created_at": "2026-03-29T12:00:00.000000+00:00",
  "updated_at": "2026-03-29T12:00:00.000000+00:00"
}
```

### `remote_mcp_parameters` fields

The following table describes the parameters for remote MCP connections:

| Field | Required | Description |
|---|---|---|
| `server_url` | **Yes** | URL of the MCP server endpoint |
| `server_label` | No | Human-readable label for the server |

## Step 2: Create an indexed source MCP knowledge source

To register the built-in MCP server pointing at a specific indexed source (collection), send a POST request with the indexed source connection details:

```bash
curl -X POST https://<cluster-domain>/edgeai/knowledgesources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "my-docs-search",
    "kind": "indexed_sources_mcp",
    "auth_type": "microsoft_entra_id",
    "description": "Search the my-docs collection",
    "indexed_sources_parameters": {
      "server_url": "https://<cluster-domain>/edgeai/mcp",
      "indexed_source_ref": "my-docs",
      "server_label": "indexed-sources-mcp-server"
    }
  }'
```

### `indexed_sources_parameters` fields

The following table describes the parameters for indexed source connections:

| Field | Required | Description |
|---|---|---|
| `server_url` | **Yes** | URL of the indexed-source-mcp server endpoint. |
| `indexed_source_ref` | **Yes** | The name of the indexed source (collection) to connect to. |
| `server_label` | No | Human-readable label. Defaults to `"indexed-sources-mcp-server"`. |

> [!IMPORTANT]
> When using the built-in MCP server's search tools, `indexed_source_ref` refers to a collection name (for example, `"my-docs"`). This is the link between the Agentic Layer and the knowledge layer's collections.

## MCP validation and authentication

When a knowledge source is created or its connection parameters are updated, the service validates the MCP connection before persisting:

- Extracts `server_url` from the parameters.
- Performs an SSRF/DNS safety check (blocks private/internal IPs).
- Sends a JSON-RPC `initialize` request to the MCP server (connect timeout: 5s, read timeout: 30s).
- Retries up to three times with exponential backoff (1s, 2s).
- If validation *succeeds*: `validation_status` is set to `active`, the record is persisted.
- If validation *fails*: the request is rejected with `400 Bad Request`. Nothing is persisted (on create) or changed (on update).

| Status | Description |
|---|---|
| `unknown` | Default status. Has not been validated. |
| `active` | MCP `initialize` succeeded. `validated_at` is set. |

> [!NOTE]
> If validation fails on create, the request is rejected with `400 Bad Request` and no record is persisted. If validation fails on update, no fields are changed. In both cases, the error response contains the validation failure details.

### Supported authentication types

The following table describes the supported authentication types for knowledge sources:

| Auth type | Description | When to use |
|---|---|---|
| `microsoft_entra_id` | Forwards the caller's `Authorization` header to the MCP server. | When the MCP server validates Entra ID tokens. |
| `unauthenticated` | No auth headers are forwarded to the MCP server. | When the MCP server has no auth requirement (for example, internal service). |

## Step 3: Update a knowledge source

To modify an existing knowledge source, send a PATCH request with the fields you want to update. If connection parameters or `auth_type` are changed, MCP validation is re-run. If revalidation fails, no fields are updated.

Send a PATCH request with the fields to update:

```bash
curl -X PATCH https://<cluster-domain>/edgeai/knowledgesources/<ks-id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "updated-name",
    "description": "Updated description"
  }'
```

> [!IMPORTANT]
> The `kind` field is immutable after creation. Supplying a parameters field that doesn't match the knowledge source's `kind` returns `400 Bad Request`.

## Step 4: Delete knowledge sources

You can delete knowledge sources individually or in batch operations.

1. Delete a single knowledge source.

    ```bash
    curl -X DELETE https://<cluster-domain>/edgeai/knowledgesources/<ks-id> \
      -H "Authorization: Bearer $TOKEN"
    ```

1. Batch delete multiple knowledge sources.

    ```bash
    curl -X POST https://<cluster-domain>/edgeai/knowledgesources/batch-delete \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d '{
        "ks_ids": ["<ks-id-1>", "<ks-id-2>"]
      }'
    ```

## Step 5: Complete an end-to-end example

Follow this complete flow to create a knowledge source, knowledge base, and agent together.

1. Create a knowledge source for your collection.

    ```bash
    KS_RESPONSE=$(curl -s -X POST https://<cluster-domain>/edgeai/knowledgesources \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d '{
        "name": "my-docs-search",
        "kind": "indexed_sources_mcp",
        "auth_type": "microsoft_entra_id",
        "description": "Search the my-docs collection",
        "indexed_sources_parameters": {
          "server_url": "https://<cluster-domain>/edgeai/mcp",
          "indexed_source_ref": "my-docs",
          "server_label": "indexed-sources-mcp-server"
          }
        }
      }')

    KS_ID=$(echo $KS_RESPONSE | jq -r '.id')
    echo "Knowledge Source ID: $KS_ID"
    ```

1. Create a knowledge base.

    ```bash
    KB_RESPONSE=$(curl -s -X POST https://<cluster-domain>/knowledge-bases \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d "{
        \"name\": \"Product Docs KB\",
        \"description\": \"Knowledge base for product documentation\",
        \"knowledge_source_ids\": [\"$KS_ID\"]
      }")

    KB_ID=$(echo $KB_RESPONSE | jq -r '.data.id')
    echo "Knowledge Base ID: $KB_ID"
    ```

1. Create an agent that uses the knowledge base.

    ```bash
    curl -X POST https://<cluster-domain>/agents \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d "{
        \"name\": \"Product Support Agent\",
        \"instructions\": \"You are a helpful agent. Answer questions using your knowledge base.\",
        \"endpoint_url\": \"https://my-model.example.com/v1\",
        \"knowledge_base_id\": \"$KB_ID\",
        \"temperature\": 0.7,
        \"top_p\": 0.9
      }"
    ```

The agent is now ready to use. To create threads, send messages, and execute runs, see [Quickstart: Create your first agent](create-agent-quickstart.md).

## Related content

- [Create a knowledge base in Agentic RAG](knowledge-bases-guide.md)
- [Agentic layer overview](agentic-overview.md)
- [MCP server in Agentic RAG](mcp-server-overview.md)
<!-- - [Knowledge sources API reference](APIs/KNOWLEDGE-SOURCES-API-REFERENCE.md) -->
