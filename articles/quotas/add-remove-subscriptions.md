---
title: Add and remove subscriptions
description: Learn how to add and or remove subscriptions from a quota group. 
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/07/2025
---
# Add subscriptions to a Quota Group
- This section covers how to add subscriptions  after the Quota group is created.  
- When added to the group, subscriptions carry their existing quota and usage.  
- The subscriptions' quota is not manipulated when added to a group. Subscription quota remains separate from the group limit.  

<!-- Please write at least a sentance to introduce this subsection. -->
<!-- Consider breaking add and remove into their own seperate sections. -->

### [REST API](#tab/rest-3)
To add subscriptions from the Quota Group using the REST API, make a `PUT` request to the following endpoint:

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01
```
Example using az rest:
```json
az rest --method put --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01"
```
Sample response
```
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}",
  "name": "dbd56dd1-1e41-4dff-a289-b815fc1acd96",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas/subscriptions"
}
```

## Remove subscriptions from a Quota Group
- This section covers how to remove subscriptions from the Quota Group.
-  At the moment of removal, subscriptions carry its existing quota and usage.  
-  The group limit is not manipulated based on subscription removal.  

To remove subscriptions from the Quota Group using the REST API, make a `DELETE` request to the following endpoint:

```http
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01

```

### [Azure portal](#tab/portal-3)
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
