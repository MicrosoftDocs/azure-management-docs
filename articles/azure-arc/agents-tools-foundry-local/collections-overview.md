---
title: Collections in Agents and Tools with Foundry Local
description: Learn how collections organize indexed content, metadata, and access in Agentic Retrieval for Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/08/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-generated
---

# Collections in Agents and Tools with Foundry Local

Collections are the fundamental unit of data organization for indexed sources in Agentic Retrieval, the retrieval and grounding component of Agents and Tools with Foundry Local. Each collection is a logical grouping that contains ingested documents stored as vector embeddings and metadata. This article explains how collections work, how they're organized internally, and when to use multiple collections in your solution.

## What are collections?

A collection is a named container for vector data. When you ingest documents, you parse, chunk, embed, and store them in a specific collection. When you query, you specify which collections to search.

Collections provide:

- **Data separation**: Keep datasets for different use cases or departments in separate collections.
- **App-role collection access**: Filter collections for callers who use app-role fallback. Azure-role-authorized callers can access every collection.
- **Independent lifecycle**: Create, ingest, query, and delete collections independently.

Each collection maps to *up to four Milvus vector collections* and associated *Postgres tables*. When image embedding is available (GPU mode), you create all four collections. In CPU-only mode, you provision only the two text collections.

The system replaces hyphens in collection names with underscores to form the internal `storage_prefix`. For example, collection `my-docs` uses storage prefix `my_docs`.

## Collection lifecycle

The collection lifecycle follows these steps:

**Create → Ingest → Query → Update → Delete**

| Stage | API | Description |
|---|---|---|
| **Create** | `POST /collections` | Creates the collection and provisions up to four Milvus collections (four in GPU mode, two in CPU-only mode). |
| **Ingest** | `POST /ingestion/jobs/{job_id}` | Ingests documents into the collection. Specify `collectionName` in the request body. |
| **Query** | `POST /openai/v1/responses`, `POST /mcp`, or `POST /chat/completions` | External `POST /chat/completions` is deprecated and restricted to the developer portal (which queries `edgeragapp`). External or agentic queries over collections use an `indexed_sources_mcp` knowledge source via `/openai/v1/responses` or `/mcp`. |
| **Update** | `PATCH /collections/{name}` | Updates the collection description (name is immutable). |
| **Delete** | `DELETE /collections/{name}` | Deletes the collection and all associated Milvus collections, Postgres tables, and metadata. |

## Default collection

The Agents and Tools with Foundry Local extension automatically creates a default collection named `edgeragapp` on startup. This collection:

- Is used when you don't specify a `collectionName` during ingestion or querying.
- Can't be deleted; returns `409 Conflict`.
- Requires an `edgeragapp` app role for end users only when a query uses app-role fallback. Azure-role-authorized queries don't need a collection app role.

## Collection naming rules

- Use only lowercase letters, digits, and hyphens.
- Must start with a lowercase letter and end with a lowercase letter or digit.
- 2–49 characters.
- Names `default`, `system`, `edgeragenduser`, and `edgeragdeveloper` are reserved.
- The default collection `edgeragapp` is automatically created and can't be deleted.

## Collections and RBAC

Azure role-based access control (Azure RBAC) checks the caller's permissions at the extension resource scope first. **Reader** allows collection reads. **Cognitive Services Contributor** allows creation, changes, and deletion. **Cognitive Services OpenAI User** allows classic chat and search. In disconnected deployments, use **Reader** for reads and **Contributor** for changes, chat, and search.

Callers allowed by an Azure role can access **every collection**. Azure RBAC doesn't apply a per-collection filter. Don't use collection app roles as an isolation boundary for these callers.

If the Azure role check doesn't allow a request, supported classic APIs check app roles as a fallback. This *app-role collection access* uses roles from the caller's JSON Web Token (JWT):

| Role | Access level |
|---|---|
| `EdgeRAGDeveloper` | Fallback management and query access to *all* collections. |
| `EdgeRAGEndUser` | Access only to collections where the user has a matching app role (for example, role `finance-docs` grants access to collection `finance-docs`). |

When app-role fallback allows the route, an `EdgeRAGEndUser` caller without a matching collection role receives `403 Forbidden` for the query. Assign matching app roles when you use this fallback. App roles don't allow Responses or Conversations; running agents requires an Azure role.

For step-by-step instructions on creating collection-specific app roles, see [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access).

> [!NOTE]
> Classic collection and inference services skip authentication and collection checks for internal calls without the ingress external-request marker, including direct port-forwarded calls. This compatibility behavior doesn't apply to protected Agent Service APIs. Port forwarding doesn't remove their authentication requirement. Limit direct cluster access to trusted administrators.

## Collections and knowledge sources

Collections connect to Agent Service through knowledge sources:

- The built-in MCP server exposes search tools that query collections.
- A knowledge source with `indexed_source_ref` set to a collection name maps an MCP search tool to a specific collection.
- Knowledge sources are grouped into *knowledge bases*, which are assigned to *agents*.

**Agent → Knowledge base → Knowledge source (indexed_source_ref = "my-docs") → MCP server → Collection "my-docs"**

The `indexed_source_ref` field on a knowledge source refers to a *collection name* when pointing to the built-in MCP server. This field is the bridge between agents in Agent Service and the indexed content in Agentic Retrieval.

## When to use multiple collections

The following scenarios describe which collections to use:

| Scenario | Recommendation |
|---|---|
| **Single dataset, single team** | Use the default `edgeragapp` collection. |
| **Department-scoped data** | Create one collection per department (for example, `engineering-docs`, `hr-policies`). App-role filters apply only to fallback callers. |
| **Multitenant** | Don't use collections as a tenant access boundary for Azure-role-authorized callers. Their access isn't filtered by collection. |
| **Versioned datasets** | Create time-stamped collections (for example, `catalog-2026-q1`). Delete old collections when no longer needed. |
| **Mixed confidentiality** | Separate datasets, but don't rely on collection app roles to restrict Azure-role-authorized callers. |

## Related content

- [Collections API reference](reference/collections-api-reference.md)
- [Ingestion API reference](reference/ingestion-api-reference.md)
- [Inference API reference](reference/inference-api-reference.md)
- [Agent Service overview](agentic-overview.md)
