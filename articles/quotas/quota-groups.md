# Azure Quota Group (Preview)

## Overview
Azure Quota Groups is a new offering from Azure that allows customers to share quota among a group of subscriptions, reducing the number of quota transactions. This feature elevates the quota construct from a subscription level to a Quota Group (ARM object), enabling customers to self-manage their procured quota within a group without needing approvals.

## Key Benefits
• 	Quota Sharing Across Subscriptions: Share procured quotas within a group of subscriptions  
• 	Self-Service Management: Distribute or reallocate unused quota without Microsoft intervention  
• 	Fewer Support Requests: Avoid filing support tickets when reallocating quota or managing new subscriptions  
• 	Group Quota Requests: Request quota at the group level and allocate it across subscriptions as needed  


### Supported Scenarios  
• 	Deallocation – transfer unused quota from subscription(s) to Group Quota  
• 	Allocation – transfer quota from group to target subscription(s)  
• 	Submit Quota Group increase request for given region and VM family, once request is approved transfer quota from group to target subscription(s)  
• 	Quota Group increase requests are subject to the same checks as subscription quota increase requests. If capacity is low, then requests will be rejected and require customers to contact their customer owner. 

## Prerequisites
`Important`
Before you can use Quota Group feature, you must:  
•	Register the Microsoft.Quota resource provider on all relevant subscriptions using PowerShell  
•	Management Group is needed to create Quota Group, group will inherit quota write and or read permissions from the Management Group; subscriptions belonging to other MGs can 	be added to Quota Group  
•	Assign the GroupQuota Request Operator role on the Management Group where the Quota Group will be created  
•	Assign the Quota Request Operator role on all participating subscriptions to the relevant users or applications managing quota operations  

## Preview Limitations
•	Available only for Enterprise Agreement and Internal subscriptions  
•	Supports IaaS compute resources only  
•	Available in public cloud regions only  
•	Management group deletion will result in customer’s loss of access to Quota Group limit, please ensure to zero out group limit by allocating cores to subscription(s), deleting subscriptions, then Quota Group object before deletion of Management Group. In the even that MG is deleted, customer can access Quota Group limit by recreating MG with same ID  

## Quota Group as an ARM object overview
Quota Group is a global ARM object created under a Management Group to logically group subscriptions for quota management. While it is tied to the Management Group for permissions, it does not auto-sync subscription membership. This means you have full flexibility to include subscriptions from different Management Groups.

•	Created at the Management Group scope (not the Root Management Group)  
•	Inherits permissions from its parent Management Group  
•	Designed as an orthogonal grouping mechanism—independent of subscription placement in the Management Group hierarchy  
•	Subscription lists are not auto-synced from Management Groups, giving you flexibility to organize quotas separately from policy or role management  

The following diagram illustrates this concept.  
The existing Management Group hierarchy was set up with sub 1 and sub 2 being part of Management Group A, and sub 3 being a part of Management Group B, customer chose to create all quota groups under a single management group (A). 
 
 :::image type="content" source="./media/quota-groups/sample-management-group-quota-group-hierarchy.png" alt-text="Diagram of Management Group hierarchy with sample Quota Groups created under Management Group.":::
 
Figure 1: Sample MG and Quota Group Hierarchy  

### Recommended group setup
A single Quota Group object can manage quotas across multiple regions and VM families. Design your quota group structure with access control in mind—since access is inherited from the Management Group, consider creating a tiered Management Group structure to ensure proper role assignments.  
Example Hierarchy:  
•	Management Group A owns Quota Groups 1 & 2  
•	Management Group B owns Quota Group 3  
•	Each quota group may used to manage different applications, departments, and or regions
•	When performing operations such as quota transfers or increase requests, actions are scoped to specific regions and VM families  

 :::image type="content" source="./media/quota-groups/sample-recommended-quota-group-setup.png" alt-text="Diagram of Management Group hierarchy with multiple Quota Groups created under Management Group.":::
Figure 2: Sample Quota Group hierarchy

