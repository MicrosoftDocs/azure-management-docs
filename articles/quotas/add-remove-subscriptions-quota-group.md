---
title: Add or remove subscriptions from a Quota Group
description: Learn how to add and or remove subscriptions from a Quota Group. 
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/23/2025
---

# Add or remove subscriptions from a Quota Group

You can add or remove subscriptions from a Quota Group after the group is created.    

## Add subscriptions to a Quota Group
When added to the group, subscriptions carry their existing quota and usage. The subscriptions' quota isn't manipulated when added to a group. Subscription quota remains separate from the group limit.

### [REST API](#tab/rest-1)
To add subscriptions from the Quota Group using the REST API, make a `PUT` request to the following endpoint:

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01
```

The following example uses `az rest`:

```json
az rest --method put --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01"
```

Sample response:

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

### [Azure portal](#tab/portal-1)
Add subscriptions from the Quota Group through the Azure portal. Subscriptions can only belong to one quota group at a time.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group.
5. Select **Edit** or **Add** button under **Subscriptions added** column.
   1. In the Edit subscriptions, view the existing subscriptions listed and select the **Add subscription** button.
   2. In the Add Subscriptions view, select the desired subscription and select **Save**.
6. You can search for the subscription in the search box at the top of the blade. Subscriptions are greyed out if they belong to an existing group and indicate 'No' under **Available to add** column.
7. A notification indicates that the subscription was successfully added and the Edit Subscriptions view is updated with the added subscriptions.

---

## Remove subscriptions from a Quota Group
At the moment of removal, subscriptions carry their existing quota and usage. The group limit isn't manipulated based on subscription removal.  

### [REST API](#tab/rest-2)
To remove subscriptions from the Quota Group using the REST API, make a `DELETE` request to the following endpoint:

```http
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01
```

### [Azure portal](#tab/portal-2)
Remove subscriptions from the Quota Group through the Azure portal. Subscriptions can only belong to one quota group at a time.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group.
5. Select **Edit** button under **Subscriptions added** column.
6. In the Edit subscriptions view, select the trashcan icon for the subscription you want to remove.
7. A pop-up asks whether you're sure about removing the selected subscription, select **remove**.
8. A notification indicates that the subscription was successfully removed and the Edit Subscriptions view is updated with the latest list of subscriptions.

--- 
