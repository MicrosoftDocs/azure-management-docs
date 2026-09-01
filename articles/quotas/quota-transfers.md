---
title: Transfer quota between Azure subscriptions
description: Learn how to transfer unused Azure quota from one subscription to another by using the Quota Transfer REST API.
author: tejasm-microsoft
ms.author: tejasma
ms.topic: how-to
ms.date: 09/01/2026
ai-usage: ai-assisted
---

# Transfer quota between Azure subscriptions

> [!IMPORTANT]
> Quota Transfer uses the preview API version `2026-09-01-preview`. Availability is limited to subscriptions, resource providers, regions, and quota resources enabled for the preview. Confirm enrollment and supported scope before integrating with this API.

## Overview

Quota Transfer moves unused quota from one Azure subscription (the **donor**) to another Azure subscription (the **recipient**). A transfer applies to one resource provider, region, and quota resource.

For example, a transfer can move 50 `standardDv5Family` vCPUs for `Microsoft.Compute` in `eastus` from one subscription to another.

A transfer:

- Decreases the donor limit and increases the recipient limit by the same amount.
- Doesn't create additional total quota.
- Doesn't move resources, deployments, usage, reservations, or billing ownership.
- Doesn't guarantee that capacity is available when the recipient deploys resources.
- Can't reduce the donor limit below its current usage during settlement.

## Prerequisites

Before creating a transfer, verify that:

1. Both subscriptions are enabled for the Quota Transfer preview.
1. The target resource provider, region, and quota resource support transfer.
1. The donor and recipient subscriptions belong to the same billing account.
1. The donor has unused quota at least equal to the transfer amount.
1. The caller has the required Azure RBAC permissions.
1. The `resourceName` is the quota resource name returned by the quota or usage API, not a display name.

The service validates the billing account for both subscriptions. The request fails closed if the service can't resolve either subscription's billing account or it doesn't match `billingAccountId`.

## Required permissions

The required Azure RBAC action is:

```text
Microsoft.Quota/quotas/write
```

For a two-step transfer, the donor caller requires this permission on the donor subscription, and the recipient caller requires it on the recipient subscription. For `autoApprove: true`, the submitting identity must have this permission on both subscriptions.

## Transfer models

### Two-step approval

Use the two-step model when different people manage the subscriptions or when the subscriptions are in different Microsoft Entra tenants.

1. The donor submits a transfer with `autoApprove: false`.
1. The recipient finds the transfer in its incoming-transfer list.
1. The recipient reviews and approves or rejects it.
1. After approval, the service settles both quota limits and reports the final status.

### Same-tenant auto-approval

Set `autoApprove: true` only when:

- The donor and recipient subscriptions have the same home tenant.
- The submitting identity has the required permission on the recipient subscription.

Always inspect the returned `transferStatus`. If the transfer remains `Pending`, the recipient must use the two-step approval flow.

## Resource paths

The donor chooses `transferName`. The service generates `transferId`, which the recipient uses.

```text
# Donor resource
/subscriptions/{donorSubscriptionId}/providers/{targetProvider}/locations/{region}/providers/Microsoft.Quota/quotaTransfers/{transferName}

# Recipient resource
/subscriptions/{recipientSubscriptionId}/providers/{targetProvider}/locations/{region}/providers/Microsoft.Quota/incomingQuotaTransfers/{transferId}
```

`transferName` must be 3-63 characters, start with an alphanumeric character, and contain only letters, numbers, `.`, `_`, or `-`.

## Submit a transfer

Set the variables used in the examples:

```bash
API_VERSION="2026-09-01-preview"
DONOR_SUBSCRIPTION_ID="<donor-subscription-id>"
RECIPIENT_SUBSCRIPTION_ID="<recipient-subscription-id>"
TARGET_PROVIDER="Microsoft.Compute"
REGION="eastus"
TRANSFER_NAME="compute-stdDv5-uplift-101"
BILLING_ACCOUNT_ID="<billing-account-id>"
RESOURCE_NAME="standardDv5Family"
ACCESS_TOKEN="<Azure Resource Manager access token>"

DONOR_RESOURCE_URI="https://management.azure.com/subscriptions/${DONOR_SUBSCRIPTION_ID}/providers/${TARGET_PROVIDER}/locations/${REGION}/providers/Microsoft.Quota/quotaTransfers/${TRANSFER_NAME}"
DONOR_URI="${DONOR_RESOURCE_URI}?api-version=${API_VERSION}"
```

Submit from the donor subscription:

```bash
curl --request PUT "$DONOR_URI" \
  --header "Authorization: Bearer $ACCESS_TOKEN" \
  --header "Content-Type: application/json" \
  --data @- <<JSON
{
  "properties": {
    "displayName": "Move 50 Dv5 vCPU to recipient",
    "comment": "Capacity for the recipient production rollout.",
    "destinationSubscriptionId": "${RECIPIENT_SUBSCRIPTION_ID}",
    "billingAccountId": "${BILLING_ACCOUNT_ID}",
    "resourceName": "${RESOURCE_NAME}",
    "amount": 50,
    "autoApprove": false
  }
}
JSON
```

