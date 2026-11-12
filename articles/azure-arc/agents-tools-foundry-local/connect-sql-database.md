---
title: Connect a SQL database as a knowledge source in Agents and Tools with Foundry Local
description: "Learn how to connect a read-only SQL knowledge source, use its retrieval and aggregation tools, preview and update schemas, and rotate credentials."
author: Lea1702
ms.author: leabenero
ms.topic: how-to
ms.date: 11/05/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
ms.custom:
  - sfi-ropc-nochange
#CustomerIntent: As a developer, I want to expose a read-only slice of a SQL database to an agent so that it can answer questions from live business data without being able to change or over-read it.
---

# Connect a SQL database as a knowledge source

A `sql` knowledge source exposes a read-only slice of a SQL database to an agent. You choose which tables and columns to expose, and Agents and Tools with Foundry Local manages a [Data API builder (DAB) SQL MCP server](/azure/data-api-builder/mcp/overview) for the source.

Follow this workflow to preview the schema, select the exposed data, create the source, and associate it with a knowledge base. For lookup and aggregation capabilities, see [SQL tools by engine](#sql-tools-by-engine). For request fields and SQL-only endpoints, see [SQL API reference](#sql-api-reference).

[!INCLUDE [sql-knowledge-source-preview-notice](includes/sql-knowledge-source-preview-notice.md)]

## Prerequisites

- Deploy Agents and Tools with Foundry Local and configure authentication for the Knowledge Sources API.

- A reachable SQL Server, PostgreSQL, or MySQL database.

- TLS enabled on the database, with a server certificate whose chain is trusted by both the database validator and the SQL serving workload. The hostname in the connection string must match the certificate.

- A database credential that can read every table you plan to expose, and nothing more. Create a dedicated read-only login.

- A Microsoft Entra ID access token for the extension's app registration. Assign **Cognitive Services Contributor** at the extension scope for schema previews, creation, and changes, or **Reader** for read-only operations. In disconnected deployments, use **Contributor** for changes and **Reader** for reads. The `EdgeRAGDeveloper` app role also works as a fallback. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

  ```azurecli
  az account clear
  az login --tenant <your-tenant-id> --output none

  TOKEN=$(az account get-access-token \
    --resource "api://<app-registration-client-id>" \
    --query accessToken -o tsv)
  ```

## Prepare a connection string

Use the following connection-string forms with full certificate verification. Replace the host, database, read-only user, and password with your database values, and change the port if needed. Submit the completed string in `connection_string`; don't include it in logs or source control.

For SQL Server (`mssql`), use:

```text
Server=tcp:<database-host>,1433;Database=<database>;User ID=<read-only-user>;Password=<password>;Encrypt=True;TrustServerCertificate=False
```

For PostgreSQL (`postgresql`), use:

```text
Host=<database-host>;Port=5432;Database=<database>;Username=<read-only-user>;Password=<password>;SSL Mode=VerifyFull
```

For MySQL (`mysql`), use:

```text
Server=<database-host>;Port=3306;Database=<database>;User ID=<read-only-user>;Password=<password>;SslMode=VerifyFull
```

The API accepts a supported set of connection-string options, not every option a database driver offers. Unsupported options return `400`. Don't add driver-specific certificate-file options such as PostgreSQL `RootCertificate` or MySQL `SslCa` as a workaround for missing deployment trust.

> [!IMPORTANT]
> A connection string requests verified TLS; it doesn't install certificate trust. For a database that uses a private CA, have your deployment administrator confirm that the deployed release supports that CA and that **both** the validator and the generated serving workload trust its certificate chain. Making a CA available only to validation isn't enough. These API routes don't configure CA trust, and you shouldn't edit the generated workloads or disable certificate verification to work around a trust failure. If trust isn't configured, resolve that prerequisite before creating a source.

## How the workflow fits together

Creating a SQL knowledge source isn't a single call. The API separates *discovering* what's in your database from *deciding* what an agent can see, so that the decision is explicit and reviewable.

| Step | Call | What happens |
|---|---|---|
| 1 | `POST /knowledgesources/schema-preview` | Reads the database and returns its tables and columns. Creates nothing. |
| 2 | *(you)* | Trim the returned list to the minimum an agent needs. Describe each table and column. |
| 3 | `POST /knowledgesources` with `kind: sql` | Validates the credential and the trimmed list, then creates the knowledge source. |
| 4 | *(response)* | `202 Accepted`. The knowledge source isn't usable yet. |
| 5 | `GET /knowledgesources/{name}` | Poll until `provisioning_state` is `Succeeded` or `Failed`. |

## Step 1: Preview the database schema

Send the engine, a write-only connection string, a database label, and optionally a list of schemas to read.

The example uses SQL Server. For PostgreSQL, use `engine: "postgresql"` and `include_schemas: ["public"]`. For MySQL, use `engine: "mysql"` and omit `include_schemas`.

```bash
curl -X POST https://<cluster-domain>/knowledgesources/schema-preview \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "engine": "mssql",
    "connection_string": "<connection-string>",
    "database_label": "sales",
    "include_schemas": ["dbo"]
  }'
```

This call creates nothing. It connects, reads the catalog, and returns what it found:

```json
{
  "schema_spec": {
    "engine": "mssql",
    "database_label": "sales",
    "entities": [
      {
        "name": "Orders",
        "source_object": "Orders",
        "source_type": "table",
        "description": "",
        "fields": [
          { "name": "OrderId", "data_type": "int", "description": "int", "alias": null, "is_primary_key": true },
          { "name": "CustomerId", "data_type": "int", "description": "int", "alias": null, "is_primary_key": false },
          { "name": "OrderDate", "data_type": "datetime", "description": "datetime", "alias": null, "is_primary_key": false },
          { "name": "Total", "data_type": "decimal", "description": "decimal", "alias": null, "is_primary_key": false },
          { "name": "InternalNotes", "data_type": "nvarchar", "description": "nvarchar", "alias": null, "is_primary_key": false }
        ]
      }
    ]
  },
  "discovered_entity_count": 1,
  "truncated": false
}
```

If `truncated` is `true`, the database holds more tables than the response can carry. `discovered_entity_count` tells you how many it found. Narrow the read with `include_schemas` and preview again.

Keep the exact `source_object` returned by preview. SQL Server's default `dbo` schema and PostgreSQL's default `public` schema return bare object names, such as `Orders`. MySQL also returns bare object names. Objects in non-default SQL Server or PostgreSQL schemas retain their schema prefix. Don't add a default-schema prefix yourself: create validation compares these names with the discovered catalog.

## Step 2: Trim the schema and describe it

Take `schema_spec` from the preview and cut it down. Two things matter here:

- **Remove everything the agent doesn't need.** The exposed set is the agent's entire view of the database. In the preceding example, `InternalNotes` is dropped.
- **Write a real `description` on every table and column you keep.** The preview seeds `description` with the column's data type, which tells an agent nothing. A description is the only field that can't be derived from the database, and it's what stops the agent from guessing what a column means.

```json
{
  "name": "Orders",
  "source_object": "Orders",
  "source_type": "table",
  "description": "One row per customer order. Doesn't include cancelled orders.",
  "fields": [
    { "name": "OrderId", "description": "Unique order identifier.", "is_primary_key": true },
    { "name": "CustomerId", "description": "Identifier of the customer who placed the order." },
    { "name": "OrderDate", "description": "Date the order was placed, in UTC." },
    { "name": "Total", "description": "Order total in US dollars, including tax." }
  ]
}
```

You can expose up to 200 tables or views, with up to 500 columns each.

> [!IMPORTANT]
> Keep a nonempty `fields` list to restrict the exposed columns. Omitting `fields` or sending an empty list permits reads of all columns the database principal can access. Use an explicit column allow list and a least-privilege database credential together.

## Step 3: Create the SQL knowledge source

Send the trimmed entity list with the credential. You must include `auth_type` even for SQL sources. Use `unauthenticated` as shown; the database authenticates by using the connection string, not the caller's Microsoft Entra token. This source setting doesn't grant access to the management API.

```bash
curl -X POST https://<cluster-domain>/knowledgesources \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "name": "northwind-orders",
    "kind": "sql",
    "auth_type": "unauthenticated",
    "description": "Order data from the sales database",
    "sql_parameters": {
      "engine": "mssql",
      "database_label": "sales",
      "connection_string": "<connection-string>",
      "entities": [
        {
          "name": "Orders",
          "source_object": "Orders",
          "source_type": "table",
          "description": "One row per customer order. Doesn'\''t include cancelled orders.",
          "fields": [
            { "name": "OrderId", "description": "Unique order identifier.", "is_primary_key": true },
            { "name": "OrderDate", "description": "Date the order was placed, in UTC." },
            { "name": "Total", "description": "Order total in US dollars, including tax." }
          ]
        }
      ]
    }
  }'
```

The knowledge source name uses lowercase letters, digits, and hyphens, up to 40 characters, starting and ending with a letter or digit. Invalid names are rejected, not rewritten, and there's no rename.

## Step 4: Receive 202 Accepted

A successful create returns `202`, not `201`:

```json
{
  "name": "northwind-orders",
  "kind": "sql",
  "description": "Order data from the sales database",
  "auth_type": "unauthenticated",
  "remote_mcp_parameters": null,
  "indexed_sources_parameters": null,
  "provisioning_state": "Provisioning",
  "created_by": "user@contoso.com",
  "created_at": "2026-09-08T12:00:00Z"
}
```

The database was validated inside the request, but the operator still has to build and handshake the workload that serves it. **The knowledge source isn't usable yet.** Don't attach it to a knowledge base until it reports `Succeeded`.

## Step 5: Poll until the knowledge source is ready

```bash
curl https://<cluster-domain>/knowledgesources/northwind-orders \
  -H "Authorization: Bearer $TOKEN"
```

Poll until `provisioning_state` reaches a terminal value.

| `provisioning_state` | Meaning | What to do |
|---|---|---|
| `Provisioning` | Asynchronous work or recovery is in progress. | Keep polling. |
| `Succeeded` | The current generation was successfully handshaked. | Attach the knowledge source to a knowledge base. |
| `Failed` | Terminal. The attempted generation needs intervention. | Check the resource's `status.message`, fix the cause, and retry. |

Managed knowledge base execution normally requires a current-generation `Succeeded` source. During a validated credential-only rotation of a ready SQL source, the previous ready generation can remain available while the API reports `Provisioning`. Schema changes don't have this exception.

`provisioning_state` is already current-generation aware. When you change a SQL knowledge source, the API reports `Provisioning` until the work for *that* change completes, so a `Succeeded` left over from an earlier generation is never reported as though it described your edit. You don't need to compare generation numbers yourself. If you inspect the custom resource directly with `kubectl`, compare `status.observedGeneration` to `metadata.generation`.

### If validation succeeds but serving fails

A successful schema preview or `202` response confirms database validation, not serving-workload readiness. If the source reaches `Failed`, inspect its message:

```bash
kubectl get ks northwind-orders -n arc-rag \
  -o jsonpath='{.status.provisioningState}{"\n"}{.status.message}{"\n"}'
```

Have your deployment administrator inspect the source's serving-workload logs. Errors such as `PartialChain`, `unable to get local issuer certificate`, or an SSL handshake failure can indicate that the serving database client doesn't trust the certificate chain, even when validation does. Resolve the [certificate prerequisites](#prepare-a-connection-string), then retry through the API. Keep the source out of a knowledge base until it reports `Succeeded`.

## Step 6: Attach the knowledge source to a knowledge base

[Create a named knowledge base](knowledge-bases-guide.md), or replace an existing knowledge base definition by using PUT. Add `northwind-orders` to the complete `knowledgeSources` array, and include every other required knowledge base field.

The Knowledge Bases API doesn't support PATCH. For the full request body and ETag replacement workflow, see [Update a knowledge base](knowledge-bases-guide.md#update-a-knowledge-base).

## Security and lifecycle contracts

SQL creation validates the credential, checks that it can read every selected table, and verifies that the selected schema renders a servable read-only configuration before creating the source. The following contracts also apply to SQL preview and maintenance, as indicated.

| Contract | Behavior |
|---|---|
| Write-only credentials | No route returns the connection string. Preview discards it after the request; create and rotation store it in a source-owned Kubernetes Secret. |
| Verified TLS | Preview, create, and rotation reject connection strings that disable TLS or certificate verification. Both validation and serving must trust the certificate chain. |
| Address restrictions | Database destinations can't be loopback, link-local, multicast, reserved, or otherwise blocked addresses. Preview, create, and rotation check the destination. |
| Read-only exposure | Use a least-privilege database principal and explicit, nonempty field lists. Omitting fields doesn't restrict column access. |
| Credential validation | Create and rotation verify access to every exposed table before changing resources. |
| Failed create validation | Returns `400` without leaving a partial source or credential Secret. Failed rotation validation leaves the existing source unchanged. |

### Capacity limits return 429

| Cause | Status |
|---|---|
| The maximum number of SQL knowledge sources already exist (default 25). | `429` |
| Too many concurrent database reads are already in progress. | `429` |

A `429` status means "not now," not "your request was wrong." Retry after a delay.

### An unavailable engine driver returns 500

If the driver for the requested engine isn't available in the deployed image, the request returns `500`. This status code indicates a service fault, not a problem with your request.

### Response codes and readiness

SQL creation and the dedicated schema and credential `PUT` routes return `202`, because the operator must roll and handshake the serving workload before the change takes effect.

`PATCH /knowledgesources/{name}` returns `200 OK`, with `provisioning_state` set to `"Provisioning"` in the response body when `sql_parameters` changes, requiring `GET /knowledgesources/{name}` polling for readiness. In contrast, `PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`.

Wait for an in-progress operation to finish before submitting another schema or credential change. If a request returns `409 Conflict` because the source was modified concurrently, read the current state and retry after the previous operation finishes.

### Deletion cleans up managed resources

Deleting a SQL knowledge source returns `204`; garbage collection of its credential Secret, rendered configuration, and serving workload can continue afterward. It doesn't delete the database or its data. Don't hand-manage those objects or edit generated DAB configuration.

## SQL tools by engine

Before you use the SQL tools, ensure the SQL source is ready and associated with a knowledge base. An agent configured with the managed `kb-<knowledge-base-name>` connection can use the source's tools through knowledge base retrieval. **The effective toolset depends on the database engine:** it includes native DAB tools and, for PostgreSQL and MySQL, an aggregation tool added by Knowledge Base Execution.

| Tool base name | SQL Server (`mssql`) | PostgreSQL | MySQL |
|---|---|---|---|
| `describe_entities` | Native DAB discovery. | Native DAB discovery. | Native DAB discovery. |
| `read_records` | Native DAB reads. | Native DAB reads. | Native DAB reads. |
| `aggregate_records` | Native database aggregation. | Not provided natively. | Not provided natively. |
| `aggregate_read_records` | Not added for this engine. | Added by the execution layer. | Added by the execution layer. |

The runtime discovers native tools and applies its tool-exposure policy before presenting them to the agent. Use the actual presented tool names and argument schemas. The table distinguishes supported capabilities, not a guarantee that every native discovery tool is presented in every execution context.

Use `read_records` for row lookups and the available aggregation tool for counts, totals, averages, minima, maxima, and supported grouping or distinct operations. Don't have the model calculate totals from the first page or manually combine pages.

On SQL Server, native `aggregate_records` supports `count`, `sum`, `avg`, `min`, and `max` and performs the aggregation in the database.

Neither aggregation tool supports joins or arbitrary calculations. Expose a database view when the question requires a supported read over joined or calculated data.

### Aggregation for PostgreSQL and MySQL

For a managed PostgreSQL or MySQL source, Knowledge Base Execution automatically adds `aggregate_read_records` over the discovered DAB `read_records` tool and the source's schema metadata. You don't enable it through a separate public setting or install another MCP server.

The helper reads all matching rows within its limits and calculates the result in the execution layer, not in the language model. It isn't a native DAB tool: inspecting `tools/list` directly on the SQL serving workload won't show this addition.

Its structured arguments differ from the native SQL Server aggregation tool:

| Argument | Required | Type | Description |
|---|---|---|---|
| `entity` | Yes | String | One entity in the configured SQL schema. |
| `function` | Yes | String | `count`, `sum`, `avg`, `min`, or `max`. |
| `field` | Yes | String | An exposed field. Use `*` only for `count`; distinct counts require a field name. |
| `filter` | No | String | A nonempty OData filter, up to 8,192 characters. |
| `distinct` | No | Boolean | Aggregate distinct field values. Defaults to `false`. |
| `groupby` | No | Array of strings | Up to eight unique exposed field names. |

`sum`, `avg`, `min`, and `max` require numeric values. Configure explicit fields and primary keys for the entity so the helper can validate field access and paginate safely. Use exact, case-sensitive entity and exposed field names, including field aliases when configured.

Successful numeric aggregate values can be decimal strings rather than JSON numbers.

The helper accepts one entity, not arbitrary SQL, joins, expressions, or multiple entities. It must complete the matching-record scan within these limits:

| Budget | Limit |
|---|---|
| Pages scanned | 100 |
| Rows scanned | 10,000 |
| Input size | 10 MiB |
| Total execution time | 30 seconds |
| Local aggregation query time | 2 seconds |
| `groupby` fields | 8 |
| Filter length | 8,192 characters |
| Output rows | 1,000 |
| Output size | 1 MiB |

If the helper can't complete the scan or exceeds a limit, it returns an explicit error instead of a partial aggregate. A question that succeeds with native SQL Server aggregation can exceed the helper's scan budget on PostgreSQL or MySQL.

The 100-page limit counts actual `read_records` page reads. If the scan is still incomplete after 100 page reads, the helper returns:

```json
{
  "code": "sql_aggregation_failed",
  "error": "SQL aggregation page limit prevented a complete scan."
}
```

This failed helper result contains no `value` or partial aggregate. The outer agent's answer or `mcp_call.error` in the Responses API can report a generic knowledge base tool failure instead of this exact helper message. Check the [response status and tool errors](run-agents-responses-api.md#tool-errors). Don't interpret a failed or incomplete scan as a full total. Narrow the supported `filter` to reduce the number of matching records, and retry.

### Lookup and aggregation examples

These illustrative tool arguments use the `Orders` entity and fields exposed earlier in this guide.

For "What is the total for order 10248?", use `read_records`:

```json
{
  "entity": "Orders",
  "filter": "OrderId eq 10248",
  "select": "OrderId,Total"
}
```

For "What is the total value of all orders?", use `aggregate_records` on SQL Server, following its presented argument schema. On PostgreSQL or MySQL, use `aggregate_read_records` with:

```json
{
  "entity": "Orders",
  "function": "sum",
  "field": "Total"
}
```

The helper scans matching records to completion within its budget. It doesn't estimate the total from a sample.

### Tool names are prefixed

Typical tool names include the source kind, source name, and tool name, separated by double underscores. For a SQL Server source named `northwind-orders`, examples are:

```text
sql__northwind-orders__read_records
sql__northwind-orders__aggregate_records
```

For a PostgreSQL or MySQL source with that name, the added aggregation tool is typically `sql__northwind-orders__aggregate_read_records`.

The prefix prevents tool name collisions when a knowledge base contains multiple sources that expose the same underlying tool. The runtime sanitizes and truncates long prefixes and can add a collision suffix. Use the actual tool names presented to the agent rather than assuming an arbitrarily long prefix is preserved.

## Rotate the credential

Use `PUT /{name}/credential` to replace a connection string. Don't use `PATCH` — the update body doesn't include a `connection_string` field.

```bash
curl -X PUT https://<cluster-domain>/knowledgesources/northwind-orders/credential \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "connection_string": "<new-connection-string>"
  }'
```

The rotation process works as follows:

1. The replacement credential is validated **before** making any changes. It must require TLS with full certificate verification, its host must pass the address guard, and it must reach the database and still read every table the knowledge source exposes.
1. **If validation fails, the request returns `400` and nothing changes.** The existing credential and the serving workload remain unchanged, and the knowledge source keeps answering on the old credential.
1. On success, the new credential is written to a new Secret alongside the old one, the request returns `202`, and the operator rolls the workload.
1. For a credential-only rotation of a ready source, managed knowledge base execution can keep using the previous ready generation while the replacement starts. The API still reports `Provisioning` until the current generation is ready.
1. Poll `GET /{name}` for `provisioning_state`.
1. **If the replacement never becomes ready, the knowledge source deliberately stops and reports `Failed`.** It doesn't fall back to the old credential indefinitely. A credential that is rotated away from is treated as one that must no longer be used.
1. Old credential Secrets are retired after the new one is successfully adopted.

This route doesn't include an `engine` field. The engine is fixed at creation, so a rotation can't change it.

## Change the exposed schema

Use `PUT /{name}/schema` to change which tables and columns the agent can see. Read the current set first:

```bash
curl https://<cluster-domain>/knowledgesources/northwind-orders/schema \
  -H "Authorization: Bearer $TOKEN"
```

The response contains `engine`, `database_label`, and `entities`. Extract and edit `entities`, then send only an object containing that list. Don't send the full GET response: PUT doesn't accept `engine` or `database_label`, and unknown fields return `400`.

```bash
curl -X PUT https://<cluster-domain>/knowledgesources/northwind-orders/schema \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer $TOKEN" \
  -d '{
    "entities": [
      {
        "name": "Orders",
        "source_object": "Orders",
        "source_type": "table",
        "description": "One row per customer order.",
        "fields": [
          { "name": "OrderId", "description": "Unique order identifier.", "is_primary_key": true },
          { "name": "OrderDate", "description": "Date the order was placed, in UTC." }
        ]
      }
    ]
  }'
```

The update proceeds as follows:

1. The supplied entity list **replaces** the exposed tables and columns. It isn't merged with the current list.
1. **An invalid or empty list that renders no servable configuration returns `400`, and the live knowledge source doesn't change.**
1. On success the request returns `202` and triggers a workload rollout and a fresh handshake.
1. The source can be unavailable to managed knowledge base execution during the rollout. Keeping an old workload running doesn't provide the credential-rotation serving exception for a schema change.
1. A failed rollout restores the prior schema and reports `Failed`.

Poll `GET /{name}` until `provisioning_state` is `Succeeded` or `Failed`. Pending edits are never reported as `Succeeded`.

## SQL API reference

SQL sources use the same `/knowledgesources` base path and shared create, get, list, PATCH, and delete operations as other source kinds. See the [Knowledge sources REST API reference](reference/knowledge-sources-api-reference.md) for common fields, names, authentication, paging, and the resource response.

The following SQL-specific contracts complement the workflow examples above. JSON request bodies reject unknown fields.

### SQL parameters

Send `sql_parameters` in shared create or PATCH requests. Create also requires `kind: "sql"` and `auth_type`.

| Field | Type | Create | PATCH | Description |
|---|---|---|---|---|
| `engine` | String | Required. | Required. | `mssql`, `postgresql`, or `mysql`. PATCH must match the stored engine. |
| `connection_string` | String | Required. | Not accepted. | Write-only database credential. Use the dedicated credential PUT to rotate it. |
| `database_label` | String | Optional. | Optional. | 1-64 characters. Defaults to `database`, including when omitted from a replacement PATCH block. |
| `entities` | Array | Optional in the model; a servable nonempty set is required. | Required, nonempty. | Up to 200 exposed tables or views. An empty create set returns `400`. |

PATCH replaces the exposed entity set rather than merging it, while retaining the stored credential. Shared SQL `PATCH /knowledgesources/{name}` returns `200 OK` (with `provisioning_state` set to `"Provisioning"` in the response body when `sql_parameters` changes, requiring `GET /knowledgesources/{name}` polling), while `PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`. See [Response codes and readiness](#response-codes-and-readiness).

Each entity has these fields:

| Field | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | String | Agent-facing identifier matching `^[A-Za-z_][A-Za-z0-9_]*$`. 1-128 characters. |
| `source_object` | Yes | String | Exact table or view name from preview, including any non-default schema prefix. 1-256 characters. |
| `source_type` | No | String | `table` or `view`. Defaults to `table`. |
| `description` | No | String | Table or view meaning. Maximum 500 characters. |
| `fields` | No | Array | Up to 500 exposed columns. Omitted or empty permits all columns readable by the principal. |

Each field has these properties:

| Property | Required | Type | Description |
|---|---|---|---|
| `name` | Yes | String | Column name. 1-128 characters. |
| `description` | No | String | Column meaning. Maximum 500 characters. |
| `alias` | No | String or null | Alternative exposed name. Maximum 128 characters. |
| `is_primary_key` | No | Boolean | Defaults to `false`. |
| `data_type` | No | String | Informational type from preview. Accepted in requests but not stored in the custom resource. |

`source_object`, field names, and aliases reject unsafe identifier characters, including quotes, backticks, brackets, semicolons, backslashes, control characters, and SQL-comment markers. Entity and field models reject unknown properties.

Keep explicit nonempty field lists, useful descriptions, and primary-key metadata when selecting the schema. See [Trim the schema and describe it](#step-2-trim-the-schema-and-describe-it).

### Preview schema

```http
POST /knowledgesources/schema-preview
```

This synchronous request doesn't create a source. See the [preview example](#step-1-preview-the-database-schema).

| Request field | Required | Type | Description |
|---|---|---|---|
| `engine` | Yes | String | `mssql`, `postgresql`, or `mysql`. |
| `connection_string` | Yes | String | Write-only. Used for this request, then discarded; never stored or returned. |
| `database_label` | No | String | 1-64 characters. Defaults to `database`. |
| `include_schemas` | No | Array of strings or null | Up to 20 schema names, each 1-128 characters, to narrow discovery. |

| Response field | Type | Description |
|---|---|---|
| `schema_spec` | Object | `engine`, `database_label`, and the discovered `entities`, using the models above. Review before creating a source. |
| `discovered_entity_count` | Integer | Readable entities found before truncation. |
| `truncated` | Boolean | Whether discovery exceeded the returned entity limit. Narrow `include_schemas` and retry. |

Only `schema_spec` maps to the SQL create parameters, after review and addition of `connection_string`. `discovered_entity_count` and `truncated` aren't accepted create fields. Preserve exact `source_object` values as explained in the preview workflow.

### Get exposed schema

```http
GET /knowledgesources/{name}/schema
```

`name` is the required SQL source name. Returns `engine`, `database_label`, and `entities` using the models above, without credentials. This is the saved exposed schema, not confirmation that a pending change is already served.

To update it, extract and edit only `entities`. Don't submit the full GET response to PUT.

### Update exposed schema

```http
PUT /knowledgesources/{name}/schema
```

`name` is the required SQL source name. The body accepts only `entities`, a required array of 1-200 entities using the models described earlier. Including other top-level fields, such as `engine` and `database_label`, returns `400`.

The list fully replaces the exposed schema. Synchronous validation rejects invalid or unservable input before saving it. Success returns `202` with the common knowledge source response; serving adoption continues afterward. For the example, rollout availability, and failed-rollout rollback, see [Change the exposed schema](#change-the-exposed-schema).

### Rotate credential

```http
PUT /knowledgesources/{name}/credential
```

`name` is the required SQL source name. The body accepts only a required, write-only `connection_string`. There's no `engine` field; the stored engine can't change.

Validation completes before any change. Success returns `202` with the common knowledge source response while serving adopts the replacement. For the example, temporary previous-generation serving, terminal failure, and credential retirement, see [Rotate the credential](#rotate-the-credential).

### SQL response codes

The following table summarizes SQL-specific outcomes. Invalid request-model input returns `400`.

| Operation | Success | Errors |
|---|---|---|
| Shared create with `kind: "sql"` | `202`; common resource response. | `400` invalid request, database validation, or unservable schema; `409` duplicate name; `429` source/read capacity; `500` engine driver unavailable. |
| Shared SQL PATCH | `200 OK`; common resource response (`provisioning_state` is `"Provisioning"` when `sql_parameters` changes, requiring `GET /knowledgesources/{name}` polling; `PUT /knowledgesources/{name}/schema` and `PUT /knowledgesources/{name}/credential` return `202 Accepted`). | `400` invalid request, changed engine, wrong parameters kind, or unservable schema; `404` missing source; `409` concurrent modification. |
| Preview schema | `200`; preview response. | `400` invalid request or unreadable database; `429` concurrent read limit; `500` engine driver unavailable. |
| Get exposed schema | `200`; exposed-schema response. | `400` non-SQL source; `404` missing source. |
| Update exposed schema | `202`; common resource response. | `400` non-SQL source or invalid/unservable schema; `404` missing source; `409` concurrent modification. |
| Rotate credential | `202`; common resource response. | `400` non-SQL source or invalid/unverified credential; `404` missing source; `409` concurrent modification; `429` concurrent read limit; `500` engine driver unavailable. |

After a `202` or schema-affecting PATCH, poll the shared `GET /{name}` for current-generation readiness. A schema GET or successful preview doesn't replace that check.

## Related content

- [Knowledge sources REST API reference](reference/knowledge-sources-api-reference.md) for shared management operations.
- [Knowledge bases REST API reference](reference/knowledge-bases-api-reference.md) to group sources for an agent.
- [Configure a knowledge source](knowledge-sources-guide.md) for the common management workflow.
- [Indexed Sources MCP](mcp-server-overview.md) for indexed retrieval and external MCP source connections.
