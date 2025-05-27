---
title: Limit access to Run command
description: Learn how to use RBAC roles and allowlists and blocklists to limit who can use Run commands and where they can execute.
ms.date: 02/28/2025
ms.topic: concept-article
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As an IT administrator, I want to limit access to Run commands using RBAC roles and allowlists/blocklists, so that I can enhance the security of remote access to virtual machines and control who can execute commands.
---

# Limit access to Run commands (Preview)

While remote access enabled by the Run command lowers overhead for performing certain tasks on a virtual machine (VM), there are a few ways you can make sure remote access to the Arc-enabled server is limited.

- Limit access to the Run command in a subscription
- Allow or block Run commands on specific servers locally 

## Limit access to Run command using role-based access (RBAC)

You can use RBAC to control what roles in a subscription are able to execute commands and scripts with the Run command. The following table describes action you can take with the Run command, the permission needed to perform the action, and the RBAC role that grants the permission.

|Action  |Permission  | RBAC with permission |
|---------|---------|---------|
|List Run commands or show details of the command|`Microsoft.HybridCompute/machines/runCommands/read`|Built-in [Reader](/azure/role-based-access-control/built-in-roles) role and higher|
|Run a command|`Microsoft.HybridCompute/machines/runCommands/write`|[Azure Connected Machine Resource Administrator](/azure/role-based-access-control/built-in-roles) role and higher|

To control access to the Run command functionality, use one of the [built-in roles](/azure/role-based-access-control/built-in-roles) or create a [custom role](/azure/role-based-access-control/custom-roles) that grants a Run command permission.

## Block run commands locally

You can control whether the Connected Machine agent allows access to the VM through Run commands by adding the Run command extension to an allowlist (inclusive) or a blocklist (exclusive). 

> [!TIP]
> If you wanted to disable the Run command at some time in the future, you'd add the Run command extension to a blocklist. 

To learn more, see [Extension allowlists and blocklists](security-extensions.md#allowlists-and-blocklists).

### Windows example
The following example adds the Run command extension to a blocklist on a Windows VM.

```azurecli
azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerwindows"
```

### Linux example
The following example adds the Run command extensions to an allowlist on a Linux VM.

```azurecli
azcmagent config set extensions.allowlist "microsoft.cplat.core/runcommandhandlerlinux"`
```

## Related content
- [What is Run command on Azure Arc-enabled servers?](run-command.md)
