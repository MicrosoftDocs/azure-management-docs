---
title: Share Quota Across Subscriptions with Azure Quota Groups
description: Learn how to share quota across Azure subscriptions with Azure Quota Groups to reduce the number of quota transactions.
author: yaya-5
ms.author: yaalanis
ms.topic: how-to
ms.date: 07/23/2025
---

# Azure Quota Groups

Azure Quota Groups allow you to share quota among a group of subscriptions, reducing the number of quota transactions. This feature elevates the quota construct from a subscription level to a Quota Group Azure Resource Management (ARM) object, enabling customers to self-manage their procured quota within a group without needing approvals.

## Key benefits

- **Quota sharing across subscriptions**: Share procured quotas within a group of subscriptions.  
- **Self-service management**: Distribute or reallocate unused quota without Microsoft intervention.  
- **Fewer support requests**: Avoid filing support tickets when reallocating quota or managing new subscriptions.  
- **Group quota requests**: Request quota at the group level and allocate it across subscriptions as needed.  


## Supported scenarios  
The transfer of unused quota between subscriptions is done via Quota Group object created. At the moment of creating a Quota group object, the group limit is set to 0. Customers must update the group limit themselves, either by transferring quota from a subscription in the group or by submitting a Quota group limit increase request and getting approved. When deploying resources the quota check at runtime is done against the subscription quota.  

- Deallocation: Transfer unused quota from your subscriptions to Group Quota. 
- Allocation: Transfer quota from group to target subscriptions. 
- Submit Quota Group increase request for a given region and Virtual Machine (VM) family. Once your request is approved, transfer quota from group to target subscriptions. 
- Quota Group increase requests are subject to the same checks as subscription quota increase requests. If capacity is low, then the request is rejected. 

## Prerequisites

Before you can use the Quota Group feature, you must:  
- Register the `Microsoft.Quota` and `Microsoft.Compute` resource provider on all relevant subscriptions before adding to a Quota Group. For more information, see [Registering the Microsoft Quota resource provider](/rest/api/quota/#registering-the-microsoft-quota-resource-provider).
- A Management Group (MG) is needed to create a Quota Group. Your group inherits quota write and or read permissions from the Management Group. Subscriptions belonging to another MG can be added to the Quota Group.
- Certain permissions are required to create Quota Groups and to add subscriptions.    

## Limitations

- Available only for Enterprise Agreement or Microsoft Customer Agreement and Internal subscriptions. 
- Supports IaaS compute resources only.  
- Available in public cloud regions only.  
- Management Group deletion results in the loss of access to the Quota Group limit. To clear out the group limit, allocate cores to subscriptions, delete subscriptions, then the Quota Group object before deletion of Management Group. In the even that the MG is deleted, access your Quota Group limit by recreating the MG with the same ID as before.
- A subscription can belong to a single Quota group at a time.
- Quota Groups addresses the quota management pain point, it does not address the regional and or zonal access pain point. To get region and or zonal access on subscriptions, [see region access request process](/troubleshoot/azure/general/region-access-request-process). Quota transfers between subscriptions and deployments will fail unless regional and or zonal access is provided on the subscription.  

## Quota Group is an ARM object

Quota Group is a global ARM object created under a Management Group to logically group subscriptions for quota management. While it is tied to the Management Group for permissions, it doesn't auto-sync subscription membership. This means you have full flexibility to include subscriptions from different Management Groups. Quota Groups are: 

- Quota Groups are created at the Management Group scope.  
- Quota Groups are inherit permissions from its parent Management Group.
- Quota Groups are designed as an orthogonal grouping mechanism. They're independent of subscription placement in the Management Group hierarchy.
- Subscription lists are not auto-synced from Management Groups, giving you flexibility to organize quotas separately from policy or role management.

The following diagram shows and existing MG hierarchy set up with *subscription 1* and *subscription 2* being part of *Management Group A*, and *subscription 3* being part of *Management Group B*. In this example, the customer chose to create all quota groups under the single *Management Group A*. 
 
 :::image type="content" source="./media/quota-groups/sample-management-group-quota-group-hierarchy.png" alt-text="Diagram of Management Group hierarchy with sample Quota Groups created under Management Group."::: 

## Recommended group setup

A single Quota Group object manages quotas across multiple regions and VM families. Design your quota group structure with access control in mind. Access is inherited from the Management Group, so consider creating a tiered Management Group structure to ensure proper role assignments.  

Example hierarchy:  
- Management Group A owns Quota Groups 1 & 2.
- Management Group B owns Quota Group 3.
- Each quota group may be used to manage different applications, departments, and or regions.
- Quota Group operations such as quota transfers or increase requests, actions are scoped to specific regions and VM families.

 :::image type="content" source="./media/quota-groups/sample-recommended-quota-group-setup.png" alt-text="Diagram of Management Group hierarchy with multiple Quota Groups created under Management Group.":::

## Permissions

Certain permissions are required to create Quota Groups and to add subscriptions. For more information, see [Assign Azure roles using Azure CLI](/azure/role-based-access-control/role-assignments-cli) or [Assign Azure roles using the Azure portal](/azure/role-based-access-control/role-assignments-portal).
- Assign the *GroupQuota Request Operator* role on the Management Group where the Quota Group is created.
- Assign the *Quota Request Operator* role on all participating subscriptions to the relevant users or applications managing quota operations.
- Assign the *Reader* role on all participating subscriptions to the relevant users or applications managing quota operations to view quota group resources in portal.
 
## Quota Group APIs

Use [Quota Group APIs](https://github.com/Azure/azure-rest-api-specs/blob/main/specification/quota/resource-manager/Microsoft.Quota/stable/2025-03-01/groupquota.json) to do the following supported Quota Group operations:  
- Create or delete a Quota Group.
- Add or remove subscriptions from a Quota Group.
- Transfer or deallocate unused quota from subscriptions to a Quota Group. 
- Submit a Quota Group limit increase request.
- Submit a support ticket via portal if Quota Group limit request is rejected.
- View Group limit.

## SDK sample links

Use the below links to download the latest supported SDKs for Quota Group operations.

- [Go](https://pkg.go.dev/github.com/Azure/azure-sdk-for-go/sdk/resourcemanager/quota/armquota@v1.1.0)
- [Python](https://pypi.org/project/azure-mgmt-quota/2.0.0/)
- [Java](https://central.sonatype.com/artifact/com.azure.resourcemanager/azure-resourcemanager-quota/1.1.0)
- [.NET](https://www.nuget.org/packages/Azure.ResourceManager.Quota/1.1.0#readme-body-tab)
- [JS](https://www.npmjs.com/package/@azure/arm-quota/v/1.1.0)

