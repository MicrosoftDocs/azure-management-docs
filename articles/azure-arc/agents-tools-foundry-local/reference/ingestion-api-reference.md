---
title: Ingestion REST API Reference - Agentic Retrieval in Foundry Local
description: REST API reference for managing document ingestion jobs in Agentic Retrieval in Foundry Local.
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/27/2026
ms.subservice: edge-rag
ai-usage: ai-assisted
---

# Ingestion REST API reference

Manage ingestion jobs in Agentic Retrieval in Foundry Local. The ingestion API ingests documents from NFS or SharePoint data sources into a collection. The API parses, chunks, embeds, and stores documents in the collection's vector collections and database tables.

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## API information

All endpoints require the `api-version=2024-10-01-preview` query parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Ingestion API (`ingestionapi`) |
| **Port** | 8000 |
| **Dapr App ID** | `ingestionapi` |

## Access Methods

You can reach the Ingestion API through three access methods. The access method determines whether authentication is required.

### External access (via ingress)

```
https://<cluster-domain>/edgeai/ingestion/...
```

Requires a valid JWT token (see [Authentication](#authentication)). The ingress adds the `X-External-Request` header which triggers Entra Auth sidecar validation.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/ingestionapi-deployment 8000:8000 -n arc-rag
```

Then call `http://localhost:8000/edgeai/ingestion/...`. No `Authorization` header needed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: ingestionapi" http://localhost:3500/edgeai/ingestion/...
```

No `Authorization` header needed for internal Dapr calls.

## Authentication

Required only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

All external API calls require a valid Entra ID JWT token with the `EdgeRAGDeveloper` app role.

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

## Create

Starts an ingestion job that parses, chunks, embeds, and stores documents from an NFS or SharePoint data source into a collection.

### Request

```http
POST /edgeai/ingestion/jobs/{job_id}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `job_id` | path | True | String | A user-chosen name for the ingestion job (for example, `nfs-quarterly-reports`). Reusing the same `job_id` creates a new "run" under that job. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

| Name | Required | Type | Description |
|---|---|---|---|
| `datasource.kind` | True | String | `"nfsv3"` or `"sharepoint"`. |
| `datasource.connection.nfsServerPath` | Conditional | String | NFS server path in format `server_ip:/path`. Required when `kind` is `"nfsv3"`. |
| `datasource.connection.nfsUid` | False | Integer | NFS user ID. Default: `0`. |
| `datasource.connection.nfsGid` | False | Integer | NFS group ID. Default: `0`. |
| `datasource.connection.sharePointUrl` | Conditional | String | SharePoint site URL. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointFolderPath` | Conditional | String | SharePoint folder path. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointUsername` | Conditional | String | SharePoint username in `DOMAIN\username` format. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointPassword` | Conditional | String | RSA-encrypted base64-encoded password. Required when `kind` is `"sharepoint"`. |
| `datasource.chunking.chunkSize` | True | String | Chunk size in characters. **Must be a string, not an integer.** |
| `datasource.chunking.chunkOverlap` | True | String | Overlap between chunks in characters. **Must be a string, not an integer.** |
| `collectionName` | False | String | Target collection name. Must exist before ingestion (create via [Collections API](collections-api-reference.md)). Default: `"edgeragapp"`. The alias `embeddingIndexName` is also accepted for backward compatibility. |
| `dataRefreshIntervalInHours` | False | Integer | `-1` = one-time ingestion, `1`/`4`/`24` = recurring sync. Default: `-1`. |
| `parsingMode` | False | String | `"basic"` (LangChain) or `"advanced"` (Docling). Default: `"basic"`. Use `"advanced"` for better results. |
| `excludeLgrFailureUpdate` | False | Boolean | If `true`, don't mark job as FAILED when LazyGraphRAG build fails. Default: `false`. |

### Example: NFS data source

#### Request

```http
POST /edgeai/ingestion/jobs/q1-ingest?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "datasource": {
    "kind": "nfsv3",
    "connection": {
      "nfsServerPath": "10.244.3.70:/exports/my-data"
    },
    "chunking": {
      "chunkSize": "2000",
      "chunkOverlap": "200"
    }
  },
  "collectionName": "my-collection",
  "parsingMode": "advanced"
}
```

#### Response

**Status code:** 200 OK

**Response headers:**

```http
operation-location: /edgeai/ingestion/jobs/q1-ingest/runs/{run_id}?api-version=2024-10-01-preview
```

**Response body:**

```json
{
  "status": "SUCCESS",
  "message": {
    "datasource": { "..." : "..." },
    "dataRefreshIntervalInHours": -1,
    "embeddingIndexName": "my_collection",
    "originalCollectionName": "my-collection",
    "ingestionGroupName": "",
    "parsingMode": "advanced",
    "excludeLgrFailureUpdate": false
  }
}
```

The `embeddingIndexName` in the response shows the sanitized storage prefix with underscores. The `originalCollectionName` shows the original name you provided.

### Example: SharePoint data source

#### Request

```http
POST /edgeai/ingestion/jobs/sp-ingest?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "datasource": {
    "kind": "sharepoint",
    "connection": {
      "sharePointUrl": "https://company.sharepoint.com/sites/docs",
      "sharePointFolderPath": "/sites/site-name/Shared Documents",
      "sharePointUsername": "DOMAIN\\username",
      "sharePointPassword": "<RSA-encrypted-base64>"
    },
    "chunking": {
      "chunkSize": "2000",
      "chunkOverlap": "200"
    }
  },
  "collectionName": "my-collection",
  "parsingMode": "advanced"
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The service accepts the ingestion job. The `operation-location` response header contains the URI to monitor the run. |
| 400 | Bad Request. Collection not found (`COLLECTION_NOT_FOUND`) or invalid NFS path. |
| 422 | Unprocessable Entity. Missing required fields (Pydantic validation error). |

## List

Returns all ingestion job runs.

### Request

```http
GET /edgeai/ingestion/jobs?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Example

#### Request

```http
GET /edgeai/ingestion/jobs?api-version=2024-10-01-preview

Authorization: Bearer eyJ0eX...FWSXfwtQ
```

#### Response

**Status code:** 200 OK

```json
{
  "jobRuns": [
    {
      "jobId": "my-job",
      "runId": "abc123def456",
      "status": "COMPLETED",
      "startTime": "2026-03-04T12:00:00+00:00",
      "endTime": "2026-03-04T12:05:00+00:00",
      "createdBy": "<your-display-name>",
      "collectionName": "my-collection"
    }
  ]
}
```

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the list of job runs. |

### Job status values

| Status | Description |
|---|---|
| `PENDING` | The job is queued and hasn't started. |
| `RUNNING` | The job is currently processing documents. |
| `COMPLETED` | The job finished successfully. |
| `FAILED` | The job encountered an error. |

## Get Run

Gets detailed information about a specific ingestion run, including progress and error information.

### Request

```http
GET /edgeai/ingestion/jobs/{job_id}/runs/{run_id}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `job_id` | path | True | String | The ingestion job name. |
| `run_id` | path | True | String | The auto-generated run identifier (returned in the `operation-location` header). |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains detailed run info including `totalItems`, `processedItems`, `failedItems`, and `errorInfo`. |

## Delete Runs

Deletes all runs for a given job. You can't delete runs while jobs are in `PENDING` or `RUNNING` status.

### Request

```http
DELETE /edgeai/ingestion/jobs/{job_id}/runs?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `job_id` | path | True | String | The ingestion job name. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The job runs were deleted. |
| 409 | Conflict. Cannot delete while jobs are in `PENDING` or `RUNNING` status. |

## Related content

- [Collections API Reference](collections-api-reference.md) — Create and manage collections before ingesting
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG after ingestion
