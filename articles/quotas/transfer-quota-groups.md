---
title: Transfer quota within a Quota Group
description: Learn how to transfer quota within a quota group.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/07/2025
---
# Transfer quota within Quota Group
Using the Quota Group APIs, customers are able to self-distribute quota from group to subscriptions in a group. 
### [REST API](#tab/rest-4)
- Transfer unused quota from your subscription to a Quota Group or from a Quota Group to a subscription.
- Once your quota group is created and subscriptions are added, you can transfer quota between subscriptions by transferring quota from source subscription to group. First, deallocate quota from the source subscription and return it to the group. Then, allocate that quota from the group to the target subscription.
- To allocate or transfer quota from group to target subscription, update subID to target subscription, then set the limit property to the new desired subscription limit. If your current subscription quota is 10 and you want to transfer 10 cores from group to target subscription, set the new limit to 20. This applies to a specific region and VM family.  
- You can view quota allocation snapshot for subscription in Quota Group or view group limit to validate transfer and stamping of cores at group level.  
- To view your existing subscription usage for a given region, please use the [Compute Usages API](/rest/api/compute/usage/list?view=rest-compute-2023-07-01&tabs=HTTP&tryIt=true&source=docs#code-try-0).  

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

Example using `az rest`: 
 I transfer 10 cores  of *standarddv4family* in centralus from subscription to group by setting limit to 50


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
### View quota allocation snapshot for subscription in Quota Group
- Limit = current subscription limit  
- Shareable quota = how many cores have been deallocated/transferred from sub to group  ‘-5’ = 5 cores were given from sub to group  

```http
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01
```

Example using `az rest`:
My new subscription limit is 50 cores for *standarddv4family* in centralus and my shareable quota is -10 because I gave 10 cores to my Quota group. 
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

<!-- Portal steps on how to do quota transfer -->
### [Azure portal](#tab/portal-4)
Transfer unused quota between subscriptions via the Quota Group object. Below steps are how to do quota transfer from source subscription to group and from group to target subscription.

Transfer from source subscription to group. 
1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.  
2. Under settings in left hand side, select **Quota groups**.  
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group  
4. Select **Quota group** from list of Quota Group(s)  
5. In the Quota Group resources view there will be the list of Quota group resources by region by Group quota (limit)  
6. Use the filters to select **Region** and or **VM Family**, you can also search for region and or VM family in the search bar  
7. Select the **Quota group resource** link to view the Quota Group resources details view for selected resource, view your **Group Quota** also known as group limit, **Total current usage / limit** also know as the sum of subscriptions  usage over sum of subscription quota. Lastly view list of subscriptions  in selected group -> Quota Group resource  
8. Select from the list of subscriptions  and select **Manage subscription quota** button  
9. Select **Return quota to family limit** option, the **Increase subscription quota** option will be greyed out if Group quota for selected   resource = 0  
10. In the **Distribute** blade you can view your **Group quota** also known as group limit and the **Manage subscription quota** drop down, ensure **Return quota to group quota** is selected if you want to transfer unused quota from source subscription to group.  
11. View the list of subscriptions  by **Current usage / limit** and **Return to group quota column**, input value of quota you'd like to transfer from source subscription to group. Output message **Your new quota limit will be** will indicate the new subscription quota if you complete transfer. You cannot insert value above the current subscription quota or above the value of (subscription quota - usage)  
12. Select **Next** button to view the **Review + Distribute** page. You can view your **New group quota** value if you submit quota transfer, you can also view list of subscription selected by **New usage / limit** with the value inputted in previous step. The **Returned to group quota** column indicates the quota being moved from subscription to group.  
13. Select **Submit** button to trigger quota transfer, notification **We are reviewing your request to adjust quota** on right hand side will surface. Quota transfer may take up to ~3 minutes to complete.  
14. Once completed notification **Your quota has been adjusted** with subscription name and new subscription limit value will surface on right hand side.  
15. Select the Quota group resource / VM family in breadcrumb **Home -> Quota |Quota group -> QuotaGroupName -> Quota group resource / VM family** to view updated Group quota  

Transfer from group to target subscription 
1. To view the Quotas page, sign in to the Azure portal and enter "quotas" into the search box, then select **Quotas**.  
2. Under settings in left hand side, select **Quota groups**.  
3. To view existing Quota group, select **Management Group** filter and select management group used to create Quota Group  
4. Select **Quota group** from list of Quota Group(s)  
5. In the Quota Group resources view there will be the list of Quota group resources by region by Group quota (limit)  
6. Use the filters to select **Region** and or **VM Family**, you can also search for region and or VM family in the search bar  
7. Select the **Quota group resource** link to view the Quota Group resources details view for selected resource, view your **Group Quota** also known as group limit, **Total current usage / limit** also know as the sum of subscriptions  usage over sum of subscription quota. Lastly view list of subscriptions  in selected group -> Quota Group resource
8. Select from the list of subscriptions  and select **Manage subscription quota** button
9. Select **Increase subscription quota** option, this will be geyed out if Group quota for selected resource = 0
10. In the **Distribute** blade you can view your **Group quota** also known as group limit and the **Manage subscription quota** drop down, ensure **Increase subscription quota** is selected if you want to transfer unused quota from group to target subscription
11. View the list of subscriptions  by **Current usage / limit** and **Distribute to subscription quota**, input value of quota you'd like to transfer from group to target subscription. Output message **Your new quota limit will be** will indicate the new subscription quota if you complete transfer. You cannot insert value above the group quota limit or 0.
12. Select **Next** button to view the **Review + Distribute** page. You can view your **New group quota** value if you submit quota transfer, you can also view list of subscription selected by **New usage / limit** with the value inputted in previous step. The **Distribute to subscription quota** column indicates the quota being moved from group to subscription. 
13. Select **Submit** button to trigger quota transfer, notification **We are reviewing your request to adjust quota** on right hand side will surface. Quota transfer may take up to ~3 minutes to complete.  
14. Once completed notification **Your quota has been adjusted** with subscription name and new subscription limit value will surface on right hand side.
15. Select the Quota group resource / VM family in breadcrumb **Home -> Quota |Quota group -> QuotaGroupName -> Quota group resource / VM family** to view updated Group quota  
--- 
