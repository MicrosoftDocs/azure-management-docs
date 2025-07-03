---
title: Share Quota Across Subscriptions with Azure Quota Groups
description: Learn how to share quota across Azure subscriptions with Azure Quota Groups to reduce the number of quota transactions.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 05/29/2025
---


## Submit Quota Group Limit increase request
One of the key benefits of Quota Group offering is the ability to submit Quota Group Limit increase requests rather than at the per subscription level. If your group limit request is approved you can then follow steps to allocate/transfer quota from group to target subscriptions  for a given region x VM family.  
- Require *GroupQuota Request Operator* role on the Management Group to submit Quota Group limit increase request.
- Customers can submit Quota Group limit increase requests for a region x VM family combination, and if approved, quota will be stamped on the specified Quota GroupID.  
- Quota Group Limit increase requests undergo the same checks as subscription level requests. Value should be absolute value of the new desired amount.   
- If Quota Group  Limit request is rejected then customer must submit support ticket via the self-serve Quota group request blade.  
- Support tickets for Quota Groups will be created based on a preselected subscriptionID within the group, the customer has the ability to edit the subID when updating request details.

### [REST API](#tab/rest-5)
```http
PATCH https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01
```
```json
{
  "properties": {
    "value": [
      {
        "properties": {
          "resourceName": "standardddv4family",
          "limit": 50,
          "comment": "comments"
        }
      }
    ]
  }
}
```
Example using `az rest`
- I submit PATCH Quota Group limit increase request of 50 cores for ***standardddv4family*** in centralus
- Use the groupQuotaOperationsStatus ID in my response header to validate the status of request in next section

```http
az rest --method patch --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01" --body '{"properties":{"value":[{"properties":{"resourceName":"standardddv4family","limit":20,"comment":"comments"}}]}}' --verbose
```


```json
{
  "properties": {
    "value": [
      {
        "properties": {
          "resourceName": "standardddv4family",
          "limit": 50,
          "comment": "comments"
        }
      }
    ]
  }
}
```
Example response Quota Group increase request
```json
Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaOperationsStatus/6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2?api-version=2025-03-01
Retry-After: 30 
Response Content
```

## GET groupQuotaLimit request status
Since groupQuotaLimit request is async operation, capture status of request using groupQuotaOperationsStatus ID from response header when submitting limit increase request
```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/{requestId}?api-version=2025-03-01
```


Example using `az rest`
I used the groupQuotaOperationsStatus ID from my PATCH Quota Group Limit increase request of 50 cores for ***standardddv4family*** in centralus succeeded
```http
az rest --method get --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2?api-version=2025-03-01"
```
Sample response of approved Quota Group increase request
```json
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2",
  "name": "6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2",
  "properties": {
    "provisioningState": "Succeeded",
    "requestProperties": {
      "requestSubmitTime": "2025-05-06T17:57:00.0001431+00:00"
    },
    "requestedResource": {
      "properties": {
        "limit": 20,
        "name": {
          "localizedValue": "STANDARDDDV4FAMILY",
          "value": "STANDARDDDV4FAMILY"
        },
        "provisioningState": "Succeeded",
        "region": "centralus"
      }
    }
  },
  "type": "Microsoft.Quota/groupQuotas/groupQuotaRequests"
}
```
### GET groupQuotaLimits  

Validate that the correct number of cores  were transferred from source subscription to group or that your group limit increase request was approved. Consider the below when interpreting the API response. 
- Available limit = how many  cores do I have at group level to distribute  
- Limit = how many cores have been explicitly requested and approved/stamped on your group via quota increase requests  
- Quota allocated = how many cores the sub has been allocated from group, ‘-‘ value indicates cores have been de-allocated from sub to group 

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```
Example using `az rest`:
```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```

Example using `az rest`:
I do a GET group limit for my quota group in centralus.  
- For the resource standardddv4family my **availableLimit** = 50 cores which match the number of cores  I requested and got approved at the group level
- The **Limit** = 40 because even though I submitted an increase for 50 I already had 10 cores at the group level from quota transfer example, and Azure only stamped an additional 40 cores 
- The **quotaAllocated** = -10 because I transferred 10 cores from source sub to group from previous section
```http
az rest --method get --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01&$filter=resourceName eq standardddv4family

```

Az rest sample response:

```json
az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01&$filter=resourceName eq 'standardddv4family'"
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus",
  "name": "eastus",
  "properties": {
    "nextLink": "",
    "provisioningState": "Succeeded",
        "value": [
      {
        "properties": {
          "allocatedToSubscriptions": {
            "value": [
              {
                "quotaAllocated": -10,
                "subscriptionId": "226818a0-4fa2-4c2d-be7f-03b9b92ab3a2"
              }
            ]
          },
          "availableLimit": 50,
          "limit": 40,
          "name": {
            "localizedValue": "standardddv4family",
            "value": "standardddv4family"
          },
          "resourceName": "standardddv4family",
          "unit": "Count"
        }
      }
    ]
  },
  "type": "Microsoft.Quota/groupQuotas/groupQuotaLimits"
}

```
