# Azure Quota Group Public Preview

## What is Azure Quota Groups? 
Quota Groups is a new Azure offering that will empower customers to share quota amongst a group of subscriptions and reduce the number of quota transactions. Quota Group levels the quota construct from a subscription to a Quota Group (ARM object), enabling customers to self-manage their procured quota in a group without any approvals.

## Quota Group Allocation/Self-distribution option
The Quota Group Allocation/Self-Distribution feature of the Quota Group product allows customers to self-distribute or transfer unused quota between subscriptions within the group without needing to file a support ticket.  
This feature eliminates the necessity for customers to involve Microsoft and procure additional quota whenever a subscription is nearing its limit.
With this option, customers can add new subscriptions to a group and transfer unused quota from existing subscriptions within the group without filing a support ticket. 
Additionally, customers can request quota at the group level instead of per subscription. Once the quota is approved, customers can transfer it to the subscriptions within the group as needed.  

Supported Scenarios  
1.	Deallocation – transfer unused quota from subscription(s) to Group Quota  
2.	Allocation – transfer quota from group to target subscription(s)  
3.	Submit Quota Group increase request for given region and VM family, once request is approved transfer quota from group to target subscription(s)  
a.	Quota Group increase requests are subject to the same checks as subscription quota increase requests. If capacity is low, then requests will be rejected and require customers to contact their customer owner. 

## Prerequisites
`Important`
Before you can use Quota Group feature, you must:  
•	Register the Microsoft.Quota resource provider for all your subscriptions using PowerShell  
•	Management Group is needed to create Quota Group, group will inherit quota write and or read permissions from the Management Group; subscriptions belonging to other MGs can be added to Quota Group  
•	Assign the GroupQuota Request Operator role for the Management Group that will be used to create Quota Group  
•	Assign the Quota Request Operator role on the subscription(s) to all user accounts and applications that will perform quota operations  

## Quota Group Limitations`
•Quota Group offering is only supported for Enterprise Agreement and Internal subscriptions  
•	Quota Group is only supported for IaaS compute resources  
•	Public cloud support only  
•	Management group deletion will result in customer’s loss of access to Quota Group limit, please ensure to zero out group limit by allocating cores to subscription(s), deleting subscriptions, then Quota Group object before deletion of Management Group. In the even that MG is deleted, customer can access Quota Group limit by recreating MG with same ID  

## Quota Group ARM object overview
Quota Group is a new ARM Global object created by customers to group their subscriptions for the purpose of quota management. 
Quota Group objects are created at the Management Group scope and inherit their permissions from their parent Management Group. 
To create a Quota Group object, customers must use a Management Group at least one level below Root Management Group. 
Quota Group cannot be created using Root Management Group. It’s important to note, while the Quota Groups are created as part of the Management Group hierarchy, they are an orthogonal grouping mechanism, and they don’t synchronize the list of subscriptions with any of the management groups. 
This allows for more flexibility in grouping subscriptions, keeping quota management separate from how the policy and permissions are managed by Management Groups.
The following diagram illustrates this concept. 
Please note that the existing Management Group hierarchy was set up with sub 1 and sub 2 being part of Management Group A and sub 3 being a part of Management Group B, customer chose to create all quota groups under a single management group (A). 

 
Figure 1: Sample MG and Quota Group Hierarchy
Recommended group setup
A single Quota Group object can be used to manage quotas for multiple regions and or VM families since it is a global object, the group setup may depend on whether subscription(s) are region specific or span multiple regions. Quota groups inherit permissions from the Management Group, it is therefore important to think through how to design your Quota Group structure in context of who will have access to a specific Quota Group. We recommend creating at least one level of Management Groups in your tenant to manage access to individual quota groups, like in the diagram below. In this example, the Management Group A owner (see individual operations for specific permissions requirements) will have access to Quota Group 1 and Quota group 2. The owner of Management Group B will have access to Quota Group 3. When doing certain quota group transactions, such as quota transfers or submitting Quota Group increase request, operation is specific to a given region and VM family.
 
Figure 2: Sample Quota Group hierarchy

## Permissions required to create Quota Group and add subscription(s)
### Assign Management group Level Permissions to user and or app serviceID, assign the Quota Group request Operator role for the Management Group resource  
•	Please assign user and or app service the "GroupQuota Request Operator" role for the Management Group that will be used to create Quota Group  
•	How to assign role via CLI: Assign Azure roles using Azure CLI - Azure RBAC | Microsoft Learn  

