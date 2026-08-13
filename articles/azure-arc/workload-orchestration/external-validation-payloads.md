---
title: Event Grid external validation payload
description: Learn about Event Grid payload for external validation of solution versions in workload orchestration.
author: sethmanheim
ms.author: sethm
ms.topic: how-to
ms.date: 05/10/2025
ms.custom:
  - build-2025
# Customer intent: "As a cloud architect, I want to learn the details on the Event Grid payload for external validation of solution versions in workload orchestration."
---

# Event Grid external validation payload

This article provides details on the Event Grid payload for external validation of solution versions in workload orchestration. It includes information on the Event Grid message format, field descriptions, and API endpoints for getting solution version resources and updating external validation status.

## Event Grid payload for `Microsoft.Edge.SolutionVersionPublished`

When you publish a solution version, the system sends an event to the Event Grid topic associated with the context resource. This event contains information about the solution version and a callback URL for updating the external validation status.

The JSON payload of the Event Grid message contains the following fields. The `data` section includes the `externalValidationId`, which is a unique identifier for the external validation request, and the `callbackUrl`, which is the endpoint to invoke after the validation result is received.


```json
[
  {
    "id": "event-id-guid",
    "topic": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/contexts/<context-name>",
    "subject": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
    "eventType": "Microsoft.Edge.SolutionVersionPublished",
    "eventTime": "2025-04-21T11:19:25.281991Z",
    "data": {
      "externalValidationId": "validation-id-guid",
      "targetId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>",
      "solutionTemplateId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/solutionTemplates/<template-name>",
      "solutionTemplateVersionId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/solutionTemplates/<template-name>/versions/<template-version>",
      "solutionVersionId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
      "apiVersion": "2025-01-01-preview",
      "callbackUrl": "https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/updateExternalValidationStatus?api-version=2025-01-01-preview"
    },
    "dataVersion": "1"
  }
]
```

The following table describes the fields in the Event Grid payload:


| Field                            | Description                                                                      |
| -------------------------------- | -------------------------------------------------------------------------------- |
| `id`                             | Unique identifier for the event.                                                 |
| `topic`                          | Topic name; here, it indicates the context resource. |
| `subject`                        | Resource path of the published solution version, used to route or filter events.    |
| `eventType`                      | Type of event published — `Microsoft.Edge.SolutionVersionPublished`.             |
| `eventTime`                      | Timestamp when the event occurred, in UTC format.                                |
| `data.externalValidationId`      | Correlation ID for the external validation request.                              |
| `data.targetId`                  | ARM resource ID of the target resource.                    |
| `data.solutionTemplateId`        | ARM resource ID of the solution template.                                 |
| `data.solutionTemplateVersionId` | ARM ID of the specific version of the solution template.                         |
| `data.solutionVersionId`         | ARM ID of the actual solution version being validated.                           |
| `data.apiVersion`                | API version which you can use to query the resources.                           |
| `data.callbackUrl`               | Endpoint to invoke after the validation result, includes `api-version`.           |
| `dataVersion`                    | Version of the data schema.                         |


## GET API – Get a solution version resource

The `GET API` command can be executed on the solution version resource to fetch the resolved configurations. Here solution version ARM ID can be accessed from the `data.solutionVersionId` attribute and API version from `data.apiVersion` in the Event Grid payload.

Other Resource IDs in the Event Grid payload can also be queried using a similar `GET API`.

**Endpoint:**

```
GET https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>?api-version=2025-01-01-preview
```

**Headers:**

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

**Response Body:**

```json
{
  "properties": {
    "solutionTemplateVersionId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/solutionTemplates/<template-name>/versions/<template-version>",
    "revision": 1,
    "targetDisplayName": "<target-name>",
    "configuration": "<config-data>",
    "specification": {
      "components": [
        {
          "name": "helmcomponent",
          "type": "helm.v3",
          "properties": {
            "chart": {
              "repo": "<acr-url>/helm/<chart-name>:<chart-tag>",
              "version": "0.3.0",
              "wait": true,
              "timeout": "5m"
            },
            "values": {
              "AppName": "Hotmelt",
              "TemperatureRangeMax": "100",
              "ErrorThreshold": "20",
              "HealthCheckEndpoint": "localhost:8080",
              "EnableLocalLog": "true",
              "AgentEndpoint": "localhost:8080",
              "HealthCheckEnabled": "true",
              "ApplicationEndpoint": "localhost:8080"
            }
          }
        }
      ]
    },
    "reviewId": "<review-id-guid>",
    "externalValidationId": "<validation-id-guid>",
    "state": "PendingExternalValidation",
    "solutionInstanceName": "<solution-name>",
    "provisioningState": "Succeeded"
  },
  "extendedLocation": {
    "name": "/subscriptions/<subscription-id>/resourceGroups/<cluster-rg>/providers/Microsoft.ExtendedLocation/customLocations/<custom-location>",
    "type": "CustomLocation"
  },
  "eTag": "\"<etag-value>\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
  "name": "<version-name>",
  "type": "microsoft.edge/targets/solutions/versions"
}
```

## POST API – Update external validation status

Use the `POST API` command to update the external validation status of a solution version. Call this API after the external validation process finishes. It updates the solution version resource with the validation result.


**Endpoint:**

