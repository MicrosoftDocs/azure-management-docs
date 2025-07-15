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

Limitations
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


1. Procedure step
1. Procedure step
1. Procedure step

<!-- Required: Steps to complete the task - H2

In one or more H2 sections, organize procedures. A section
contains a major grouping of steps that help the user complete
a task.

Begin each section with a brief explanation for context, and
provide an ordered list of steps to complete the procedure.

If it applies, provide sections that describe alternative tasks or
procedures.

-->

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