```
az role assignment create --assignee "{assignee}" 
--role "{roleNameOrId}" 
--scope "/providers/Microsoft.Management/managementGroups/{managementGroupName}"
```



•	How to assign via Portal: Assign Azure roles using the Azure portal - Azure RBAC | Microsoft Learn
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
```


```
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
### Assign Subscription Level Permissions to user and or app serviceID; assign Quota Request Operator role at the subscription level  
•	Please assign user and or app service the "Quota Request Operator" for the subscription(s) that will be added to the group  
•	How to assign role via CLI: Assign Azure roles using Azure CLI - Azure RBAC | Microsoft Learn  

```
az role assignment create --assignee "{assignee}" 
--role "{roleNameOrId}" 
--scope "/subscriptions/{subscriptionId}"
```

•	How to assign via Portal: Assign Azure roles using the Azure portal - Azure RBAC | Microsoft Learn

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
Advanced users can create custom roles if they want to give granular permissions to specific applications or users. 
More details at - Azure custom roles - Azure RBAC | Microsoft Learn  

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
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/testMgIdRoot/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota?api-version=2025-03-01 
Request Body:
{
  "properties": {
    "displayName": "allocationGroupTest"
  }
}
```
Sample Response: 
```
yaya [ ~ ]$ az rest --method put --url https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota?api-version=2025-03-01 --body '{
  "properties": {
    "displayName": "allocationGroupTest"
  }
}'
{
  "id": "/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota",
  "name": "sdk-test-group-quota",
  "properties": {
    "provisioningState": "ACCEPTED"
  },
  "type": "Microsoft.Quota/groupQuotas"
}
```

### Add / remove subscriptions from Group
```
PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementgroup}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01```

202 – status code
Response header
'Location': 'https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/subscriptionRequestsOperationsStatus/9ff167a4-7ab8-4036-9eca-b500206d0d04?api-version=2025-03-01
Retry-After: 30 
Response content
{"properties":{"provisioningState":"ACCEPTED"},"id":"/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a","type":"Microsoft.Quota/groupQuotas/subscriptions","name":"075216c4-f88b-4a82-b9f8-cdebf9cc097a"}
DELETE https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementgroup}/providers/Microsoft.Quota/groupQuotas/{groupquota}/subscriptions/{subscriptionId}?api-version=2025-03-01

Status 204
Response header


```
Get List of subscriptions in group

```
GET
https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroup}/providers/Microsoft.Quota/groupQuotas/{groupQuota}/subscriptions?api-version=2025-03-01

```
Azrest sample

```
az rest –method get –debug –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroup}/providers/Microsoft.Quota/groupQuotas/{groupQuota}/subscriptions?api-version=2025-03-01” –debug

yaya [ ~ ]$ az rest –method get –debug –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/subscriptions?api-version=2025-03-01” –debug

{
  “values”: [
    {
      “Properties”: {
        “provisioningState”: “SUCCEEDED”,
        “subscriptionId”: “075216c4-f88b-4a82-b9f8-cdebf9cc097a”
      },
      “id”: “/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a”,
      “name”: “075216c4-f88b-4a82-b9f8-cdebf9cc097a”,
      “type”: “Microsoft.Quota/groupQuotas/subscriptions”
    },
    {
      “Properties”: {
        “provisioningState”: “SUCCEEDED”,
        “subscriptionId”: “aa3b53ad-601b-473e-b727-f933435c8263”
      },
      “id”: “/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/subscriptions/aa3b53ad-601b-473e-b727-f933435c8263”,
      “name”: “aa3b53ad-601b-473e-b727-f933435c8263”,
      “type”: “Microsoft.Quota/groupQuotas/subscriptions”
    }
  ]
}

```

COME BACK TO THIS Get list of subs and their quotas?
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementgroup}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01


### Allocate/Deallocate quota from sub to group OR group to sub  
•	PATCH subscription quotaAllocations quota/ transfer quota from sub to group and group to sub, set subscription limit as absolute value  
To Deallocate/transfer quota from subscription to group:  
o	Set the limit property to your new desired subscription limit. If you want to transfer 10 cores of standarddv4family to your group and your current subscription limit is 120, set the new limit to 110.  

To allocate quota from group to subscription:  
o	Set the limit property to your new desired subscription limit. If your current subscription quota is 110 and you want to add 10 more cores from group, set the new limit to 120.  