```
POST https://management.azure.com/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/updateExternalValidationStatus?api-version=2025-01-01-preview
```

**Headers:**

```http
Content-Type: application/json
Authorization: Bearer <access-token>
```

**Request Body:**

```json
{
  "externalValidationId": "validation-id-guid",
  "solutionVersionId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
  "validationStatus": "Invalid",
  "errorDetails": {} //explained below
}
```

The request body contains the following fields:

- `externalValidationId`: A unique GUID for tracking the validation operation. To ensure no stale updates, pass the same GUID you receive from the Event Grid message payload (`data.externalValidationId`).
- `solutionVersionId`: The full ARM ID of the solution version being validated. This ID is present in the Event Grid message payload (`data.solutionVersionId`).

### Error details object

The following example shows an `errorDetails` object that you can include in the request body when the validation status is `"Invalid"`. This object provides more information about the validation failure.

```json
{
  "code": "InvalidConfigurations",
  "message": "The provided configurations are invalid.",
  "target": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
  "additionalInfo": [
    {
      "type": "InvalidHelmChart",
      "info": {
        "level": "Error",
        "message": "The Helm chart provided is invalid or not supported."
      }
    }
  ]
}
```

The `errorDetails` object contains information about the validation failure. The following fields are included:

* `code`: A machine-readable error code indicating the issue category.
* `message`: A human-readable explanation of what went wrong.
* `target`: The ARM resource path affected by the error.
* `additionalInfo[]`:

  * `type`: The category or component related to the failure, such as Helm.
  * `info.level`: The severity of the error, such as `Error` or `Warning`.
  * `info.message`: Additional context for troubleshooting.



> [!NOTE]
> You don't need to include the `errorDetails` object if the validation status is `"Valid"`. In this case, omit the `errorDetails` field from the request body.


## POST API response

The response body for the POST API returns the updated solution version resource with the new validation status. The `state` field shows the new status. The `errorDetails` field contains any error information if the validation status is `"Invalid"`.


```json
{
  "properties": {
    "solutionTemplateVersionId": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/solutionTemplates/<template-name>/versions/<template-version>",
    "revision": 1,
    "errorDetails": {
      "code": "InvalidConfigurations",
      "message": "The provided configurations are invalid.",
      "target": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
      "additionalInfo": [
        {
          "type": "InvalidHelmChart",
          "info": {
            "level": "Error",
            "message": "The Helm chart provided is invalid or not supported."
          }
        }
      ]
    },
    "targetDisplayName": "<target-name>",
    "configuration": "<config-data>",
    "specification": {
      "components": [
        {
          "name": "helmcomponent",
          "type": "helm.v3",
          "properties": {
            "chart": {
              "repo": "<acr-url>/helm/<chart-name>:<chart-tag>",
              "version": "0.3.0",
              "wait": true,
              "timeout": "5m"
            },
            "values": {
              "AppName": "Hotmelt",
              "TemperatureRangeMax": "100",
              "ErrorThreshold": "20",
              "HealthCheckEndpoint": "localhost:8080",
              "EnableLocalLog": "true",
              "AgentEndpoint": "localhost:8080",
              "HealthCheckEnabled": "true",
              "ApplicationEndpoint": "localhost:8080"
            }
          }
        }
      ]
    },
    "reviewId": "<review-id-guid>",
    "externalValidationId": "<validation-id-guid>",
    "state": "ExternalValidationFailed",
    "solutionInstanceName": "<solution-name>",
    "provisioningState": "Succeeded"
  },
  "extendedLocation": {
    "name": "/subscriptions/<subscription-id>/resourceGroups/<cluster-rg>/providers/Microsoft.ExtendedLocation/customLocations/<custom-location>",
    "type": "CustomLocation"
  },
  "eTag": "\"<etag-value>\"",
  "id": "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Edge/targets/<target-name>/solutions/<solution-name>/versions/<version>",
  "name": "<version-name>",
  "type": "microsoft.edge/targets/solutions/versions"
}
```

## Update external validation status

The `validationStatus` field indicates the result of validation, which can be `"Valid"` or `"Invalid"`.

### Set valid configuration

#### [Bash](#tab/bash)

```bash
az workload-orchestration target update-external-validation-status \
 --resource-group <rg-name> \
 --target-name <target-name> \
 --external-validation-id <external-validation-id> \
 --solution-version-id <solution-version-id> \
 --validation-status "Valid"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target update-external-validation-status `
 --resource-group <rg-name> `
 --target-name <target-name> `
 --external-validation-id <external-validation-id> `
 --solution-version-id <solution-version-id> `
 --validation-status "Valid"
```
***

### Set invalid configuration

#### [Bash](#tab/bash)

```bash
az workload-orchestration target update-external-validation-status \
 --resource-group <rg-name> \
 --target-name <target-name> \
 --external-validation-id <external-validation-id> \
 --solution-version-id <solution-version-id> \
 --validation-status "Invalid" \
 --error-details "@error.json"
```

#### [PowerShell](#tab/powershell)

```powershell
az workload-orchestration target update-external-validation-status `
 --resource-group <rg-name> `
 --target-name <target-name> `
 --external-validation-id <external-validation-id> `
 --solution-version-id <solution-version-id> `
 --validation-status "Invalid" `
 --error-details "@error.json"
```

***

The *error.json* file uses the same schema as the `errorDetails` object.
