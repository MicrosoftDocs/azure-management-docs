---
title: Quota Group scenario and example walkthrough
description: Walks through Quota Group scenario and how to use quota group operations to self-manage quota.
author: yaya-5
ms.author: yaalanis
ms.service: Quota Groups
ms.topic: how-to #Don't change
ms.date: 07/15/25

#customer intent: As a <role>, I want <what> so that <why>.

---

<!-- --------------------------------------

- Use this template with pattern instructions for:

How To

- Before you sign off or merge:

Remove all comments except the customer intent.

- Feedback:

https://aka.ms/patterns-feedback

-->

# Quota Group example walkthrough

<!-- Required: Article headline - H1

Identify the product or service and the task the
article describes.

-->
Customers begin by creating a Quota Group object and adding the relevant subscriptions to it. This enables them to manage quota collectively across those subscriptions. Instead of submitting separate quota requests for each subscription, customers can submit a single group-level limit request for an entire application or workload. Once the quota is approved, they can distribute it across subscriptions to support successful deployments. Additionally, customers can group existing subscriptions—such as those tied to a specific application or department—and seamlessly transfer or reallocate quota between them using the Quota Group object, all without needing to file a support ticket.
This covers a Quota Group scenario and walks through an end-to-end Quota Group example. 

## Quota Group Scenario
Anna is a lead developer on the Contoso Cloud Readiness team. Contoso manages hundreds of Azure subscriptions and continues to stamp new ones to support business continuity. One of Anna’s key challenges is the inability to deploy new VMs due to quota limitations. Managing quotas at the individual subscription level is inefficient—each time a subscription runs out of quota or a new one is created, Anna must file a support ticket.
Anna explores Azure’s new Quota Groups feature, which streamlines quota management by allowing it to be handled at the group level. With the Quota Group APIs, she can easily create a group tailored to a new application she plans to deploy. Instead of submitting multiple quota requests for each subscription, Anna submits a single group-level request for the necessary compute resources. Once the quota is approved, she can allocate cores from the group to individual subscriptions, ensuring smooth and consistent deployments. As her application scales, Anna can continue to manage quotas efficiently by submitting group-level requests, eliminating the need for repetitive, subscription-specific requests.
With Quota Groups, Anna can also support and maintain existing applications more effectively by grouping subscriptions and redistributing already procured quota among them. For instance, if one of her current subscriptions has unused quota, she can seamlessly transfer it to another subscription that needs it. This flexibility allows Anna to keep her deployments running smoothly—without the need to file a support ticket—by reallocating resources where they’re needed most.

The reader will learn how to complete the following operations via API and portal.
1.	Creating a Quota Group object to manage quota for a group of subscription(s)
2.	Adding subscription to Quota Group object 
3.	Submit Quota Group Limit increase request
  a.	GET groupQuotaLimit request status
  b.	GET groupQuotaLimits
4.	Transferring quota from source subscription to target subscription 
  a.	View request status for quotaAllocation request
  b.	View quota allocation snapshot for subscription in Quota Group
5.	GroupLimit request is escalated and support ticket is filed via portal

<!-- Required: Introductory paragraphs (no heading)

Write a brief introduction that can help the user
determine whether the article is relevant for them
and to describe the task the article covers.

-->

## Prerequisites
•	Before you can use the Quota Group feature, you must:  
•	Register the Microsoft.Quota and Microsoft.Compute resource provider on all relevant subscriptions before adding to a Quota Group.  
•	A Management Group (MG) is needed to create a Quota Group. Your group inherits quota write and or read permissions from the Management Group. Subscriptions belonging to another MG can be added to the Quota Group.  
•	Certain permissions are required to create Quota Groups and to add subscriptions.  

## Limitations
•	Available only for Enterprise Agreement or Microsoft Customer Agreement.  
•	Supports IaaS compute resources only.  
•	Available in public cloud regions only.  
•	Management Group deletion results in the loss of access to the Quota Group limit. To clear out the group limit, allocate cores to subscriptions, delete subscriptions, then the Quota Group object before deletion of Management Group. In the even that the MG is deleted, access your Quota Group limit by recreating the MG with the same ID as before.  


<!-- Optional: Prerequisites - H2

If included, "Prerequisites" must be the first H2 in the article.

List any items that are needed to complete the How To,
such as permissions or software.

