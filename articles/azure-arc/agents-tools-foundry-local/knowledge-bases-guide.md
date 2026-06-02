---
title: Manage the Knowledge Base in Agentic Retrieval in Foundry Local
description: Learn how to manage the default knowledge base in Agentic Retrieval in Foundry Local, which groups knowledge sources and defines what data the system can access.
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 04/30/2026
ms.service: azure-arc
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Manage the knowledge base in Agentic Retrieval in Foundry Local

This article explains how to manage your knowledge base in Agentic Retrieval. A *knowledge base* is a configuration and boundary object that groups one or more *knowledge sources* together. It defines what data the system can access when processing user queries.

Each deployment includes a default knowledge base that is automatically provisioned. Changes to the knowledge base are automatically synced to its paired internal agent.

> [!NOTE]
> Each deployment includes a default knowledge base. You cannot create additional knowledge bases or delete the default one. Use GET, PATCH, or PUT to view and update the default knowledge base.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

- Deploy Agentic Retrieval in combined or agentic mode.

- Create at least one knowledge source. See [Configure a knowledge source](knowledge-sources-guide.md).

- Configure authentication. You need a bearer token with **EdgeRAGDeveloper** role for write operations.

  ```azurecli
  az account clear
  az login --tenant <your-tenant-id> --output none

  TOKEN=$(az account get-access-token \
    --resource "api://<app-registration-client-id>" \
    --query accessToken -o tsv)
  ```

## Step 1: Get your default knowledge base

To retrieve your default knowledge base, send a GET request:

```bash
curl https://<cluster-domain>/knowledge-bases?limit=1 \
  -H "Authorization: Bearer $TOKEN"
```

The response includes the knowledge base ID that you use as `<kb-id>` in subsequent steps.

## Step 2: Link knowledge sources

Knowledge sources can be linked to your knowledge base via PATCH. To add sources, use a PATCH request to update the `knowledge_source_ids` array.

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

## Step 3: Update the knowledge base

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

## Step 4: List knowledge bases

To retrieve a list of all knowledge bases, send a GET request to the knowledge bases endpoint:

```bash
curl https://<cluster-domain>/knowledge-bases?limit=10&order=desc \
  -H "Authorization: Bearer $TOKEN"
```

The response includes cursor-based pagination with `after` and `before` parameters for navigating large result sets.

## Best practices

1. **Group related sources**: Link all related knowledge sources into your default knowledge base (for example, product manuals and FAQs together).
1. **Start simple**: Begin with the built-in MCP server's search tools, then add external MCP servers as needed.
1. **Use metadata**: Tag your knowledge base with department, version, or environment for organization.
1. **Audit before PATCH**: Since PATCH replaces the entire `knowledge_source_ids` array, always read the current state first to avoid accidentally removing sources.

## Related content

- [Configure a knowledge source](knowledge-sources-guide.md)
- [The agentic layer in Agentic Retrieval](agentic-overview.md)
- [Quickstart: Query your data with Agentic Retrieval](quickstart-edge-rag.md)
<!-- - [Knowledge Base Manager API reference](APIs/knowledge-base-manager-api.md) -->