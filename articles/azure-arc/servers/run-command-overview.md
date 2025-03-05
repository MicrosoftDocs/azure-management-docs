---
title: Run command on Azure Arc-enabled servers (Preview)
description: 
ms.date: 02/28/2025
ms.topic: overview
ms.custom: devx-track-azurecli, devx-track-azurepowershell
---

# What is Run command on Azure Arc-enabled servers (Preview)?

The Run command on Azure Arc-enabled servers (Public Preview) lets you securely execute scripts or commands on Arc-enabled virtual machines (VMs) without connecting to them directly through Remote Desktop Protocol or SSH. 

Because you don't have to log into each VM individually, the Run command lowers the overhead and effort to perform administrative tasks like installing software, restarting services, or troubleshooting VM performance issues.   

Although there are some differences, the Run command on Azure Arc-enabled servers is similar to the [Run command functionality you can use on Azure VMs](./azure/virtual-machines/run-command-overview). As an example, the Run command on Azure Arc-enabled servers isn't currently available on the Portal.

## How it works

When you use the Run command to execute a script or command from the Azure CLI, PowerShell, or REST API, Azure directs the Connected Machine agent installed on the VM to complete the specified action. You don't have to install any other extensions to the VM.

While the Run command on Azure Arc-enabled servers is free to use, any scripts you store in Azure incur billing charges. 

## Prerequisites

- The Connected Machine agent version on the Arc-enabled server must be 1.33 or higher.


## Support

The Run command is available across many configurations: 

- **Experiences:** Azure CLI, PowerShell, and REST API

- **Operating Systems:** Windows and Linux

- **Environments:** Non-Azure environments including on-premises, VMware, SCVMM, AWS, GCP, and OCI  



