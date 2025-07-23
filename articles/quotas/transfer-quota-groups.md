---
title: Transfer quota within an Azure Quota Group
description: Learn how to transfer quota within an Azure Quota Group.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/23/2025
---

# Transfer quota within an Azure Quota Group

You can self-distribute quota from a group to subscriptions in a group using the Azure Quota Group ARM object. Once quota is approved at the group level, you can distribute approved quota across the subscription to support successful deployments. Additionally, you can transfer quota from a source subscription to your group, then move to a target subscription also in the group.  

## Transfer quota
Transfer unused quota from your subscription to a Quota Group or from a Quota Group to a subscription.

### [REST API](#tab/rest-1)
To allocate or transfer quota from rhe group to the target subscription, update *subID* to target subscription, then set the *limit* property to the new desired subscription limit. If your current subscription quota is 10 and you want to transfer 10 cores from the group to the target subscription, set the new limit to 20. This applies to a specific region and Virtual Machine (VM) family.  

To de-allocate or transfer quota from the source subscription to the group, update *subID* to source subscription, then set *limit* property to the new desired subscription limit. If your current subscription quota is 20 cores and you want to transfer 10 cores from the source subscription to the group, set the new limit to 10. This will initiate the transfer for the specific region and VM Family.  

View the quota allocation snapshot for your subscription in the Quota Group or view the group limit to validate the transfer and stamping of cores at the group level. To view your existing subscription usage for a given region, use the [Compute Usages API](/rest/api/compute/usage/list?view=rest-compute-2023-07-01&tabs=HTTP&tryIt=true&source=docs#code-try-0).  

```http
PATCH https://management.azure.com/"providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01"
```

```json
{
  "properties": {
    "value": [{
        "properties": {
          "limit": 50,
          "resourceName": "standardddv4family"
        }
      }]
  }
}
```

In this example using `az rest`, we transfer 10 cores of *standarddv4family* in *centralus* from the subscription to the group by setting our limit to 50.

```json
az rest –method patch –url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01" –body ‘{
  "properties": {
    "value": [
      {
        "properties": {
          "limit": 50,
          "resourceName": "standardddv4family"
        }
      }
    ]
  }
}’ –debug
```

### [Azure portal](#tab/portal-1)
Transfer unused quota between subscriptions through the Quota Group object. The following steps are how to do a quota transfer from the source subscription to the group and from the group to the target subscription.

#### Transfer from source subscription to group 

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group. 
4. Select **Quota group** from the list of Quota Groups.
5. In the Quota Group resources view, there's a list of Quota Group resources by region and by Group quota (limit).  
6. Use the filters to select **Region** and or **VM Family**.  
7. Select the **Quota group resource** link to view the Quota Group resources details view for selected resource. View your **Group Quota** also known as group limit, **Total current usage / limit** also know as the sum of subscriptions usage over sum of subscription quota. Lastly, view the list of subscriptions in the selected group.  
8. Select from the list of subscriptions and select the **Manage subscription quota** button.  
9. Select the **Return quota to family limit** option. The **Increase subscription quota** option will be greyed out if Group Quota for the selected resource is `0`.
10. In the **Distribute** blade, view your **Group quota** also known as group limit and the **Manage subscription quota** drop-down. Ensure **Return quota to group quota** is selected if you want to transfer unused quota from source subscription to group.  
11. View the list of subscriptions by **Current usage / limit** and **Return to group quota column**. Input the amount of quota you'd like to transfer from source subscription to group. Output message **Your new quota limit will be** will indicate the new subscription quota if you complete the transfer. You can't insert a value above the current subscription quota or above the value of subscription quota usage.   
12. Select the **Next** button to view the **Review + Distribute** page. View your **New group quota** value if you submit a quota transfer. View the list of subscriptions selected by **New usage / limit** with the value inputted in the previous step. The **Returned to group quota** column indicates the quota being moved from subscription to group.  
13. Select the **Submit** button to trigger the quota transfer and a notification stating **We are reviewing your request to adjust quota** will pop up. Quota transfer may take several minutes to complete.  
14. Once completed, notification **Your quota has been adjusted** with the subscription name and the new subscription limit value will pop up.  

#### Transfer from group to target subscription 

1. Sign in to the [Azure portal](https://portal.azure.com).
2. Type "quotas" into the search box, then select **Quotas**.
3. Under the settings in left hand side, select **Quota groups**.
4. To view existing Quota group, select **Management Group** filter and select the management group used to create your Quota Group. 
5. Select **Quota group** from the list of Quota Groups.
6. In the Quota Group resources view, there's a list of Quota Group resources by region and by Group quota (limit).  
7. Use the filters to select **Region** and or **VM Family**.  
8. Select the **Quota group resource** link to view the Quota Group resources details view for selected resource. View your **Group Quota** also known as group limit, **Total current usage / limit** also know as the sum of subscriptions usage over sum of subscription quota. Lastly, view the list of subscriptions in the selected group.  
9. Select from the list of subscriptions and select the **Manage subscription quota** button.  
10. Select the **Increase subscription quota** option. This option will be greyed out if Group Quota for the selected resource is `0`.
11. In the **Distribute** blade, view your **Group quota** also known as group limit and the **Manage subscription quota** drop-down. Ensure **Increase subscription quota** is selected if you want to transfer unused quota from the group to the taget subscription.
12. View the list of subscriptions by **Current usage / limit** and **Distribute to subscription quota**. Input the amount of quota you'd like to transfer from the group to the target subsciption. Output message **Your new quota limit will be** will indicate the new subscription quota if you complete the transfer. You can't insert a value above the group quota limit or 0.   
13. Select the **Next** button to view the **Review + Distribute** page. View your **New group quota** value if you submit a quota transfer. View the list of subscriptions selected by **New usage / limit** with the value inputted in the previous step. The **Distribute to subscription quota** column indicates the quota being moved from group to subscription.  
14. Select the **Submit** button to trigger the quota transfer and a notification stating **We are reviewing your request to adjust quota** will pop up. Quota transfer may take several minutes to complete.  
15. Once completed, notification **Your quota has been adjusted** with the subscription name and the new subscription limit value will pop up.  

---

## Quota allocation snapshot
View the quota allocation snapshot for the subscription in you Quota Group.

- **Limit** is current subscription limit.
- **Shareable quota** is how many cores have been deallocated or transferred from the subsciption to the group. For example, a value of `-5` is 5 cores were given from the subsciption to the group.   

### [REST API](#tab/rest-2)

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01
```

In the following example using `az rest`, our new subscription limit is 50 cores for *standarddv4family* in *centralus* and our shareable quota is `-10`, because we gave 10 cores to our Quota Group. 

```json
az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/eastus?api-version=2025-03-01&\$filter=resourceName eq 'standardddv4family'" --debug
Response content

{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/eastus",
  "name": "eastus",
  "provisioningState": "Succeeded",
  "type": "Microsoft.Quota/groupQuotas/quotaAllocations",
  "value": [
    {
      "properties": {
        "limit": 50,
        "name": {
          "localizedValue": "standardddv4family",
          "value": "standardddv4family"
        },
        "resourceName": "standardddv4family",
        "shareableQuota": -10
      }
    }
  ]
}

```

--- 