### Request properties

| Property | Required | Validation |
| --- | --- | --- |
| `displayName` | Yes | Human-readable name, maximum 80 characters |
| `destinationSubscriptionId` | Yes | Valid subscription GUID different from the donor |
| `billingAccountId` | Yes | Must be the billing account for both subscriptions |
| `resourceName` | Yes | Supported quota resource under `targetProvider` |
| `amount` | Yes | Integer greater than or equal to 1 |
| `comment` | No | Maximum 500 characters |
| `autoApprove` | No | Defaults to `false` |

Save `properties.transferId` from the response. It's required for recipient operations and is the best support correlation identifier.

### Long-running response

Submit and approve can return `202 Accepted`, a polling URL in a response header, and `Retry-After`. Follow the URL returned by the service and wait at least the specified number of seconds between requests. Preview deployments can return the polling URL in `Location` or `Azure-AsyncOperation`. Clients must support either header.

After polling completes, get the transfer resource and use `properties.transferStatus` as the business outcome.

## Get or list outgoing transfers

```bash
# Get one transfer
curl --request GET "$DONOR_URI" \
  --header "Authorization: Bearer $ACCESS_TOKEN"

# List transfers for a provider and region
curl --request GET \
  "https://management.azure.com/subscriptions/${DONOR_SUBSCRIPTION_ID}/providers/${TARGET_PROVIDER}/locations/${REGION}/providers/Microsoft.Quota/quotaTransfers?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

Follow `nextLink` until it's empty when listing transfers.

## Get or list incoming transfers

The recipient can list one provider and region or list across all enabled providers and regions in the subscription:

```bash
# List for one provider and region
curl --request GET \
  "https://management.azure.com/subscriptions/${RECIPIENT_SUBSCRIPTION_ID}/providers/${TARGET_PROVIDER}/locations/${REGION}/providers/Microsoft.Quota/incomingQuotaTransfers?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN"

# List across the recipient subscription
curl --request GET \
  "https://management.azure.com/subscriptions/${RECIPIENT_SUBSCRIPTION_ID}/providers/Microsoft.Quota/incomingQuotaTransfers?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

Build the incoming resource URI with the server-generated ID:

```bash
TRANSFER_ID="<properties.transferId from the submit or list response>"
INCOMING_RESOURCE_URI="https://management.azure.com/subscriptions/${RECIPIENT_SUBSCRIPTION_ID}/providers/${TARGET_PROVIDER}/locations/${REGION}/providers/Microsoft.Quota/incomingQuotaTransfers/${TRANSFER_ID}"
INCOMING_URI="${INCOMING_RESOURCE_URI}?api-version=${API_VERSION}"

curl --request GET "$INCOMING_URI" \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

The incoming response identifies the donor in `properties.sourceSubscriptionId`.

## Approve an incoming transfer

Approve and reject use optimistic concurrency. First, get the incoming transfer and copy `properties.sourceEtag`.

> [!CAUTION]
> Use `properties.sourceEtag` in `If-Match`. Don't use the incoming resource's top-level `etag`, and don't use `If-Match: *`.

```bash
SOURCE_ETAG="<properties.sourceEtag from a current incoming GET>"

curl --request POST "${INCOMING_RESOURCE_URI}/approve?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN" \
  --header "Content-Type: application/json" \
  --header "If-Match: $SOURCE_ETAG" \
  --data '{"comment":"Approved by the recipient capacity owner."}'
```

Approval can be long-running. Follow the response-provided polling URL, and then get either side of the transfer to obtain the final `transferStatus`.

## Reject an incoming transfer

Reject is synchronous and also requires a current `properties.sourceEtag`.

```bash
curl --request POST "${INCOMING_RESOURCE_URI}/reject?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN" \
  --header "Content-Type: application/json" \
  --header "If-Match: $SOURCE_ETAG" \
  --data '{"reason":"Recipient capacity is no longer required."}'
```

If approve or reject returns `412 SourceResourceModified`, get the incoming transfer again, review its current state, and retry with the new `properties.sourceEtag` only if it's still `Pending`.

## Cancel a pending transfer

The donor can cancel only while the transfer is `Pending`. Cancellation is synchronous.

```bash
curl --request POST "${DONOR_RESOURCE_URI}/cancel?api-version=${API_VERSION}" \
  --header "Authorization: Bearer $ACCESS_TOKEN" \
  --header "Content-Type: application/json" \
  --data '{"reason":"The donor no longer wants to transfer this quota."}'
