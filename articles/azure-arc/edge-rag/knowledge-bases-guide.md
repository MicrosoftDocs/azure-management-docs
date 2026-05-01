---
title: Create a Knowledge Base in Agentic RAG
description: Learn how to create and manage knowledge bases in Agentic RAG, which group knowledge sources for agent access.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 04/30/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Create a knowledge base in Agentic RAG

This article explains how to create and manage knowledge bases in Agentic RAG. A *knowledge base* is a configuration and boundary object that groups one or more *knowledge sources* together. It defines what knowledge an agent is allowed to access, decoupling knowledge configuration from agent logic.

An agent references a knowledge base; the knowledge base itself doesn't execute anything. Multiple agents can reuse the same knowledge base, and agents can be rebound to different knowledge bases per tenant, deployment, or region.

## Prerequisites

- Deploy Agentic RAG in combined or agentic mode.

- Create at least one knowledge source. See [Configure a knowledge source](knowledge-sources-guide.md).

- Configure authentication. You need a bearer token with **EdgeRAGDeveloper** role for write operations.

  ```azurecli
  az account clear
  az login --tenant <your-tenant-id> --output none

  TOKEN=$(az account get-access-token \
    --resource "api://<app-registration-client-id>" \
    --query accessToken -o tsv)
  ```

## Step 1: Create a knowledge base

To create a new knowledge base, send a POST request with the knowledge base configuration to the knowledge bases endpoint.

```bash
curl -X POST https://<cluster-domain>/knowledge-bases \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "Product Documentation KB",
    "description": "Knowledge base for product manuals and FAQs",
    "instructions": "You are a helpful product support assistant. Answer questions using the knowledge base. Always cite your sources.",
    "temperature": 0.7,
    "top_p": 0.9,
    "knowledge_source_ids": [
      "<knowledge-source-id-1>",
      "<knowledge-source-id-2>"
    ],
    "metadata": {
      "department": "engineering",
      "version": "2.0"
    }
  }'
```

You can verify the successful response (201 Created):

```json
{
  "data": {
    "id": "asst_abc123def456789012345678",
    "object": "knowledge_base",
    "created_at": "2026-04-13T10:30:00Z",
    "name": "Product Documentation KB",
    "description": "Knowledge base for product manuals and FAQs",
    "knowledge_source_ids": ["<ks-id-1>", "<ks-id-2>"],
    "metadata": { "department": "engineering", "version": "2.0" }
  }
}
```

### Request fields

The following table describes the fields you can include in the knowledge base request body:

| Field | Required | Description |
|---|---|---|
| `name` | **Yes** | Knowledge base name (1–256 characters, non-empty). |
| `description` | No | Description (max 512 characters). |
| `instructions` | No | System instructions that guide the knowledge base's response behavior. |
| `temperature` | No | Sampling temperature (0–2). Controls response randomness. Default: `1.0`. |
| `top_p` | No | Nucleus sampling (0–1). Controls response diversity. Default: `1.0`. |
| `knowledge_source_ids` | No | List of knowledge source IDs to associate. Default: `[]`. |
| `metadata` | No | Custom key-value pairs (max 16 pairs, 64-char keys, 512-char values). |

## Step 2: Link knowledge sources

Knowledge sources can be linked at creation time through `knowledge_source_ids` or added later by updating the knowledge base. To add sources to an existing knowledge base, use a PATCH request to update the `knowledge_source_ids` array.

1. Get the current knowledge source IDs:

    ```bash
    CURRENT=$(curl -s https://<cluster-domain>/knowledge-bases/<kb-id> \
      -H "Authorization: Bearer $TOKEN" | jq -r '.data.knowledge_source_ids')
    ```

