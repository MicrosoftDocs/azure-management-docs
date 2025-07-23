---
title: Create and delete Azure Quota Groups to manage quota for a group of subscriptions
description: Learn how to create and delete Azure Quota Group objects to manage quota for a group of subscriptions. 
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/23/2025
---

# Create or delete Azure Quota Groups to manage quota for a group of subscriptions

An Azure Quota Group is an Azure Resource Manager (ARM) object that can be used to manage quota at the group level. This object can self-distribute quota between subscriptions added to the group.   

## Considerations
- Creating or deleting Quota Groups requires the *GroupQuota Request Operator* role on the Management Group.
- All subscriptions must be removed from your Quota Group before deletion. 

## Create Quota Groups 
Create a Quota Group object under a Management Group.

### [REST API](#tab/rest-1)
To create a Quota Group using the REST API, make a `PUT` request to the following endpoint:

```http
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01

{
  "properties": {
    "displayName": "GQdemo"
  }
}
```

The following example uses the `az rest` command:

```json
az rest --method put --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01 --body '{
  "properties": {
    "displayName": "GQdemo"
  }
}'
```

Sample response: 

```
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}",
  "name": "{groupquota}",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas"
}
```

### [Azure portal](#tab/portal-1)
Create a Quota Group through the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group.
5. Select the **Create** button.
6. In the **Basics** tab, fill out required fields. The Quota Group *name* can't contain any special characters or spaces.
7. Select **Management group**. If the default Management group is not the desired one, select **Change management group**, then select **Next** button.
8. In the **Subscription selection** tab, select subscriptions to be added to the quota group. Subscription is greyed out and unable to be selected if it already belongs to an existing Quota Group.
9. Select the **Next** button.
10. In the **Review + create** tab, review details of your Quota group before creation.

---

## Delete Quota Groups
All subscriptions must be removed from a Quota Group before deletion. 

### [REST API](#tab/rest-2)
To delete a Quota Group using the REST API, make a `DELETE` request to the following endpoint:

```http
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupQuotaName}?api-version=2025-03-01
```

### [Azure portal](#tab/portal-2)
Delete a Quota Group through the Azure portal.

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group.
5. Select the box next to the quota group to be deleted.
6. Select the **Delete** button at the top. 
7. In the **Delete a quota group** view, the Quota Group to be deleted is shown along with a list of **Dependent subscription to be removed from quota group**.
8. Select the **Delete** button on the bottom
9. A **Delete confirmation** pops up, notifying **Deleting this quota group is a permanent action and cannot be undone**. By selecting delete you are giving permission for Azure to remove your subscriptions. If there are any from a group, then the group will also be deleted.
10. A notification will pop up stating **Your quota group has been deleted** once the group is deleted. 
--- 