```

Cancellation stops a transfer that hasn't entered settlement. It doesn't reverse a completed transfer. To move quota back after completion, create a separate transfer in the opposite direction after confirming the subscriptions still satisfy all prerequisites.

## Delete a transfer record

Delete removes the donor-side transfer record. It doesn't move or restore quota. Only a terminal transfer can be deleted.

```bash
curl --request DELETE "$DONOR_URI" \
  --header "Authorization: Bearer $ACCESS_TOKEN"
```

A successful delete returns `200 OK`. Deleting a record that's already absent returns `204 No Content`. Deleting a nonterminal record returns `409 Conflict`.

## Status and lifecycle

Use `properties.transferStatus`, not `provisioningState`, to determine the business outcome.

| `transferStatus` | Meaning | Terminal |
| --- | --- | --- |
| `Pending` | Waiting for recipient action | No |
| `InProgress` | Approved and settlement is running | No |
| `Completed` | Donor debit and recipient credit completed | Yes |
| `Cancelled` | Donor canceled before approval | Yes |
| `Rejected` | Recipient rejected the transfer | Yes |
| `Expired` | Recipient didn't act before `expiresAt` | Yes |
| `Failed` | Settlement ended unsuccessfully | Yes |

`provisioningState` reports the latest ARM operation rather than the transfer outcome. For example, cancel, reject, and expiry can have `provisioningState: Canceled`, while `transferStatus` provides the specific reason.

Pending transfers currently expire after 30 days. Use the server-provided `properties.expiresAt` rather than calculating the deadline because preview behavior can change.

## Duplicate and concurrent transfers

Quota Transfer doesn't currently provide idempotent create behavior. Submitting another request with an existing `transferName` returns `409 Conflict`, even when the request properties are unchanged.

Only one nonterminal transfer can exist for the same:

```text
donor subscription + recipient subscription + target provider + region + resourceName
```

A request for dimensions that already have a nonterminal transfer returns `409 TransferAlreadyInFlight`. Wait for the active transfer to become terminal, or cancel it if it's still pending.

If a submit request times out, don't immediately repeat the `PUT`. First, get the donor resource using the same `transferName`. If it exists, continue using its `transferId`. Submit again only after confirming that the original request wasn't created.

## Common errors

| HTTP status or code | Cause | Action |
| --- | --- | --- |
| `400 Bad Request` | Invalid name, body, amount, subscription, quota resource, or missing `If-Match` | Correct the request using the response error details |
| `403 AuthorizationFailed` | Caller lacks `Microsoft.Quota/quotas/write` at the required subscription scope | Assign the permission on the donor or recipient subscription, as applicable |
| `404 Not Found` | Transfer doesn't exist, scope is incorrect, or preview/resource type is unavailable | Verify subscription, provider, region, identifier, and preview enrollment |
| `409 Conflict` | The transfer name already exists or the requested action is invalid for the current state | Get the existing transfer and review its status |
| `409 TransferAlreadyInFlight` | The same quota bucket already has a nonterminal transfer | Wait, reject, or cancel the existing transfer |
| `412 SourceResourceModified` | `sourceEtag` is stale or incorrect | Get the incoming transfer and retry with its latest `properties.sourceEtag` |
| `429` or transient `5xx` | Throttling or temporary service failure | Honor `Retry-After`. After an uncertain submit result, get by `transferName` before submitting again |

Admission-time donor headroom validation is best effort. Usage can change after submission, so settlement can still fail if the donor no longer has enough unused quota.

## Integration recommendations

- Persist `transferName`, `transferId`, both subscription IDs, target provider, region, and `resourceName`.
- Use a unique `transferName` for each new transfer.
- After a submit timeout, get by `transferName` to determine whether the transfer was created before issuing another `PUT`.
- Follow `nextLink` for every list operation.
- Follow only the polling URL returned by a long-running response.
- Treat unknown nonterminal preview statuses as pollable rather than successful.
- Verify the final limits through the target provider's quota or usage API after `Completed`.
- Never delete a transfer as a way to undo it.

## REST API reference

- [Quota Transfers - List](/rest/api/quota/quota-transfers/list?view=rest-quota-2026-09-01-preview)
- [Quota Transfers - Cancel](/rest/api/quota/quota-transfers/cancel?view=rest-quota-2026-09-01-preview)
- [Quota Transfers - Delete](/rest/api/quota/quota-transfers/delete?view=rest-quota-2026-09-01-preview)
- [Incoming Quota Transfers - Get](/rest/api/quota/incoming-quota-transfers/get?view=rest-quota-2026-09-01-preview)
- [Incoming Quota Transfers - Approve](/rest/api/quota/incoming-quota-transfers/approve?view=rest-quota-2026-09-01-preview)
- [Incoming Quota Transfers - Reject](/rest/api/quota/incoming-quota-transfers/reject?view=rest-quota-2026-09-01-preview)
- [Incoming Quota Transfers - List by subscription](/rest/api/quota/incoming-quota-transfers/list-by-subscription?view=rest-quota-2026-09-01-preview)
