---
title: How to remotely and securely configure servers using Run command (Preview)
description: Learn how to remotely and securely configure servers using Run Command.
ms.date: 02/07/2024
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-azurepowershell
---

What? Use the Run command to securely execute scripts or commands on virtual machines (VMs) without connecting to them directly through Remote Desktop Protocol or SSH. 

Why? The Run command lowers the overhead and effort to perform administrative tasks like installing software, restarting services, or troublehsooting VM performance issues.  

How? When you execute a script or command from the Azure CLI or Powershell, Azure directs the Connected Machine agent installed on the VM

Benefits:
- Task automation
- Troubleshooting
- Security - 
- Cross-platform support

1. Configure 


## Supported environment and configuration

The Run command has the following support: 

- **Experiences:** Azure CLI and PowerShell 

- **Operating Systems:** Windows and Linux

- **Environments:** Non-Azure environments including on-premises, VMware, SCVMM, AWS, GCP, and OCI  

- **Cost:** Free of charge. However, storage of scripts in Azure may incur billing.

- **Configuration:** Doesn't require more configuration or the deployment of any extensions. The
Connected Machine agent version must be 1.33 or higher.

## Control Run command
