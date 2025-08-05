---
title: Cloud-native governance and policy
description: Cloud-native inventory means all your servers, Azure or not, show up in one consolidated view, and you use Azure's organizational tools to sort and manage them.
ms.date: 08/01/2025
ms.topic: concept-article
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand Azure's approach to governance and enforcing policies across resources, so I can manage my hybrid servers consistently and maintain organizational compliance."
---

# Cloud-native governance and policy

Cloud-native policy management supplements traditional Group Policy Objects (GPOs) with Azure-powered audit and enforcement. In a traditional setup, GPOs in Active Directory enforce configurations, such as password policies and audit settings on your servers. Azure provides similar capabilities through [Azure Policy](/azure/governance/overview), which includes a Guest Configuration feature that can audit and configure settings inside VMs.

Azure Policy lets you audit critical settings, with built-in definitions for password policies, encryption, and more, letting you get a holistic view of your environment. Over time, you can enforce configurations so that whether a server is on-premises or in Azure, it meets your standards, delivering "policy as code" across your hybrid environment.

Let’s break down how Azure Policy works for servers and how it compares to Group Policy.

## Azure machine configuration

Azure Policy usually applies to cloud resources. [Azure machine configuration](/azure/governance/machine-configuration/overview) (previously called Azure Policy Guest Configuration) extends Azure Policy inside the operating system, giving you GPO-like control over Arc-connected servers. Machine configuration uses Desired State Configuration (DSC) under the hood to check or set OS settings on VMs or Arc-enabled servers. For example, Azure Policy can audit if the password complexity policy on a Windows server meets requirements, or even set it if not. Microsoft provides built-in machine configuration policies for common scenarios, such as ensuring the Windows Firewall is enabled, password minimum length is X, or certain services are running. This capability is similar in spirit to Group Policy templates, and many of the same settings can be enforced by either tool.
