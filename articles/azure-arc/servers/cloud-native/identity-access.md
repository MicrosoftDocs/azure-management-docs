---
title: Identity and access management with Azure Arc-enabled servers
description: You can use Microsoft Entra with Azure Arc-enabled servers to manage identity and access control in your hybrid environment.
ms.date: 08/19/2025
ms.topic: concept-article
# Customer intent: "As a cloud administrator managing a hybrid environment, I want to control access to Azure Arc-enabled servers through Microsoft Entra, so I can use Azure's identity system to control access to resources."
---

# Identity and access management with Azure Arc-enabled servers

Managing identity for servers traditionally revolved around Active Directory: servers are domain-joined, admins are given domain accounts that are added to local Administrators via domain groups, and Windows settings are managed using Group Policy. In the cloud management model, [Microsoft Entra](/entra/fundamentals/what-is-entra) becomes the cornerstone of identity and access, while Active Directory (AD) can still be used for app authentication and legacy protocols on on-premises Windows machines.

Cloud-native identity in server management is achieved by using Microsoft Entra for authenticating admins and the servers themselves. On-premises AD domain-joined servers can still be accessed by cloud-native Windows (workstation) devices or users on those devices. Through Microsoft Entra, you gain unified credentials, as [Microsoft Entra ID](/entra/fundamentals/whatis) can manage virtual machines (VMs), Arc-enabled servers, Office 365, and more. Features like multifactor authentication (MFA) and conditional access improve security. Servers in your hybrid environment can use Azure's identity system to access resources securely. These benefits let you reduce time spent maintaining service accounts or granting local admin rights per machine. It's a transition in thinking, but one that aligns with a fully cloud-managed ecosystem.

Let's look at how a system administrator's world changes with the benefits of Microsoft Entra.

## Microsoft Entra integration

[Microsoft Entra ID](/entra/fundamentals/whatis) is a cloud-based identity service. Unlike [Active Directory Domain Services (AD DS)](/windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview), Microsoft Entra ID isn't structured in Organizational Units and doesn't focus on Kerberos authentication. Instead, Microsoft Entra ID manages user identities, apps, and access to Microsoft resources, including Azure, Microsoft 365, and other applications and operating systems that support Microsoft Entra ID.

Servers themselves don’t "join" Microsoft Entra ID the way that they join a domain. Instead, an Arc-enabled server is joined to an Azure tenant governed by Microsoft Entra ID when it first connects to Azure. With Microsoft Entra ID, users can be assigned roles to a designated scope (or added to a group with those permissions) by using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview). Then, users with the appropriate permissions can use a Remote Desktop connection to access Windows Server machines, or [use SSH to access Linux](../ssh-arc-overview.md).

Your "admin account" is your Microsoft Entra ID identity (or a synced AD account) with appropriate roles in Azure. For example, to manage Arc-enabled servers, a Microsoft Entra ID user might have the Azure built-in role [Virtual Machine Administrator Login](/azure/role-based-access-control/built-in-roles/compute) role, or a custom role assignment you create with appropriate permissions. Rather than having one admin account that allows full access to every server, Microsoft Entra ID lets you scope roles to a specific set of Azure workloads, granting only the permissions required to perform the necessary tasks on Arc-enabled servers and native Azure resources.

## System-assigned managed identity

Arc-enabled servers require a [system-assigned managed identity](../managed-identity-authentication.md). This is a type of enterprise application that represents the identity of a machine resource in Azure. Instead of storing credentials, applications running on the server can use the server's managed identity to authenticate to Azure. The Azure Arc Connected machine agent exposes an endpoint that the app can use to request a token. The app doesn't need to authenticate to the nonroutable web service that provides tokens other than properly formatting the request (including a metadata header), so the expected security boundary is the VM. Your on-premises server can directly access Azure services without requiring hard-coded credentials, because Azure knows the request comes from that server, and authorizes it only based on the role assignments you set.

For a system administrator, one common scenario might be running an Azure CLI command on the server (which has Azure CLI installed) that calls into Azure Storage to retrieve an artifact used by an automation script. Since the server has an identity authorized access to that storage account, the request is completed without requiring a service account or personal access token (PAT).

## Role-based access control (RBAC)

Azure’s model encourages granular RBAC. Because different built-in roles enable different types of access, you can assign roles with very limited permissions. For instance, a user could have a role that only grants the ability to use [Run Command](../run-command.md) on Arc-enabled servers, or allows read-only access to a configuration and nothing more.

A common built-in role used with Azure Arc is the [Azure Connected Machine Onboarding role](/azure/role-based-access-control/built-in-roles/management-and-governance). Users with this role can onboard servers to Azure Arc, but can't do most other management tasks unless additional roles are granted. Similarly, you can give application owners roles that allow them to deploy patches on their servers via Azure Automation without giving them actual OS login access. This level of fine-tuning supports "just-enough" administration principles.

## Just-in-time access

To further control elevated access to your Arc-enabled servers and other Azure resources, you can enable [Microsoft Entra Privileged Identity Management (PIM)](/entra/id-governance/privileged-identity-management/pim-configure). PIM can be used for just-in-time (JIT) access, so you can require that someone explicitly elevate to a specific role in order to perform tasks that require greater levels of access. You can require an admin to approve this access, and set automatic expiration periods for the elevated role. PIM also includes an audit history to see all role assignments and activations for all privileged roles within the past 30 days (or a longer period that you configure).

Using PIM helps to reduce ongoing admin access and supports the [principle of least privilege](/entra/id-governance/scenarios/least-privileged). For example, you could use PIM to grant certain users the ability to elevate their role to [Azure Connected Machine Resource Administrator](/azure/role-based-access-control/built-in-roles/management-and-governance), allowing them to perform more advanced management tasks on Arc-enabled servers.

## Hybrid identity configurations

In practice, many enterprises run Arc-enabled servers that are also domain-joined to AD. These aren’t mutually exclusive; they complement each other. You might log into the server via AD when needed, but perform management tasks in Azure.

On individual servers, you might still manage local accounts via AD, such as using Local Administrator Password Solution (LAPS) to rotate the local admin password. Since Azure Arc doesn't manage local accounts, you may want to keep using that process. You could even use Azure Policy to ensure that LAPS is enabled and storing passwords in Microsoft Entra.

There's plenty of flexibility to use the capabilities of Microsoft Entra and Azure, while still maintaining on-premises identity solutions that work for you. Over time, you'll find you need to interact less with maintaining service accounts or granting local admin rights, since you can have options managing identities in the cloud.
