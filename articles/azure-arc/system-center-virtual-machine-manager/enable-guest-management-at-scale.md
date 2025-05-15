---
title: Install Arc agent at scale for your SCVMM VMs
description: Learn how to enable guest management at scale for Arc-enabled SCVMM VMs. 
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
ms.topic: how-to 
ms.date: 04/08/2025
keywords: "VMM, Arc, Azure"

#Customer intent: As an IT infrastructure admin, I want to install arc agents to use Azure management services for SCVMM VMs.
---

# Install Arc agents at scale for Arc-enabled SCVMM VMs

In this article, you learn how to install Azure connected machine agents at scale for SCVMM VMs which is a prerequisite  to use Azure services for securing, patching, monitoring your VMs and leverage on Azure Arc benefits.

>[!IMPORTANT]
>We recommend maintaining the SCVMM management server and the SCVMM console in the same Long-Term Servicing Channel (LTSC) and Update Rollup (UR) version.

>[!NOTE]
>This article is applicable only if you are running:  
>- SCVMM 2025 or later versions of SCVMM server or console
>- SCVMM 2022 UR1 or later versions of SCVMM server or console
>- SCVMM 2019 UR5 or later versions of SCVMM server or console
>- VMs running Windows Server 2012 R2, 2016, 2019, 2022, 2025, Windows 10, and Windows 11  
>For other SCVMM versions, Linux VMs or Windows VMs running WS 2012 or earlier, [install Arc agents through the script](install-arc-agents-using-script.md).

## Prerequisites

Ensure the following before you install Arc agents at scale for SCVMM VMs:

- The Azure Arc resource bridge must be deployed connecting your SCVMM managed environment to Azure and it must be in a *Running* state.
- The SCVMM management server must be in a *Connected* state.
- The user must have permissions listed in Azure Arc SCVMM Contributor build-in role.
- All the target machines are:
    - Powered on.
    - Running a [supported operating system](../servers/prerequisites.md#supported-operating-systems).
    - Able to connect through the firewall to communicate over the internet and [these URLs](../servers/network-requirements.md?tabs=azure-cloud#urls) aren't blocked.

## Install Arc agents at scale from portal

An admin can install agents for multiple machines from the Azure portal if the machines share the same administrator credentials.

1. Navigate to the **SCVMM management servers** blade on [Azure Arc Center](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/AzureArcCenterBlade/~/overview), and select the SCVMM management server resource.
2. Select the machines you want to onboard to Arc at-scale and choose the **Enable in Azure** option.
3. Select **Enable guest management** checkbox to install Arc agents on the selected machine.
4. Based on your organization’s network policies, choose the connectivity method for the Arc agent running in your SCVMM VM to connect to Azure. The available options are Public endpoint, Proxy server and Private endpoint.   
     - If you want to connect the Arc agent via proxy, provide the proxy server details.
     - If you want to connect Arc agent via private endpoint, follow these [steps](../servers/private-link-security.md) to set up Azure private link and provide the same details. 

      >[!Note]
      > Private endpoint connectivity is only available for Arc agent to Azure communications. For Arc resource bridge to Azure connectivity, Azure Private link isn't supported.

5. Provide the administrator username and password for the machine. For Windows VMs, the account must be part of the local administrator group; and for Linux VM, it must be a root account. 

6. Select **Enable** to start the installation of the Arc agent in the specified machines. Once installation is complete, the Guest management column will switch to Enabled for the machines with Arc agent running. You can start using Azure services for these machines. 

Apart from the portal experience, Azure Arc-enabled SCVMM supports at-scale Arc agent installation through Azure CLI, PowerShell, REST APIs, SDKs, and Infrastructure-as-Code mechanisms. Refer to the Reference section in our documentation to know more.

## Next steps

[Manage VM extensions to use Azure management services for your SCVMM VMs](../servers/manage-vm-extensions.md).