If you need to sign in to a portal to complete the How To, 
provide instructions and a link.

-->

## Creating a Quota Group object to manage quota for a group of subscriptions

•	Create a Quota Group object to be able to do quota transfers between subscriptions and submit Quota Group increase requests.  
•	Requires the GroupQuota Request Operator role on the Management Group used to create Quota group.  


### [REST API](#tab/rest-1)
1. Anna submits a PUT call to create Quota Group GQdemo

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01 

{
  "properties": {
    "displayName": "GQdemo"
  }
}
```
Example using `az rest`:
```json
az rest --method put --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01 --body '{
  "properties": {
    "displayName": "GQdemo"
  }
}'
```
2. Anna gets below response, QuotaGroup ‘GQdemo’ was successfully created
```
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo",
  "name": "GQdemo",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas"
}
```

### [Azure portal](#tab/portal-1)
Create a Quota Group through the Azure portal.

1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select **Create** button to go to Create Quota Group view
5. In the **Basics** tab, fill out required fields such name of Quota group. Quota Group name cannot contain any special characters and or spaces.
6. Select **Management group**, if the default Management group is not the desired one, select **Change management group**, then select **Next** button.
7. In the **Subscription selection** tab, select subscriptions to be added to quota group. Subscription is greyed out and unable to be selected if it already belongs to existing Quota Group, then  select **Next** button.
8. In **Review + create** tab review details of Quota group before creation. Under **Basics** name of Quota Group is displayed, under **Group selection** the selected Management group and subscriptions are displayed
---
<!-- Required: Steps to complete the task - H2

In one or more H2 sections, organize procedures. A section
contains a major grouping of steps that help the user complete
a task.

Begin each section with a brief explanation for context, and
provide an ordered list of steps to complete the procedure.

If it applies, provide sections that describe alternative tasks or
procedures.

-->
## Add or remove subscriptions from a Quota Group
This section covers how to add subscriptions after the Quota group is created. When added to the group, subscriptions carry their existing quota and usage. The subscriptions' quota is not manipulated when added to a group. Subscription quota remains separate from the group limit.

### [REST API](#tab/rest-2)
1.	Anna submits PUT call via azrest to add subscription(s), only single subscription can be added a time, she adds two subscriptions to GQdemo  
  a.	Subscription1 (dbd56dd1-1e41-4dff-a289-b815fc1acd96)  
  b.	Subscription2 (c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7)  

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01
```

Example using `az rest`:
```json
az rest --method put --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/subscriptions/dbd56dd1-1e41-4dff-a289-b815fc1acd96?api-version=2025-03-01"
```
2. Anna gets the below response
```json
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/subscriptions/dbd56dd1-1e41-4dff-a289-b815fc1acd96",
  "name": "dbd56dd1-1e41-4dff-a289-b815fc1acd96",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas/subscriptions"
}
```
3. Anna submits a LIST API call to get list of subscription(s) registered to GQdemo  
  a.	Subscription1 (dbd56dd1-1e41-4dff-a289-b815fc1acd96)  
  b.	Subscription2 (c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7)  

```json
az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/subscriptions?api-version=2025-03-01"
{
  "value": [
    {
      "Properties": {
        "provisioningState": "SUCCEEDED",
        "subscriptionId": "dbd56dd1-1e41-4dff-a289-b815fc1acd96"
      },
      "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/subscriptions/dbd56dd1-1e41-4dff-a289-b815fc1acd96",
      "name": "dbd56dd1-1e41-4dff-a289-b815fc1acd96",
      "type": "Microsoft.Quota/groupQuotas/subscriptions"
    },
    {
      "Properties": {
        "provisioningState": "SUCCEEDED",
        "subscriptionId": "c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7"
      },
      "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/subscriptions/c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7",
      "name": "c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7",
      "type": "Microsoft.Quota/groupQuotas/subscriptions"
    }
  ]

```
### [Azure portal](#tab/portal-2)
Add or remove subscriptions from the Quota Group through the Azure portal. Subscriptions can only belong to one quota group at a time.

