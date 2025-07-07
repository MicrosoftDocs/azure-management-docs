---
title: Quota Group limit increase requests
description: Learn how to submit a quota group limit increase request.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/07/2025
---


# Submit Quota Group Limit increase request
One of the key benefits of Quota Group offering is the ability to submit Quota Group Limit increase requests rather than at the per subscription level. If your group limit request is approved, you can self-distribute cores from group to target subscriptions for a given region x Virtual Machine family.  


## Considerations
- Require *GroupQuota Request Operator* role on the Management Group to submit Quota Group limit increase request.
- Customers can submit Quota Group limit increase requests for a region x Virtual Machine family combination, and if approved, quota is stamped on the specified Quota GroupID.  
- Quota Group Limit increase requests undergo the same checks as subscription level requests. Value should be absolute value of the new desired amount.   
- If Quota Group  Limit request is rejected, then customer must submit support ticket via the self-serve Quota group request blade.  
- Support tickets for Quota Groups are created based on a preselected subscriptionID within the group. The customer has the ability to edit the subID when updating request details.

   
## Quota Group limit increase request 
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

## Quota Group limit request status
Quota Group limit request is an async operation, capture status of request using groupQuotaOperationsStatus ID from response header when submitting limit increase request
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
### Snapshot of Quota Group limit

Validate that the correct number of cores  were transferred from source subscription to group or that your group limit increase request was approved. Consider this when interpreting the API response. 
- Available limit = how many cores do I have at group level to distribute  
- Limit = how many cores are explicitly requested and approved/stamped on your group via quota increase requests  
- Quota allocated = how many cores the are allocated from subscription group, ‘-‘ value indicates cores are de-allocated from subscription to group 

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```
Example using `az rest`:
```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```

Example using `az rest`:
I do a GET group limit for my quota group in centralus.  
- For the resource standardddv4family my **availableLimit** = 50 cores, which match the number of cores I requested and got approved at the group level.  
- The **Limit** = 40 because I already had 10 cores at the group level from a previous quota transfer. When submitting Quota group limit increase request and getting approved, Azure only stamped 40 more cores.  
- The **quotaAllocated** = -10 because I transferred 10 cores from source sub to group from previous section.
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
--- 
## Submit Quota Group Limit increase and file support ticket via Azure Portal
This sections covers how to submit Quota Group Limit increase via portal and file support ticket if request fails. 
- If Quota Group  Limit request is rejected via API or portal; then customer must submit support ticket via the self-serve Quota group request portal blade.  
- Support tickets for Quota Groups are created based on a preselected subscriptionID within the group. The customer has the ability to edit the subID when updating request details. Even though ticket is created using subID, if approved the quota will be stamped at the group level.  
- User requires at a minimum the **Support request contributor role** to create support ticket on subscription in the group.  
- Quota Groups addresses the quota management pain point, it does not address the regional and or zonal access pain point. To get region and or zonal access on subscriptions, [see region access request process](/troubleshoot/azure/general/region-access-request-process). Quota transfers between subscriptions and deployments fail unless regional and or zonal access is provided on the subscription.    

1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side, select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select **Quota group** from list of Quota Groups
5. In the Quota Group resources view, there is the list of Quota group resources by region by Group quota (limit)
6. Use the filters to select **Region** and or **VM Family**, you can also search for region and or VM family in the search bar
7. Select the checkbox to the desired **Quota group resource**, then select the **Increase group quota** button at the top of page
8. On right side view the **New quota request** blade with selected regions at the top, with details on the selected **Quota group resource**, the **Current group quota** value, and under **New group quota** column enter the  absolute value of desired net new group limit. For example, I want 20 cores assigned at group for DSv3 in Australia Central, I enter 20 under **New group quota**
9. Select **Submit** button, notification **We are reviewing your request to adjust the quota** this may take up to ~3 minutes to complete
10. If successful, the **New quota request** view shows the selected **Quota Group resource** by location status of request, the **Increase** value and **New limit**
11. Refresh the Quota Group resources view to view latest **Group quota** / group limit
12. If Quota Group limit increase was rejected notification **We were unable to adjust your quota** surfaces in the top right corner
13. Select the **Generate a support ticket** button to start process of creating support ticket
14. In the **Request details** view  **Deployment model** as **Resource Manager**, request details view surface the Quota Group name, Management GroupID, Quota Group resource, Location selected, and the Desired increase value, select **Save and Continue** button
16. In **Additional details** view, select required options **Advance diagnostic information** and **Preferred contact method** and select **Next**
17. Review details in **Review + Create** view and select **Create** button, notification **New Support Request** in top right corner has ticketID and link
18. To view request details return to **Quotas** blade and select the **Request** tab under the **Overview** page, see the list of quota requests, you may also search and go to **Help + Support** blade and view request under **Recent support requests** table
--- 