1. Send a PATCH request with the updated list (include all desired IDs):

    ```bash
    curl -X PATCH https://<cluster-domain>/knowledge-bases/<kb-id> \
      -H "Content-Type: application/json" \
      -H "Authorization: Bearer $TOKEN" \
      -d '{
        "knowledge_source_ids": ["<existing-ks-id>", "<new-ks-id>"]
      }'
    ```

    > [!IMPORTANT]
    > PATCH replaces the entire `knowledge_source_ids` array. It doesn't append. You must include all desired IDs in the array. Omitting an existing ID effectively removes that knowledge source from the knowledge base.

## Step 3: Assign to an agent

Send a POST request to create an agent with the knowledge base:

```bash
curl -X POST https://<cluster-domain>/agents \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "Product Support Agent",
    "instructions": "You are a helpful product support agent. Answer questions using the knowledge base.",
    "endpoint_url": "https://my-model-endpoint.example.com/v1",
    "knowledge_base_id": "<kb-id>",
    "temperature": 0.7,
    "top_p": 0.9
  }'
```

The agent now has access to all knowledge sources in the linked knowledge base when processing queries. Multiple agents can reference the same knowledge base.

> [!NOTE]
> Currently, all agents use the cluster-level BYOM endpoint configured through Helm values (`byom.apiEndpoint`).

## Step 4: Update a knowledge base

You can update a knowledge base by using either PATCH (partial update) or PUT (full replace) requests.

| Operation | `knowledge_source_ids` behavior | Other fields |
|---|---|---|
| **PATCH** | Replaces the entire array (not append). Omitting the field keeps the existing list. | Only provided fields are updated. |
| **PUT** | Replaces the entire array. Omitting the field resets to `[]`. | All fields are overwritten. Omitted fields reset to defaults. |

### Partial update (PATCH)

Send a PATCH request with only the fields you want to update:

```bash
curl -X PATCH https://<cluster-domain>/knowledge-bases/<kb-id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "Updated Product KB",
    "description": "Updated description"
  }'
```

Only provided fields are updated. Omitted fields remain unchanged.

### Full replace (PUT)

Send a PUT request with all required fields:

```bash
curl -X PUT https://<cluster-domain>/knowledge-bases/<kb-id> \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "Replaced Product KB",
    "knowledge_source_ids": ["<new-ks-id>"]
  }'
```

All fields are overwritten. Fields not included reset to defaults.

## Step 5: List knowledge bases

To retrieve a list of all knowledge bases, send a GET request to the knowledge bases endpoint:

```bash
curl https://<cluster-domain>/knowledge-bases?limit=10&order=desc \
  -H "Authorization: Bearer $TOKEN"
```

The response includes cursor-based pagination with `after` and `before` parameters for navigating large result sets.

## Step 6: Delete a knowledge base

To remove a knowledge base, send a DELETE request:

```bash
curl -X DELETE https://<cluster-domain>/knowledge-bases/<kb-id> \
  -H "Authorization: Bearer $TOKEN"
```

> [!NOTE]
> Deleting a knowledge base doesn't delete the underlying knowledge sources. It only removes the association. Knowledge sources can be reused in other knowledge bases.

## Best practices

1. **One knowledge base per domain**: Group related knowledge sources into a single knowledge base (for example, *Engineering Docs knowledge base*, *HR Policies knowledge base*).
1. **Reuse knowledge sources**: The same knowledge source can be referenced by multiple knowledge bases.
1. **Start simple**: Create a knowledge base with the built-in MCP server's search tools, then add external MCP servers as needed.
1. **Use metadata**: Tag knowledge bases with department, version, or environment for organization.
1. **Audit before PATCH**: Since PATCH replaces the entire `knowledge_source_ids` array, always read the current state first to avoid accidentally removing sources.

## Related content

- [Configure a knowledge source](knowledge-sources-guide.md)
- [The agentic layer in Agentic RAG](agentic-overview.md)
- [Quickstart: Create your first Agentic RAG agent](create-agent-quickstart.md)
<!-- - [Agent manager API reference](APIs/agent-manager-api.md) -->