GET Allocation quota request status 
o	Succeeded 
o	In progress
o	Escalated
•	If allocation request request status = succeeded then GET subscription quotaAllocations to view current subscription limit for region x SKU 
o	Limit = current subscription limit
o	Shareable quota = how many cores have been deallocated/transferred from sub to group
	‘-5’ = 5 cores were given from sub to group 
PATCH Subscription quotaAllocation
PATCH https://management.azure.com/”providers/Microsoft.Management/managementGroups/ {managementgroup}/ subscriptions/ {subscriptionId}/ providers/Microsoft.Quota/groupQuotas/ {groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01”
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

azrest
az rest –method patch –url “https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementgroup}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01” –body ‘{
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

Status code: 202
Response header:
Location: https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/quotaAllocationOperationsStatus/3b80d125-ada8-41c8-b6c4-12fae6a7f55b?api-version=2025-03-01
Retry-After: 30 
Response Content

Get Subscription quotaAllocation  request status

GET
/providers/Microsoft.Management/managementGroups/{managementGroupId}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupQuotaName}/quotaAllocationRequests/{allocationId}?api-version=2025-03-01
The sample response would be: 
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


Get Subscription quotaAllocation 
•	view current subscription limit for region x SKU 
o	Limit = current subscription limit
o	Shareable quota = how many cores have been deallocated/transferred from sub to group
	‘-5’ = 5 cores were given from sub to group
Status code: 200
Response Header: 


GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementgroup}/subscriptions/{subscriptionId}/providers/Microsoft.Quota/groupQuotas/{groupquota}/resourceProviders/Microsoft.Compute/quotaAllocations/{location}?api-version=2025-03-01




azrest
az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/resourceProviders/Microsoft.Compute/quotaAllocations/eastus?api-version=2025-03-01&\$filter=resourceName eq 'standardddv4family'" --debug
Response content

{
  "id": "/providers/Microsoft.Management/managementGroups/YayaMGtest/subscriptions/075216c4-f88b-4a82-b9f8-cdebf9cc097a/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/resourceProviders/Microsoft.Compute/quotaAllocations/eastus",
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


Submit Quota increase request
PATCH https://management.azure.com/providers/Microsoft.Management/managementGroups/ {managementgroup} /providers/Microsoft.Quota/groupQuotas/ {groupquota} /resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01
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

Get group limit request status
GET https://management.azure.com/providers/Microsoft.Management/managementGroups/{managementGroupId}/providers/Microsoft.Quota/groupQuotas/{groupQuotaName}/groupQuotaRequests/{requestId}?api-version=2025-03-01

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


Get Group limit 
•	available limit = how many  cores do I have at group level to distribute
•	limit = how many cores have been explicitly requested and approved/stamped on your group via quota increase requests 
•	quota allocated = how many cores the sub has been allocated from group, ‘-‘ value indicates cores have been allocated from sub to group

GET https://management.azure.com/providers/Microsoft.Management/managementGroups/ {managementgroup} /providers/Microsoft.Quota/groupQuotas/ {groupquota} /resourceProviders/Microsoft.Compute/groupQuotaLimits/{location}?api-version=2025-03-01&$filter=resourceName eq standarddv4family" -verbose


azrest
az rest --method get --url https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus?api-version=2025-03-01&$filter=resourceName eq standardDSv3Family

sample reponse
yaya [ ~ ]$ az rest --method get --url "https://management.azure.com/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus?api-version=2025-03-01&$filter=resourceName eq 'standardddv4family'"
{
  "id": "/providers/Microsoft.Management/managementGroups/YayaMGtest/providers/Microsoft.Quota/groupQuotas/sdk-test-group-quota/resourceProviders/Microsoft.Compute/groupQuotaLimits/eastus",
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


Quota Group Increase Escalations
When doing certain quota group transactions, such as quota transfers or submitting Quota Group increase request, operation is specific to a given region and VM family. 
Customer will submit Quota Group increase request for QuotaGroupID x region x VM family, the quota group simply replaces the subscriptionID. 
Quota Group increase requests undergo the same checks as subscription level requests. 
Whether you submit a request via portal or API, your request will be reviewed, and you'll be notified if the request can be fulfilled. This usually happens within a few minutes. If your request isn't fulfilled, you'll see a link where you can open a support request so that a support engineer can assist you with the increase.



How to request for zonal and or regional access on subscription(s) in group

•	Keep process as is