To add subscriptions from Quota Group through portal.
1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side, select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select **Edit** or **Add** button under **Subscriptions added** column
5. In the Edit subscriptions, view the existing subscriptions are listed, select **Add subscription** button
6. In the Add Subscriptions view, select the desired subscription and select **Save**. You can search for subscription in search box at the top of blade. subscriptions  will be greyed out if they belong to existing group and indicate 'No' under **Available to add** column.
7. Notification should indicate that subscriptions  was successfully added and the Edit Subscriptions view is updated with the added subscriptions 
   
To remove subscription from Quota Group through portal. 
1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side, select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select **Edit** or **Add** button under **Subscriptions added** column
5. In the Edit subscriptions, view the existing subscriptions are listed, select trashcan icon for the subscription you'd like to remove
6. Notification pops up, it asks whether you're sure about removing the selected subscription, select **remove**
7. Notification should indicate that subscriptions  was successfully removed and the Edit Subscriptions view is updated with the latest list of subscriptions 
---

## Submit Quota Group limit increase
One of the key benefits of Quota Group offering is the ability to submit Quota Group Limit increase requests rather than at the per subscription level. If your group limit request is approved you can then follow steps to allocate/transfer quota from group to target subscriptions for a given region x VM family.  
•	Require GroupQuota Request Operator role on the Management Group to submit Quota Group limit increase request.  
•	Customers can submit Quota Group limit increase requests for a region x VM family combination, and if approved, quota will be stamped on the specified Quota GroupID.  
•	Quota Group Limit increase requests undergo the same checks as subscription level requests. Value should be absolute value of the new desired amount.  
•	If Quota Group Limit request is rejected then customer must submit support ticket via the self-serve Quota group request blade.  
•	Support tickets for Quota Groups will be created based on a preselected subscriptionID within the group, the customer has the ability to edit the subID when updating request details.  

1.	Anna does PATCH call for group limit increase request for 50 cores for standardddv4family in centralus
```http
PATCH https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01

{
  "properties": {
    "value": [
      {
        "properties": {
          "limit": 50,
          "resourceName": "standardddv4family",
          "comment": "Contoso requires more quota."
        }
      }
```
Example using `az rest`:
```json
az rest --method patch --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01" --body '{"properties":{"value":[{"properties":{"resourceName":"standardddv4family","limit":50,"comment":"comments"}}]}}' –verbose
```
2. Anna gets the below response and captures the groupQuotaOperationStatus ID which she will use to get the request status in next step
```json
Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaOperationsStatus/e1933713-dea6-4f31-8431-116c5285c5c1?api-version=2025-03-01
Retry-After: 30 
Response Content
```
3. Anna submits GET groupLimit request status using groupQuotaOperationsStatus ID(e1933713-dea6-4f31-8431-116c5285c5c1) from response header when submitting limit increase request
4. Anna gets the below request status response and sees the provisioningState = Succeeded, which means the request for 50 cores in central us for Standardddv4 was approved
```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/{requestId}?api-version=2025-03-01
```
Example using `az rest`:
```json
az rest --method get --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/groupQuotaRequests/e1933713-dea6-4f31-8431-116c5285c5c1?api-version=2025-03-01"
{
  "id": "/providers/Microsoft.Management/managementGroups/YayatestMG/providers/Microsoft.Quota/groupQuotas/GQdemo/groupQuotaRequests/e1933713-dea6-4f31-8431-116c5285c5c1",
  "name": "e1933713-dea6-4f31-8431-116c5285c5c1",
  "properties": {
    "provisioningState": "Succeeded",
    "requestProperties": {
      "requestSubmitTime": "2025-06-23T19:05:26.6535642+00:00"
    },
    "requestedResource": {
      "properties": {
        "limit": 50,
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
5. Anna submit a GETgroupQuotaLimit request to view snapshot of the group limit for GQdemo group. Consider the below when interpreting the API response.  
   a. Available limit = how many cores do I have at group level to distribute  
     i. GQdemo Available limit = 50 cores because there are 50 cores available at group level from the request previously submitted  
   b.	Limit = how many cores have been explicitly requested and approved/stamped on your group via quota increase requests  
     i. GQdemo Limit = 50 because I requested and was approved 50 cores
   c. how many cores the sub has been allocated from group, ‘-‘ value indicates cores have been de-allocated from sub to group
     i. quotaAllocated for Subscription1 (dbd56dd1-1e41-4dff-a289-b815fc1acd96) = 0 and Subscription2 ((c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7) = 0 because I have not completed any quota             transfers yet

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```
Example using `az rest`:
```json
az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01&%24filter=resourceName%20eq%20'standardddv4family'" –debug
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus",
  "name": "centralus",
  "properties": {
    "nextLink": "",
    "provisioningState": "Succeeded",
    "value": [
      {
        "properties": {
          "allocatedToSubscriptions": {
            "value": [
              {
                "quotaAllocated": 0,
                "subscriptionId": "dbd56dd1-1e41-4dff-a289-b815fc1acd96"
              },
              {
                "quotaAllocated": 0,
                "subscriptionId": "c54a40cd-9a9c-4c70-bc2a-a532c75e7ca7"
              }
            ]
          },
          "availableLimit": 50,
          "limit": 50,
          "name": {
            "localizedValue": "standardddv4family",
            "value": "standardddv4family"
          },
          "resourceName": "standardddv4family",
          "unit": "Count"
        }
      }

```

