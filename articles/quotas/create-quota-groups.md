---
title: Create and delete Quota Group
description: Learn how to create and delete quota groups. 
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/07/2025
---

# Create Quota Groups object to manage quota for a group of subscriptions
A Quota group is an ARM object that can be used to manage quota at the group level and self-distribute quota between subscriptions added to the group. The first step is to create a Quota Group object under a Management Group. 

## Considerations
- Requires the *GroupQuota Request Operator* role on the Management Group used to create Quota group
<!-- Please write at least a sentance to introduce this subsection. -->

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
Example using `az rest`:
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

1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select **Create** button to go to Create Quota Group view
5. In the **Basics** tab, fill out required fields such name of Quota group. Quota Group name cannot contain any special characters and or spaces.
6. Select **Management group**, if the default Management group is not the desired one, select **Change management group**, then select **Next** button.
7. In the **Subscription selection** tab, select subscriptions to be added to quota group. Subscription is greyed out and unable to be selected if it already belongs to existing Quota Group, then  select **Next** button.
8. In **Review + create** tab review details of Quota group before creation. Under **Basics** name of Quota Group is displayed, under **Group selection** the selected Management group and subscriptions are displayed
--- 

## Delete a Quota Group
To delete a Quota Group
- Requires the *GroupQuota Request Operator* role on the Management Group used to DELETE a Quota group
- All subscriptions  must be removed from Quota Group before deletion  
### [REST API](#tab/rest-2)

```http
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupQuotaName}?api-version=2025-03-01
```

### [Azure portal](#tab/portal-2)

To delete a Quota Group through the Azure portal 
1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.
2. Under settings in left hand side select **Quota groups**.
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group
4. Select the box next to the quota group to be deleted
5. Select the **Delete**  button on the top
6. In the **Delete a quota group** view the Quota Group to be deleted is shown, along with the list of **Dependent subscription to be removed from quota group**
7. Select the **Delete** button on the bottom, **Delete confirmation** notification pops up, notifying **Deleting this quota group is a permanent action and cannot be undone**, by selecting delete you are giving permission for Azure to remove your subscriptions if there are any from group and then group will be deleted
8. Notification on right hand side will surface with **Your quota group has been deleted** once group is deleted
--- 

