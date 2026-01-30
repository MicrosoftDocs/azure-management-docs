---
title: Run command on Azure Arc-enabled servers (Preview)
description: Learn about the Run command on Arc-enabled servers.
ms.date: 02/28/2025
ms.topic: overview
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: "As a system administrator managing hybrid infrastructure, I want to remotely execute scripts on Arc-enabled servers, so that I can efficiently perform administrative tasks and enhance security without needing to log into each VM directly."
---

# What is Run command on Azure Arc-enabled servers (Preview)?

The Run command on Azure Arc-enabled servers (Preview) lets you remotely and securely execute scripts or commands on Arc-enabled virtual machines (VMs) without connecting to them directly through Remote Desktop Protocol or SSH. 

Because you don't have to log into each VM individually, Run command lowers the overhead and effort to perform administrative tasks like installing or updating software, configuring firewall rules, running health checks, or troubleshooting issues.   

One key use case is using Run command to enhance your security posture. You can use Run command to remotely apply security patches, enforce compliance policies, or remediate vulnerabilities on your Arc-enabled servers. You can also automate common security tasks, such as rotating passwords, encrypting data, or auditing logs. Through Azure Arc, you can perform these tasks consistently across your hybrid, multicloud, and edge environments, helping reduce operational overhead and response time.

> [!NOTE]
> Although there are some differences, the Run command on Azure Arc-enabled servers is similar to the [Run command functionality you can use on Azure VMs](/azure/virtual-machines/run-command-overview) - including the restrictions outlined. As an example of the differences, the Run command on Azure Arc-enabled servers isn't currently available on the Portal.

## How it works

The Run command is built in to the Connected Machine agent and supports not just the ability to run scripts but to centralize script management across creation, update, deletion, sequencing, and listing operations.

When you use the Run command to execute a script or command from the Azure CLI, PowerShell, or REST API, Azure directs the Connected Machine agent installed on the VM to complete the specified action. You don't have to install any other extensions to the VM.

While the Run command on Azure Arc-enabled servers is free to use, any scripts you store in Azure incur billing charges. 

## Prerequisites

- The Connected Machine agent version on the Arc-enabled server must be 1.33 or higher.


## Support

The Run command is available across many configurations: 

- **Experiences:** Azure CLI, PowerShell, and REST API

- **Operating Systems:** Windows and Linux

- **Environments:** Non-Azure environments including on-premises, VMware, SCVMM, AWS, GCP, and OCI  

> [!Important] 
> Run command on Azure Arc-enabled servers does not support authenticating blobs using Managed Identities yet. 

> [!NOTE]
> RunCommand on Linux does not allow names longer than 36 characters.

## Next steps
Learn how to use Run command:
- [Azure CLI requests for Run command on Azure Arc-enabled servers](run-command-cli.md)
- [PowerShell requests for Run command on Azure Arc-enabled servers](run-command-powershell.md)
- [REST API requests for Run command on Azure Arc-enabled servers](run-command-rest.md)