## GroupLimit request is escalated
•	If Quota Group Limit request is rejected via API or portal; then customer must submit support ticket via the self-serve Quota group request portal blade.  
•	Support tickets for Quota Groups will be created based on a preselected subscriptionID within the group, the customer has the ability to edit the subID when updating request details. Even though ticket is created using subID, if approved the quota will be stamped at the group level.  
•	Anna requires at a minimum the Support request contributor role to create support ticket on subscription in the group.  
### [REST API](#tab/rest-3)
1. Anna submit group limit request for 10k; PATCH call for group limit increase request for 10k  cores for standardddv4family in centralus

Example using `az rest`:
```json
az rest --method patch --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/resourceProviders/Microsoft.Compute/groupQuotaLimits/centralus?api-version=2025-03-01" --body '{"properties":{"value":[{"properties":{"resourceName":"standardddv4family","limit":10000,"comment":"comments"}}]}}' –verbose
```
2. Anna gets the below response and captures the groupQuotaOperationStatus ID which she will use to get the request status in next step
```json
Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaOperationsStatus/3e01a682-1349-4ac7-813e-b9e1b95461f1?api-version=2025-03-01
Retry-After: 30 
Response Content
```
3. Anna submit GET groupLimit request status using groupQuotaOperationsStatus ID(3e01a682-1349-4ac7-813e-b9e1b95461f1) from response header when submitting limit increase request
4. Anna gets the below request status response and sees the provisioningState = Failed, which means the request for 10k cores in central us for Standardddv4 was NOT approved and Anna must submit a support ticket via the Azure portal.
```json
az rest --method get --uri "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/GQdemo/groupQuotaRequests/3e01a682-1349-4ac7-813e-b9e1b95461f1?api-version=2025-03-01"
{
  "id": "/providers/Microsoft.Management/managementGroups/YayatestMG/providers/Microsoft.Quota/groupQuotas/GQdemo/groupQuotaRequests/3e01a682-1349-4ac7-813e-b9e1b95461f1",
  "name": "3e01a682-1349-4ac7-813e-b9e1b95461f1",
  "properties": {
    "provisioningState": "Failed",
    "requestProperties": {
      "requestSubmitTime": "2025-06-23T19:37:56.9442264+00:00"
    },
    "requestedResource": {
      "properties": {
        "limit": 10000,
        "name": {
          "localizedValue": "STANDARDDDV4FAMILY",
          "value": "STANDARDDDV4FAMILY"
        },
        "provisioningState": "Failed",
        "region": "centralus"
      }
    }
  },
  "type": "Microsoft.Quota/groupQuotas/groupQuotaRequests"
}
```
### [Azure portal](#tab/portal-3)

## Clean up resources

<!-- Optional: Steps to clean up resources - H2

Provide steps the user can take to clean up resources that
they might no longer need.

-->

## Next step -or- Related content

> [!div class="nextstepaction"]
> [Next sequential article title](link.md)

-or-

* [Related article title](link.md)
* [Related article title](link.md)
* [Related article title](link.md)

<!-- Optional: Next step or Related content - H2

Consider adding one of these H2 sections (not both):

A "Next step" section that uses 1 link in a blue box 
to point to a next, consecutive article in a sequence.

-or- 

A "Related content" section that lists links to 
1 to 3 articles the user might find helpful.

-->

<!--

Remove all comments except the customer intent
before you sign off or merge to the main branch.

-->