## Permissions required to create Quota Group and add subscription(s)
Quota write permissions are required at the Management Group level to create/delete Quota Group  
Quota write permissions are required at the subscription(s) level to add/remove subscriptions to Quota Group  
### Assign Management group Level Permissions to user and or app serviceID  
•	Please assign user and or app service the "GroupQuota Request Operator" role for the Management Group that will be used to create Quota Group  
•	[How to assign role via CLI: Assign Azure roles using Azure CLI - Azure RBAC | Microsoft Learn](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-cli)  
•	[How to assign via Portal: Assign Azure roles using the Azure portal - Azure RBAC | Microsoft Learn](https://learn.microsoft.com/en-us/azure/role-based-access-control/role-assignments-portal)

```
az role assignment create --assignee "{assignee}" 
--role "{roleNameOrId}" 
--scope "/providers/Microsoft.Management/managementGroups/{managementGroupName}"
```
	
```
"Name": "GroupQuota Request Operator"
"Id": "e2217c0e04bb4724958091cf9871bc01",
"IsServiceRole": false,
"Permissions": [
  {
    "Actions": [
      // Permissions required for a customer role
      "Microsoft.Authorization/*/read",
      "Microsoft.Insights/alertRules/*",
      "Microsoft.Resources/deployments/*",
	
//Quota operations
"Microsoft.Quota/quotaLimits/read",
       "Microsoft.Quota/quotaLimits/write",
       "Microsoft.Quota/quotaLimitsRequests/read",
       "Microsoft.Quota/register/action",
//GroupQuota Operations
      "Microsoft.Quota/GroupQuotas/*/read",
      "Microsoft.Quota/groupQuotas/*/write",
     ]    ]
A GroupQuota Reader role is defined to give readonly access.
"Name": "Quota Group Request Operator",
"IsServiceRole": false,
"Permissions": [
  {
    "Actions": [
      // Permissions required for a customer role
      "Microsoft.Authorization/*/read",
      "Microsoft.Insights/alertRules/*",
      "Microsoft.Resources/deployments/*",

//Quota operations
"Microsoft.Quota/quotaLimits/read",
       "Microsoft.Quota/quotaLimitsRequests/read",
       "Microsoft.Quota/register/action",
//GroupQuota Operations
      "Microsoft.Quota/GroupQuotas/*/read",
      ]

```
 

### Assign Subscription Level Permissions to user and or app serviceID  
•	Please assign user and or app service the "Quota Request Operator" for the subscription(s) that will be added to the group  
```
az role assignment create --assignee "{assignee}" 
--role "{roleNameOrId}" 
--scope "/subscriptions/{subscriptionId}"
```


```
{
  "assignableScopes": [
    "/"
  ],
  "description": "Read and create quota requests, get quota request status, and create support tickets.",
  "id": "/providers/Microsoft.Authorization/roleDefinitions/0e5f05e5-9ab9-446b-b98d-1e2157c94125",
  "name": "0e5f05e5-9ab9-446b-b98d-1e2157c94125",
  "permissions": [
    {
      "actions": [
        "Microsoft.Capacity/resourceProviders/locations/serviceLimits/read",
        "Microsoft.Capacity/resourceProviders/locations/serviceLimits/write",
        "Microsoft.Capacity/resourceProviders/locations/serviceLimitsRequests/read",
        "Microsoft.Capacity/register/action",
        "Microsoft.Quota/usages/read",
        "Microsoft.Quota/quotas/read",
        "Microsoft.Quota/quotas/write",
        "Microsoft.Quota/quotaRequests/read",
        "Microsoft.Quota/register/action",
        "Microsoft.Authorization/*/read",
        "Microsoft.Insights/alertRules/*",
        "Microsoft.Resources/deployments/*",
        "Microsoft.Resources/subscriptions/resourceGroups/read",
        "Microsoft.Support/*"
      ],
      "notActions": [],
      "dataActions": [],
      "notDataActions": []
    }
  ],
  "roleName": "Quota Request Operator",
  "roleType": "BuiltInRole",
  "type": "Microsoft.Authorization/roleDefinitions"
}
```  
# Azure Quota Group APIs Public Preview
## [Using Quota Group APIs](https://github.com/Azure/azure-rest-api-specs/blob/main/specification/quota/resource-manager/Microsoft.Quota/stable/2025-03-01/groupquota.json)  
With Quota Group APIs you can:  
1.	Create/delete Quota Group  
2.	Add/remove subscription(s) from Quota Group  
3.	Transfer/deallocate unused quota from subscription(s) to Quota Group  
4.	Transfer/allocation unused quota from Quota Group to subscription(s)  
5.	Create Quota Group increase request  
6.	Get the properties of Quota Group object and list of subscriptions  
7.	Get status request of allocation request  
8.	Get status request of Quota Group increase  


## SDK Sample links
Go: https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/quota/armquota@v1.1.0 
 
Python: https://pypi.org/project/azure-mgmt-quota/2.0.0/ 
 
Java: https://central.sonatype.com/artifact/com.azure.resourcemanager/azure-resourcemanager-quota/1.1.0 
 
.NET: https://www.nuget.org/packages/Azure.ResourceManager.Quota/1.1.0#readme-body-tab 
 
JS: https://www.npmjs.com/package/@azure/arm-quota/v/1.1.0  

## Examples
### Create a Quota Group
```
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01 
Request Body:
{
  "properties": {
    "displayName": "allocationGroupTest"
  }
}
```
Sample Response: 
```
user [ ~ ]$ az rest --method put --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}?api-version=2025-03-01 --body '{
  "properties": {
    "displayName": "allocationGroupTest"
  }
}'
{
  "id": "/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}",
  "name": "{groupquota}",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas"
}
```

### Add / remove subscriptions from Group
```
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01
```
```
202 – status code
Response header
'Location': 'https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptionRequestsOperationsStatus/9ff167a4-7ab8-4036-9eca-b500206d0d04?api-version=2025-03-01
Retry-After: 30 
Response content
{"properties":{"provisioningState":"ACCEPTED"},"id":"/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a","type":"Microsoft.Quota/groupQuotas/subscriptions","name":"075216c4-f88b-4a82-b9f8-cdebf9cc097a"}
```
```
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01

```

### Get List of subscriptions in group

```
GET
https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupQuota}/subscriptions?api-version=2025-03-01

```
Azrest sample

```
az rest –method get –debug –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupQuota}/subscriptions?api-version=2025-03-01” –debug

user [ ~ ]$ az rest –method get –debug –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions?api-version=2025-03-01” –debug

{
  “values”: [
    {
      “Properties”: {
        “provisioningState”: “SUCCEEDED”,
        “subscriptionId”: “075216c4-f88b-4a82-b9f8-cdebf9cc097a”
      },
      “id”: “/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a”,
      “name”: “075216c4-f88b-4a82-b9f8-cdebf9cc097a”,
      “type”: “Microsoft.Quota/groupQuotas/subscriptions”
    },
    {
      “Properties”: {
        “provisioningState”: “SUCCEEDED”,
        “subscriptionId”: “aa3b53ad-601b-473e-b727-f933435c8263”
      },
      “id”: “/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/aa3b53ad-601b-473e-b727-f933435c8263”,
      “name”: “aa3b53ad-601b-473e-b727-f933435c8263”,
      “type”: “Microsoft.Quota/groupQuotas/subscriptions”
    }
  ]
}

```

### Transfer unused quota from subscription to group OR group to subscription 
To deallocate/transfer quota from subscription to group:  
o	Set the limit property as absolute value to the new desired subscription limit. If you want to transfer 10 cores of standarddv4family to your group and your current subscription limit is 120, set the new limit to 110.  

To allocate/transfer quota from group to subscription:  
o	Set the limit property to the new desired subscription limit. If your current subscription quota is 110 and you want to transfer 10 cores from group to target subscription, set the new limit to 120.  


### PATCH Subscription Quota Allocation
```
PATCH https://management.azure.com/”providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/ {groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01”
{
  “properties”: {
    “value”: [
      {
        “properties”: {
          “limit”: 110,
          “resourceName”: “standardddv4family”
        }
      }
    ]
  }
}
```

azrest example
```
az rest –method patch –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01” –body ‘{
  “properties”: {
    “value”: [
      {
        “properties”: {
          “limit”: 110,
          “resourceName”: “standardddv4family”
        }
      }
    ]
  }
}’ –debug
```
### GET Subscription Allocation Quota request status 
o	Succeeded   
o	In progress  
o	Escalated  
•	If allocation request request status = succeeded then GET subscription quotaAllocations to view current subscription limit for region x SKU 
o	Limit = current subscription limit  
o	Shareable quota = how many cores have been deallocated/transferred from sub to group
	‘-5’ = 5 cores were given from sub to group  
```
GET
/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/quotaAllocationRequests/{allocationId}?api-version=2025-03-01
```

```
Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/{groupquota}/quotaAllocationOperationsStatus/3b80d125-ada8-41c8-b6c4-12fae6a7f55b?api-version=2025-03-01
Retry-After: 30 
Response Content
```
The sample response would be: 
```
{
"properties": {
"requestedResource": {
"properties": {
"limit": 0,
"name": {
"value": "string",
"localizedValue": "string"
},
"region": "string"
}
},
"requestSubmitTime": "2025-03-19T14:53:18.065Z",
"provisioningState": "Accepted",
"faultCode": "string"
}
}
```

### Get Subscription Quota Allocation 

•	view current subscription limit for region x SKU  
•	Limit = current subscription limit  
•	Shareable quota = how many cores have been deallocated/transferred from sub to group  ‘-5’ = 5 cores were given from sub to group  
Status code: 200
Response Header: 

```
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01
```



azrest example
```
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
        "limit": 5,
        "name": {
          "localizedValue": "standardddv4family",
          "value": "standardddv4family"
        },
        "resourceName": "standardddv4family",
        "shareableQuota": -5
      }
    }
  ]
}
```

### Submit Quota Group increase request
```
PATCH https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01
{
  "properties": {
    "value": [
      {
        "properties": {
          "resourceName": "standardddv4family",
          "limit": 100,
          "comment": "comments"
        }
      }
    ]
  }
}
```

### Get Quota Group increase request status

```
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/groupQuotaRequests/{requestId}?api-version=2025-03-01

Sample response
{
"properties": {
"requestedResource": {
"properties": {
"limit": 0,
"name": {
"value": "string",
"localizedValue": "string"
},
"region": "string",
"comments": "string"
}
},
"requestSubmitTime": "2025-03-19T15:03:30.611Z",
"provisioningState": "Accepted",
"faultCode": "string"
}
}
```

### Get Quota Group limit 
•	available limit = how many  cores do I have at group level to distribute  
•	limit = how many cores have been explicitly requested and approved/stamped on your group via quota increase requests  
•	quota allocated = how many cores the sub has been allocated from group, ‘-‘ value indicates cores have been allocated from sub to group  

```
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose
```

azrest example  
```
az rest --method get --url https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus?api-version=2025-03-01&$filter=resourceName eq standardDSv3Family

sample reponse
user [ ~ ]$ az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus?api-version=2025-03-01&$filter=resourceName eq 'standardddv4family'"
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
                "quotaAllocated": -5,
                "subscriptionId": "075216c4-f88b-4a82-b9f8-cdebf9cc097a"
              }
            ]
          },
          "availableLimit": 100,
          "limit": 95,
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


### Quota Group Increase Escalations
• 	Customers can submit Quota Group increase requests for a region x VM family combination, and if approved, quota will be stamped on the specified Quota GroupID.  
• 	Quota Group increase requests undergo the same checks as subscription level requests.  
• 	Whether you submit a request via portal or API, your request will be reviewed, and you'll be notified if the request can be fulfilled. This usually happens within a few minutes. If your request isn't fulfilled, you'll see a link 	where you can open a support request so that a support engineer can assist you with the increase.  
• 	Support tickets for Quota Groups will be created based on a preselected subscriptionID within the group, the customer has the ability to edit the subID when updating request details. 

