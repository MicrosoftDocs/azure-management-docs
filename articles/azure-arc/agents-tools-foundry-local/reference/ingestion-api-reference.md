---
title: Ingestion REST API Reference - Agents and Tools with Foundry Local
description: REST API reference for managing document ingestion jobs in Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/05/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
---

# Ingestion REST API reference

Manage ingestion jobs in Agents and Tools with Foundry Local. The ingestion API ingests documents from NFS or SharePoint data sources into a collection. The API parses, chunks, embeds, and stores documents in the collection's vector collections and database tables.

> [!NOTE]
> Local ingestion and the developer portal are available only when `indexedSourcesEnabled=true`. Create the target collection with `POST /collections` before you start ingestion. See the [Collections REST API reference](collections-api-reference.md).

## API information

All endpoints require the `api-version=2024-10-01-preview` query parameter.

| Property | Value |
|---|---|
| **API version** | `2024-10-01-preview` |
| **Service** | Ingestion API (`ingestionapi`) |
| **Port** | 8000 |
| **Dapr App ID** | `ingestionapi` |

## Access methods

You can reach the Ingestion API through three access methods. The access method determines whether authentication is required.

### External access (via ingress)

```text
https://<cluster-domain>/ingestion/...
```

Requires a valid Microsoft Entra ID JSON Web Token (JWT). See [Authentication](#authentication). The ingress adds the `X-External-Request` header, which triggers authentication and authorization checks in the service.

### Port forwarding (for development and testing)

```bash
kubectl port-forward deployment/ingestionapi-deployment 8000:8000 -n arc-rag
```

Then call `http://localhost:8000/ingestion/...`. No `Authorization` header needed.

### Internal access (from a pod via Dapr)

```bash
curl -H "dapr-app-id: ingestionapi" http://localhost:3500/ingestion/...
```

No `Authorization` header needed for internal Dapr calls.

## Authentication

Required only for **external access** (via ingress). Port forwarding and internal Dapr calls bypass authentication.

All external API calls require a valid Microsoft Entra ID JWT. The service checks Azure role permissions first. If that check doesn't allow the request, the `EdgeRAGDeveloper` app role can allow it as a fallback.

| Built-in Azure role | Operation | Azure permission |
|---|---|---|
| Reader | Read jobs and runs | `Microsoft.CognitiveServices/accounts/ingestionJobs/read` |
| Cognitive Services Contributor in connected environments; Contributor in disconnected environments | Create a run | `Microsoft.CognitiveServices/accounts/ingestionJobs/write` |

For role setup and assignment scope, see [Assign Azure roles](../prepare-authentication.md#assign-azure-roles). Azure roles allow API access, but the developer portal also requires the `EdgeRAGDeveloper` app role.

```bash
az account clear
az login --tenant <your-tenant-id> --output none

TOKEN=$(az account get-access-token \
  --resource "api://<your-app-client-id>" \
  --query accessToken -o tsv)
```

Tokens expire after about one hour. If you receive a `401 Unauthorized`, acquire a fresh token.

## Create

Creates a run that parses, chunks, embeds, and stores documents from an NFS or SharePoint data source in an existing collection. Only one ingestion run executes at a time. Other runs wait with the `PENDING` status.

### Request

```http
POST /ingestion/jobs/{job_id}?api-version=2024-10-01-preview
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `job_id` | path | True | String | A user-chosen name for the ingestion job (for example, `nfs-quarterly-reports`). Reusing the same `job_id` creates a new run under that job. |
| `api-version` | query | True | String | The API version to use. |

### Request headers

| Name | Required | Type | Description |
|---|---|---|---|
| `Content-Type` | True | String | `application/json` |
| `Authorization` | Conditional | String | `Bearer {token}`. Required for external access only. |

### Request body

A globally unique identifier (GUID) uses the `8-4-4-4-12` hexadecimal format, such as `00000000-0000-0000-0000-000000000001`.

| Name | Required | Type | Description |
|---|---|---|---|
| `datasource.kind` | True | String | `"nfsv3"` or `"sharepoint"`. |
| `datasource.connection.nfsServerPath` | Conditional | String | NFS path in `server:/path` format. Use a server DNS name or IPv4 address. Required when `kind` is `"nfsv3"`. |
| `datasource.connection.nfsUid` | False | Integer | NFS user ID. Default: `0`. |
| `datasource.connection.nfsGid` | False | Integer | NFS group ID. Default: `0`. |
| `datasource.connection.sharePointUrl` | Conditional | String | SharePoint site URL. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointFolderPath` | Conditional | String | Server-relative SharePoint folder path that starts with `/`. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointClientId` | Conditional | String | SharePoint app principal client ID, in GUID format. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointIssuerId` | Conditional | String | Trusted token issuer ID, in GUID format. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointSid` | Conditional | String | Windows security identifier (SID) of the SharePoint service account. Use `S-1-5-21-<number>-<number>-<number>-<number>`, with digits in each numeric part. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointRealm` | False | String | Optional SharePoint authentication realm, in GUID format. The service discovers it when omitted or empty. |
| `datasource.connection.sharePointCertSecretName` | Conditional | String | Existing Kubernetes Secret in `arc-rag` that contains `cert.pfx`. Required when `kind` is `"sharepoint"`. |
| `datasource.connection.sharePointCertPasswordSecretName` | False | String | Optional password Secret. Defaults to `sharePointCertSecretName`. |
| `datasource.connection.sharePointCertPasswordSecretKey` | False | String | Optional password key. Defaults to `cert-password`; the key must exist even when the password is empty. |
| `datasource.chunking.chunkSize` | True | String | Accepted, but not applied in the current release. Send a string, such as `"2000"`, not an integer. |
| `datasource.chunking.chunkOverlap` | True | String | Accepted, but not applied in the current release. Send a string, such as `"200"`, not an integer. |
| `collectionName` | False | String | Target collection name, up to 49 characters. Must exist before ingestion (create via [Collections API](collections-api-reference.md)). Default: `"edgeragapp"`. The alias `embeddingIndexName` is also accepted for backward compatibility. |
| `dataRefreshIntervalInHours` | False | Integer | `-1` = one-time ingestion. The portal offers `1`, `4`, or `24` hours for recurring sync. Default: `-1`. |
| `parsingMode` | False | String | Only `"advanced"` (Docling) is supported, and it is the default. The retired `"basic"` value returns a validation error. |

The service uses Docling HybridChunker with the BGE-M3 tokenizer to split text into chunks of about 512 tokens. A token is a small unit of text, such as a word or part of a word.

> [!NOTE]
> When `nfsUid` and `nfsGid` are both `0`, the request uses Kerberos if Kerberos is enabled for the deployment. Otherwise, `0`/`0` means root access through AUTH_SYS, which uses numeric user and group IDs. To use AUTH_SYS when Kerberos is enabled, don't set both IDs to `0`.

### Example: NFS data source

#### Request

```http
POST /ingestion/jobs/q1-ingest?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "datasource": {
    "kind": "nfsv3",
    "connection": {
      "nfsServerPath": "10.244.3.70:/exports/my-data",
      "nfsUid": 1003,
      "nfsGid": 1003
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
operation-location: /ingestion/jobs/q1-ingest/runs/{run_id}?api-version=2024-10-01-preview
```

**Response body (selected fields):**

```json
{
  "status": "SUCCESS",
  "message": {
    "dataRefreshIntervalInHours": -1,
    "collectionName": "my_collection",
    "originalCollectionName": "my-collection",
    "ingestionGroupName": "",
    "parsingMode": "advanced"
  }
}
```

The `collectionName` in the response shows the storage prefix, where hyphens become underscores. The `originalCollectionName` shows the name you provided.

### Example: SharePoint data source

#### Request

```http
POST /ingestion/jobs/sp-ingest?api-version=2024-10-01-preview

Content-Type: application/json
Authorization: Bearer eyJ0eX...FWSXfwtQ
```

```json
{
  "datasource": {
    "kind": "sharepoint",
    "connection": {
      "sharePointUrl": "https://sharepoint.contoso.com/sites/docs",
      "sharePointFolderPath": "/sites/docs/Shared Documents",
      "sharePointClientId": "00000000-0000-0000-0000-000000000001",
      "sharePointIssuerId": "00000000-0000-0000-0000-000000000002",
      "sharePointSid": "S-1-5-21-111111111-222222222-333333333-1001",
      "sharePointRealm": "",
      "sharePointCertSecretName": "sharepoint-s2s-cert",
      "sharePointCertPasswordSecretName": "sharepoint-s2s-cert",
      "sharePointCertPasswordSecretKey": "cert-password"
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

Replace the sample Client ID, Issuer ID, and SID with your SharePoint values. See [Set up SharePoint server-to-server authentication](../connect-sharepoint-setup.md).

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The service accepts the ingestion job. The `operation-location` response header contains the URI to monitor the run. |
| 400 | Bad Request. The collection, NFS connection, or SharePoint configuration isn't valid. See [Validation errors](#validation-errors). |
| 401 | Unauthorized. The token is missing or isn't valid. |
| 403 | Forbidden. Neither the Azure role check nor the app-role fallback allows the request. |
| 422 | Unprocessable Entity. Request validation fails, for example because a required field is missing, a GUID or SID isn't valid, the folder path doesn't start with `/`, or `parsingMode` is `"basic"`. |
| 503 | Service Unavailable. For example, the service can't read or validate SharePoint certificate Secrets through the Kubernetes API. Check service-account permissions and cluster connectivity. |

### Validation errors

The following checks return `400 Bad Request` before ingestion starts.

| Reason | Response |
|---|---|
| The target collection doesn't exist | `error.code` is `COLLECTION_NOT_FOUND`. The message is `Collection '<name>' not found. Create collection first via POST /collections`. |
| NFS server DNS lookup fails | `detail` is `Invalid NFS Path: Cannot resolve server '<server>'`. |
| NFS TCP port 2049 isn't reachable | `detail` is `Invalid NFS Path: Server '<server>' (IP: <ip>) port 2049 is not accessible`. |
| Another NFS path check fails | `detail` is `Invalid NFS Path`. Missing or malformed path fields can instead fail request validation with 422. |
| SharePoint ingestion is disabled | `detail` is `SharePoint ingestion is disabled for this deployment.` |
| SharePoint identity or certificate Secret name is missing | `detail` starts with `SharePoint ingestion requires:` and lists the missing Certificate Secret name, Client ID, Issuer ID, or Windows SID. Partial identity values can instead fail request validation with 422. |
| A SharePoint certificate or password Secret doesn't exist | `detail` identifies the Secret and application namespace and asks you to create it before ingestion. |
| A SharePoint Secret key is missing | `detail` identifies the Secret and required data key. The `cert.pfx` value must not be empty. The password key must exist, but its value can be empty. |
| A generated ingestion connection Secret is used for signing | `detail` asks you to use a customer-managed Secret instead. |

## List

Returns the last 10 ingestion jobs, with up to the last 10 runs for each job. The response combines those runs in one `jobRuns` array. The endpoint doesn't support paging.

### Request

```http
GET /ingestion/jobs?api-version=2024-10-01-preview
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
GET /ingestion/jobs?api-version=2024-10-01-preview

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
| `UNKNOWN` | The run state isn't known. |
| `RUNNING` | The run is processing documents. |
| `COMPLETED` | The run finished successfully. |
| `FAILED` | The run encountered an error. |
| `TERMINATED` | The run stopped before it finished. |
| `PENDING` | The run is queued and hasn't started. |
| `SUSPENDED` | The run is suspended. |

## List runs

Returns up to the last 10 runs for one ingestion job in a `jobRuns` array. The endpoint doesn't support paging.

### Request

```http
GET /ingestion/jobs/{job_id}/runs?api-version=2024-10-01-preview
Authorization: Bearer <token>
```

### URI parameters

| Name | In | Required | Type | Description |
|---|---|---|---|---|
| `job_id` | path | True | String | The ingestion job name. |
| `api-version` | query | True | String | The API version to use. |

### Response codes

| Code | Description |
|---|---|
| 200 | OK. The response contains the recent runs. |
| 404 | Not Found. `detail` is `Invalid job_id`. |

## Get run

Gets detailed information about a specific ingestion run, including progress and error information.

### Request

```http
GET /ingestion/jobs/{job_id}/runs/{run_id}?api-version=2024-10-01-preview
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
| 200 | OK. The response contains run details, including `input`, `progress`, and `errorInfo`. |
| 404 | Not Found. `detail` is `Invalid run_id`. |

### Progress fields

The `progress` object contains these counters:

| Field | Description |
|---|---|
| `totalItems` | Total number of items found for the run. |
| `processedItems` | Number of items processed. |
| `failedItems` | Number of items that failed. |
| `skippedItems` | Number of items skipped. |
| `nfsTimeoutCount` | Number of NFS timeouts. Default: `0`. |

### Run errors

The `errorInfo` array contains entries with `code`, `message`, `timestamp`, `source`, and `severity`. Severity is `error` or `warning`. Use the message to find the cause.

| Code | Meaning |
|---|---|
| `NFS_ACCESS_DENIED` | Access to the NFS share is denied. |
| `NFS_NOT_FOUND` | The NFS share or path isn't found. |
| `NFS_PERMISSION_ERROR` | The NFS identity lacks the required file permissions. |
| `NFS_TIMEOUT` | An NFS operation times out. |
| `EMBEDDING_SERVICE_ERROR` | The embedding service fails. |
| `VECTOR_DB_CONNECTION_ERROR` | The run can't connect to the vector database. |
| `INVALID_FILE_FORMAT` | The file format isn't supported or isn't valid. |
| `PROCESSING_TIMEOUT` | File processing takes too long. |
| `DOCLING_PARSE_ERROR` | Docling can't parse a document. |
| `PUBLISHER_FILE_ERROR` | The ingestion publisher can't list or prepare source files. |
| `POD_FAILED` | An ingestion pod fails. |
| `IMAGE_PULL_ERROR` | Kubernetes can't download a container image. |
| `CONTAINER_CONFIG_ERROR` | A container configuration isn't valid. |
| `POD_CRASH_LOOP` | A container repeatedly crashes. |
| `POD_STARTUP_ERROR` | An ingestion pod can't start. |
| `UNKNOWN_ERROR` | The error doesn't have a more specific code. |

## Related content

- [Collections API Reference](collections-api-reference.md) — Create and manage collections before ingesting
- [Inference API Reference](inference-api-reference.md) — Query collections using RAG after ingestion
