---
title: Cloud-native server management
description: Azure Arc transforms traditional server management by extending Azure’s control plane to on-premises and multi-cloud servers. 
ms.date: 08/01/2025
ms.topic: overview
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand how to extend Azure's management capabilities to my on-premises and multi-cloud servers."
---

# Cloud-native server management

With [Azure Arc-enabled servers](../overview.md), your Windows and Linux machines outside Azure (in your datacenter or other clouds) become Azure resources. This means you can manage them just like Azure virtual machines (VMs)—organizing them into resource groups, applying policies, running scripts, and tagging them for search—all from the Azure Portal or CLI. The journey isn't just about moving VMs to Azure; it’s about shifting the entire management experience (inventory, configuration, governance, scripting, patching, identity) into Azure's unified platform.

Once a server is connected via the [Azure Connected Machine agent](../agent-overview.md), it gets a unique Azure Resource ID and appears in your subscription along with native Azure resources. Tasks that used to require disparate on-premises tools (such as Active Directory Group Policy, Microsoft Configuration Manager (SCCM), or PowerShell remoting) can be done through Azure. For example, you can use [Azure Policy](/azure/governance/policy/overview) to audit or enforce OS settings (similar to GPO), [Azure Update Manager](/azure/update-manager/overview) to schedule patches (replacing WSUS/SCCM maintenance plans), and [Run Command](../run-command.md) to execute scripts remotely (instead of RDP/SSH into each server). Azure Arc brings cloud practices such as centralized management, at-scale automation, and uniform governance to all your servers, empowering you as a system administrator to manage hybrid infrastructure from a "single pane of glass" in Azure.

