---
title: Submit an Azure Quota Group limit increase request
description: Learn how to submit an Azure Quota Group limit increase request.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/23/2025
---

# Submit an Azure Quota Group limit increase request

One of the key benefits of the Azure Quota Group offering is the ability to submit limit requests at the Quota Group level, rather than per subscription level. If your group limit request is approved, you can self-distribute cores from the group to target subscriptions for a given region and Virtual Machine (VM) family.

## Considerations
- Requires *GroupQuota Request Operator* role on the Management Group to submit Quota Group limit increase request.
- You can submit Quota Group limit increase requests for a region and VM family combination. If approved, quota is stamped on the specified Quota `GroupID`.
- Quota Group limit increase requests undergo the same checks as subscription level requests.   
   
## Quota Group limit increase request 

### [REST API](#tab/rest-1)
To request a limit increase for a Quota Group using the REST API, make a `PATCH` request to the following endpoint:

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

The following example uses `az rest` after submitting a `PATCH` Quota Group limit increase request of 50 cores for *standardddv4family* in *centralus*. The example also uses the *groupQuotaOperationsStatus* ID in the response header to validate the status of the request in the [Quota Group limit request status](#quota-group-limit-request-status) section.

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

Example response:

```json
Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaOperationsStatus/6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2?api-version=2025-03-01
Retry-After: 30 
Response Content
```

### [Azure portal](#tab/portal-1)
Request a limit increase for a Quota Group through the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group.
5. Select a **Quota group** from list of Quota Groups.
6. The Quota Group resources view lists group resources by region and by Group quota (limit).
7. Use the filters to select **Region** and or **VM Family**.
8. Select the checkbox next to the desired **Quota group resource**, then select the **Increase group quota** button at the top of page.
9. On right side, view the **New quota request** blade with the following details: selected regions at the top, details on the selected **Quota group resource**, the **Current group quota** value. Under the **New group quota** column, enter the absolute value of desired net new group limit.
10. Select **Submit** button.
11. A **We are reviewing your request to adjust the quota** notification pops up. This process may take several minutes to complete.
12. If successful, the **New quota request** view shows the selected **Quota Group resource** by location status of request, the **Increase** value, and the **New limit**.
13. Refresh the Quota Group resources view to view latest **Group quota** group limit.

To learn how to get support for a rejected Quota Group limit increase request, see the [File a support ticket](#file-a-support-ticket) section.

---

## Quota Group limit request status

The Quota Group limit request is an async operation. This operation captures the status of the request using *groupQuotaOperationsStatus* ID from the response header when you submit a limit increase request.

### [REST API](#tab/rest-2)
Use REST API and make a `GET` request to the following endpoint:

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/{requestId}?api-version=2025-03-01
```

The following example uses `az rest` after submitting a successful `PATCH` Quota Group limit increase request of 50 cores for *standardddv4family* in *centralus*. The example also uses the *groupQuotaOperationsStatus* ID in the response header to validate the status of the request.

```http
az rest --method get --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/6c1cdfb8-d1ba-4ade-8a5f-2496f0845ce2?api-version=2025-03-01"
```

Sample response of an approved Quota Group limit increase request:

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

---

## Snapshot of Quota Group limit

Validate that the correct number of cores were transferred from source subscription to the group, or that your group limit increase request was approved. 

Consider the following when interpreting the API response: 
- **Available limit** is how many cores you have at the Quota Group level to distribute.
- **Limit** is how many cores are explicitly requested and approved or stamped on your group via quota increase requests.  
- **Quota allocated** is how many cores are allocated from the subscription group. A `-` value indicates cores are de-allocated from the subscription to the group. 

### [REST API](#tab/rest-3)
To view and validate a snapshot you your Quota Group limit using the REST API, make a `GET` request to the following endpoint:

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```

Example using `az rest`:
```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```

The following example uses `az rest` after submitting a successful `PATCH` Quota Group limit increase request of 50 cores for *standardddv4family* in *centralus*. The example also uses the *groupQuotaOperationsStatus* ID in the response header to validate the status of the request.

The following example uses `az rest`. For the resource *standardddv4family*, the **availableLimit** is `50` cores, which match the number of cores requested and approved at the group level. The **Limit** is `40` because there were already 10 cores at the group level from a previous quota transfer. When you submit a Quota Group limit increase request and get it approved, Azure only stamped 40 more cores. The **quotaAllocated** is `-10` because 10 cores were transferred from a source subscription to the group from the previous section example.

```http
az rest --method get --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01&$filter=resourceName eq standardddv4family
```

Sample response:

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

## File a support ticket

This section covers how to file a support ticket through the Azure portal if your Quota Groups limit increase request fails. 

- If a Quota Group limit request is rejected through REST API or Azure portal, then you must submit a support ticket via the self-serve Quota Group request blade in Azure portal.  
- Support tickets for Quota Groups are created based on a preselected `subscriptionID` within the group. You have the ability to edit the `subscriptionID` when updating request details.
- Even though a ticket is created using *subscriptionID*, if approved, the quota will be stamped at the group level.
- User requires at minimum the **Support request contributor role** to create support ticket on subscriptions in the group.  
- Quota Groups address the quota management pain point, it does not address the regional and or zonal access pain point. To get region and or zonal access on subscriptions, see [Region Access Request Process](/troubleshoot/azure/general/region-access-request-process). Quota transfers between subscriptions and deployments fail unless regional and or zonal access is provided on the subscription.    

The following Azure portal instructions continue from the steps outlined in the previous [Quota Group limit increase request 
](#quota-group-limit-increase-request) section. 

1. If the Quota Group limit increase was rejected, a notification of **We were unable to adjust your quota** pops up.  
1. Select the **Generate a support ticket** button to start the process of creating a support ticket.
1. In the **Request details** view, **Deployment model** as **Resource Manager**.
1. Request details view will surface the Quota Group name, Management GroupID, Quota Group resource, Location selected, and the Desired increase value.
1. Select the **Save and Continue** button.
1. In **Additional details** view, select the required options **Advance diagnostic information** and **Preferred contact method**, then select **Next**.
1. Review details in **Review + Create** view, then select the **Create** button.
1. A **New Support Request** notification will pop up, which has *ticketID* and a link.
1. To view request details, return to the **Quotas** blade and select the **Request** tab under the **Overview** page. See the list of quota requests.
1. You may also search and go to **Help + Support** blade and view request under **Recent support requests** table.

