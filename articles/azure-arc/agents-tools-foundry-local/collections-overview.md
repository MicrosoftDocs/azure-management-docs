---
title: Collections Overview for Agents and Tools with Foundry Local
description: Learn about collections, the fundamental unit of data organization in Agents and Tools with Foundry Local's Knowledge Layer, including architecture, lifecycle, naming rules, and access control.
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 04/29/2026
ms.subservice: edge-rag
ai-usage: ai-generated
---

# Collections in Agents and Tools with Foundry Local

Collections are the fundamental unit of data organization in Agents and Tools with Foundry Local's knowledge layer. Each collection is a logical grouping that contains ingested documents stored as vector embeddings and metadata. This article explains how collections work, how they're organized internally, and when to use multiple collections in your solution.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## What are collections?

A collection is a named container for vector data. When you ingest documents, they're parsed, chunked, embedded, and stored in a specific collection. When you query, you specify which collections to search.

Collections provide:
- **Data isolation**: Separate datasets for different use cases, departments, or tenants.
- **Access control**: Role-based access control (RBAC) policies can be applied per collection.
- **Independent lifecycle**: Create, ingest, query, and delete collections independently.

Each collection maps to *up to four Milvus vector collections* and associated *Postgres tables*. When image embedding is available (GPU mode), all four are created; in CPU-only mode, only the two text collections are provisioned:

> [!NOTE]
> Hyphens in collection names are replaced with underscores to form the internal `storage_prefix`. For example, collection `my-docs` uses storage prefix `my_docs`.

## Collection lifecycle

The collection lifecycle follows these steps:

**Create → Ingest → Query → Update → Delete**

| Stage | API | Description |
|---|---|---|
| **Create** | `POST /edgeai/collections` | Creates the collection and provisions 4 Milvus collections + Postgres tables. |
| **Ingest** | `POST /edgeai/ingestion/jobs/{job_id}` | Ingests documents into the collection. Specify `collectionName` in the request body. |
| **Query** | `POST /edgeai/chat/completions` | Queries the collection using RAG. Specify `data_sources[0].parameters.index_name`. |
| **Update** | `PATCH /edgeai/collections/{name}` | Updates the collection description (name is immutable). |
| **Delete** | `DELETE /edgeai/collections/{name}` | Deletes the collection and all associated Milvus collections, Postgres tables, and metadata. |

## Default collection

Agents and Tools with Foundry Local autocreates a default collection named `edgeragapp` on startup. This collection:
- Is used when no `collectionName` is specified during ingestion or querying.
- Can't be deleted; returns `409 Conflict`.
- Requires an `edgeragapp` app role for end-user access through RBAC.

## Collection naming rules

- Lowercase letters, digits, and hyphens only
- Must start and end with an alphanumeric character
- 2–49 characters
- Names `default`, `system`, `edgeragenduser`, and `edgeragdeveloper` are reserved
- The default collection `edgeragapp` is autocreated and can't be deleted

## Collections and RBAC

When accessed through the external endpoint (ingress), collection access is controlled by JSON Web Token (JWT) roles:

| Role | Access level |
|---|---|
| `EdgeRAGDeveloper` | Full access to *all* collections. Required for management APIs (collections, ingestion). |
| `EdgeRAGEndUser` | Access only to collections where the user has a matching app role (for example, role `finance-docs` grants access to collection `finance-docs`). |

> [!IMPORTANT]
> If a user has the `EdgeRAGEndUser` role but no collection-specific role assignments, they receive `403 Forbidden` when querying any collection. Ensure users are assigned app roles matching the collection names they need to access.

*RBAC is bypassed* when using port-forwarding or internal Dapr calls (development/testing only).

## Collections and knowledge sources

Collections connect to the agentic layer through knowledge sources:

1. The built-in MCP server exposes search tools that query collections.
1. A knowledge source with `indexed_source_ref` set to a collection name maps an MCP search tool to a specific collection.
1. Knowledge sources are grouped into *knowledge bases*, which are assigned to *agents*.

**Agent → Knowledge base → Knowledge source (indexed_source_ref = "my-docs") → MCP server → Collection "my-docs"**

> [!NOTE]
> The `indexed_source_ref` field on a knowledge source refers to a *collection name* when pointing to the built-in MCP server. This is the bridge between the agentic layer and the knowledge layer.

## When to use multiple collections

The following scenarios describe which collections to use:

| Scenario | Recommendation |
|---|---|
| **Single dataset, single team** | Use the default `edgeragapp` collection. |
| **Department-scoped data** | Create one collection per department (for example, `engineering-docs`, `hr-policies`). Apply RBAC per collection. |
| **Multi-tenant** | Create one collection per tenant. Use RBAC to enforce tenant isolation. |
| **Versioned datasets** | Create time-stamped collections (for example, `catalog-2026-q1`). Delete old collections when no longer needed. |
| **Mixed confidentiality** | Separate public and confidential data into different collections with different RBAC policies. |

## Related content

<!--
- [Collections API reference](APIs/collections-api-reference.md)
- [Ingestion API reference](APIs/ingestion-api-reference.md)
- [Inference API reference](APIs/inference-api-reference.md)
-->
- [Agentic layer overview](agentic-overview.md)
