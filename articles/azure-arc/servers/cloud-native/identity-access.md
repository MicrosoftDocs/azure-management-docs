---
title: Identity and access management with Azure Arc-enabled servers
description: You can use Microsoft Entra with Azure Arc-enabled servers to manage identity and access control in your hybrid environment.
ms.date: 08/13/2025
ms.topic: concept-article
# Customer intent: "As a cloud administrator managing a hybrid environment, I want to control access to Azure Arc-enabled servers through Microsoft Entra, so I can use Azure's identity system to control access to resources."
---

# Identity and access management with Azure Arc-enabled servers


Managing identity for servers traditionally revolved around Active Directory: servers are domain-joined, admins are given domain accounts that are added to local Administrators via domain groups, and Windows settings are managed using Group Policy. In the cloud management model, [Microsoft Entra](/entra/fundamentals/what-is-entra) becomes the cornerstone of identity and access, while Active Directory (AD) can still be used for app authentication and legacy protocols on on-premises Windows machines.

Cloud-native identity in server management is achieved by using Microsoft Entra for authenticating admins and the servers themselves. On-premises AD domain-joined servers can still be accessed by cloud-native Windows (workstation) devices or users on those devices. Through Microsoft Entra, you gain unified credentials, as [Microsoft Entra ID](/entra/fundamentals/whatis) can manage VMs, Arc-enabled servers,  Office 365, and more. Features like multifactor authentication (MFA) and conditional access improve security. Servers in your hybrid environment can use Azure's identity system to access resources securely. These benefits let you reduce time spent maintaining service accounts or granting local admin rights per machine. It's a transition in thinking, but one that aligns with a fully cloud-managed ecosystem.

Let's look at how a system administrator's world changes with the benefits of Microsoft Entra.

## Microsoft Entra integration

[Microsoft Entra ID](/entra/fundamentals/whatis) is a cloud-based identity service. Unlike [Active Directory Domain Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview), Microsoft Entra ID isn't structured in Organizational Units and doesn't focus on Kerberos authentication. Instead, Microsoft Entra ID manages user identities, apps, and access to Microsoft resources, including Azure, Microsoft 365, and other applications and operating systems that support Microsoft Entra ID.

Servers themselves don’t "join" Microsoft Entra ID the way that they join a domain. Instead, an Arc-enabled server is joined to an Azure tenant governed by Microsoft Entra ID when it's first connected to Azure. With Microsoft Entra ID, users can be assigned roles to a designated scope (or added to a group with those permissions) by using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview). Then, users with the appropriate permissions can use a Remote Desktop connection to access Windows Server machines, or [use SSH to access Linux](../ssh-arc-overview.md).

Your "admin account" is your Microsoft Entra ID identity (or a synced AD account) with appropriate roles in Azure. For example, to manage Arc-enabled servers, a Microsoft Entra ID user might have the Azure built-in role [Virtual Machine Administrator Login](/azure/role-based-access-control/built-in-roles/compute) role, or a custom role assignment you create with appropriate permissions. Rather than having one admin account that allows full access to every server, Microsoft Entra ID lets you scope roles to a specific set of Azure workloads, granting only the permissions required to perform the necessary tasks on Arc-enabled servers and native Azure resources.
