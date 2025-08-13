---
title: Cloud-native governance and policy with Azure Arc-enabled servers
description: Azure Policy lets you audit and enforce critical settings, providing a holistic view of your hybrid environment.
ms.date: 08/01/2025
ms.topic: concept-article
# Customer intent: "As a cloud administrator managing a hybrid environment, I want to understand Azure's approach to governance and enforcing policies across resources, so I can manage my hybrid servers consistently and maintain organizational compliance."
---

# Cloud-native governance and policy with Azure Arc-enabled servers

Cloud-native policy management supplements traditional Active Directory Group Policy Objects (GPOs) with Azure-powered audit and enforcement. In a traditional setup, GPOs in Active Directory enforce configurations, such as password policies and audit settings on your servers. Azure provides similar capabilities through [Azure Policy](/azure/governance/policy/overview), which includes a Guest Configuration feature that can audit and configure settings inside VMs.

Azure Policy lets you audit critical settings, with built-in and custom policy definitions, to provide a holistic view of your environment. Over time, you can enforce configurations so that whether a server is on-premises or in Azure, it meets your standards, delivering "policy as code" across your hybrid environment.

Let’s break down how Azure Policy works for servers and how it compares to Group Policy.

## Azure machine configuration

Azure Policy usually applies to cloud resources. [Azure machine configuration](/azure/governance/machine-configuration/overview) (previously called Azure Policy Guest Configuration) extends Azure Policy inside the operating system, giving you GPO-like control over Arc-connected servers. Machine configuration uses Desired State Configuration (DSC) under the hood to check or set OS settings on VMs or Arc-enabled servers. For example, Azure Policy can audit if the password complexity policy on a Windows server meets requirements, or even set it if not. Microsoft provides built-in machine configuration policies for common scenarios, such as ensuring the Windows Firewall is enabled, password minimum length is X, or certain services are running. This capability is similar in spirit to Group Policy templates, and many of the same settings can be enforced by either tool.

Azure Policy continually evaluates compliance, and provides a single dashboard showing all of your machines and their compliance status. To get the same information through Active Directory, you might have to comb through the Resultant Set of Policy (RSoP) on each server or use  additional tools.

One key difference to keep in mind is that Azure Policy is applied at the Azure resource level (subscription, resource group, or individual machine scope), whereas GPOs apply at domain/organizational unit (OU) levels to domain-joined machines. With Azure, you can target a policy to all Arc servers in a resource group (maybe representing a department) in one assignment. Even if those servers aren't in the same OU, they're covered by the policy as long as they're in that Azure scope.

In addition to auditing compliance, you can enforce or remediate policies. For example, if a server's RDP is not set to your required cipher, a remediate policy could flip it to compliant. With Arc-enabled servers, an extension applies the desired setting (via DSC) on each noncompliant machine.

## Desired State Configuration

If you have custom configuration needs, you're not limited to built-in policies. Azure guest config allows you to bring your own DSC configurations or scripts and deploy them as (or in) a policy. This is analogous to writing a custom GPO administrative template or a startup script.

For example, if you have a specific registry setting that isn't covered by any Microsoft-provided policy, you could write a DSC script to check for it and enforce it, then wrap that in an Azure Policy. This way, Azure becomes your one-stop shop to enforce both standard and custom configurations.

## Capability comparison

While both tools can ensure OS settings, such as password policies or firewall rules, Azure Policy governs cloud aspects that Group Policy isn't able to manage. For example, you can use Azure Policy to ensure that VMs have [Azure Backup](/azure/backup/backup-overview) enabled, or to prevent creation of an expensive VM SKU. Azure Policy can also validate that an Arc-connected server is onboarded to Azure services such as [Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction), which is beyond Group Policy's scope.

Microsoft even provides an Azure Policy initiative that maps to the [Windows security baselines](/windows/security/operating-system-security/device-management/windows-security-configuration-framework/windows-security-baselines), covering many of the same settings as the Group Policy security baseline (account policies, audit settings, etc.). This is useful if you want to check all your servers against a known baseline and view compliance percentages.

## Using Azure Policy alongside Group Policy

One big incentive to adopt Azure Policy is central visibility. If you use Azure Policy as the single point of administration for Windows Servers, regardless of whether they're in Azure or not, you get a unified compliance dashboard. You could still let Group Policy do the actual enforcement on-premises, but use Azure Policy to audit those same settings. With this option, Azure can show you if any server is drifting from the desired state. Eventually, you might let Azure actually enforce them and reduce reliance on GPO. This is part of the "cloud admin" journey: you gradually shift governance to Azure.

If you want to continue to use Group Policy on on-premises servers to manage user-centric or UI settings, such as screensaver timeout and mapped drives, or to install MSI packages via GPO. Azure Policy doesn't manage these kinds of settings on a per-user basis, because it's not integrated with the user login process. In a combination approach, you could continue using Active Directory Group Policy for what it's best at (especially if your servers are all domain-joined and you have existing GPOs), but use Azure Policy for cloud-level governance and to manage servers that aren't in your domain.

When using Azure Policy and Group Policy together, it's important to understand how they interact. Generally, Group Policy continues to apply  on domain-joined machines at login/refresh, even if the server is also connected to Azure Arc. Azure Policy machine configuration is applied via the Arc agent, and is checked regularly or when triggered by an event.

Because of this, we recommend that you avoid conflicting configurations. For example, you might use Azure Policy mainly to audit GPO-managed settings for compliance reporting, or gradually replace certain GPOs with Azure policies as you shift to cloud management. Notably, Azure Policy settings don't appear in RSoP or `gpresult` on the server. Group Policy doesn’t "know" that a certain registry setting was applied through Azure Policy; it's only aware of the setting. As an admin, you'll rely on Azure's compliance reports rather than RSOP for details about those settings.


