---
title: Manage the knowledge base (deprecated) - Agents and Tools with Foundry Local
description: Learn how to manage the default knowledge base with the deprecated API in Agents and Tools with Foundry Local.
author: Lea1702
ms.author: leabenero
ms.topic: how-to
ms.date: 10/29/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Manage the knowledge base (deprecated)

> [!WARNING]
> The earlier `/knowledge-bases` API is retired and returns `410 Gone` in the current extension. Use [Manage knowledge bases](knowledge-bases-guide.md) and the [Knowledge Bases REST API reference](reference/knowledge-bases-api-reference.md).

Earlier extension versions automatically provisioned one default knowledge base and paired it with the default agent. The current API uses named knowledge bases, named knowledge sources, model connections, and ETag-protected replacement.

## Migrate to the current API

| Earlier API | Current API |
|---|---|
| Default knowledge base identified by an agent ID | Named knowledge base at `/knowledgebases('<name>')` |
| `knowledge_source_ids` containing UUIDs | `knowledgeSources` containing source names |
| Partial PATCH updates | Full PUT replacement with `If-Match` or `If-None-Match` |
| Threads, messages, and runs | Conversations and Responses APIs |
| Implicit paired agent | Managed `kb-<knowledge-base-name>` connection added to an agent version |

Don't use the retired procedures against a current deployment. Create current knowledge sources, create a named knowledge base, and add its managed connection to an agent version.

## Related content

- [Configure a knowledge source](knowledge-sources-guide.md)
- [Manage knowledge bases](knowledge-bases-guide.md)
- [Knowledge Bases REST API reference](reference/knowledge-bases-api-reference.md)
- [Add tools to an agent](add-agent-tools.md)
