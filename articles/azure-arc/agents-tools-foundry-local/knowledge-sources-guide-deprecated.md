---
title: Configure a knowledge source (deprecated) - Agents and Tools with Foundry Local
description: Learn how to configure knowledge sources with the deprecated API in Agents and Tools with Foundry Local.
author: Lea1702
ms.author: leabenero
ms.topic: how-to
ms.date: 11/04/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Configure a knowledge source (deprecated)

> [!WARNING]
> The earlier Knowledge Sources API, which used generated IDs, is retired. The current API at `/knowledgesources` uses source names and a different request body. Use [Configure a knowledge source](knowledge-sources-guide.md) and the [Knowledge Sources REST API reference](reference/knowledge-sources-api-reference.md).

Earlier extension versions identified knowledge sources by generated UUIDs and exposed validation-specific response fields. The current API uses stable source names and current-generation provisioning status.

## Migrate to the current API

| Earlier API | Current API |
|---|---|
| `/edgeai/knowledgesources` | `/knowledgesources` |
| Generated UUID identifier | Caller-selected lowercase source name |
| UUID references in a knowledge base | Source-name references in `knowledgeSources` |
| `validation_status`, `validation_error`, and `validated_at` | `provisioning_state`, `generation`, and `observed_generation` |
| Earlier default knowledge base | Named knowledge bases and managed `kb-<name>` connections |

Don't use the retired create, update, or delete examples against a current deployment. Recreate each source through the current API, wait for it to reach `Succeeded`, and reference its name from a current knowledge base.

## Related content

- [Configure a knowledge source](knowledge-sources-guide.md)
- [Knowledge Sources REST API reference](reference/knowledge-sources-api-reference.md)
- [Manage knowledge bases](knowledge-bases-guide.md)
- [Indexed Sources MCP overview](mcp-server-overview.md)
