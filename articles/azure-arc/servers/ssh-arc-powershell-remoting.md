---
title: SSH access to Azure Arc-enabled servers with PowerShell remoting
description: Use PowerShell remoting over SSH to access and manage Azure Arc-enabled servers.
ms.date: 06/13/2025
ms.topic: concept-article
ms.custom: references_regions
# Customer intent: As a system administrator managing Azure Arc-enabled servers, I want to use PowerShell remoting over SSH, so that I can securely access and manage servers without needing public IP addresses or open ports.
---

# PowerShell remoting to Azure Arc-enabled servers

[PowerShell remoting over SSH](/powershell/scripting/security/remoting/ssh-remoting-in-powershell) can be used to [enable SSH connectivity on Arc-enabled servers](ssh-arc-overview.md).

## Prerequisites

To use PowerShell remoting over SSH access to Azure Arc-enabled servers, you must:

- Meet the prerequisites for [SSH access to Azure Arc-enabled servers](ssh-arc-overview.md#prerequisites).
- Meet the requirements for [PowerShell remoting over SSH](/powershell/scripting/security/remoting/ssh-remoting-in-powershell#general-setup-information).
- Ensure the Azure PowerShell module ([Az.Ssh](/powershell/module/az.ssh)) or the Azure CLI extension ([az ssh](/cli/azure/ssh)) is installed on the client machine.

## Connect via PowerShell remoting

Complete the following steps to connect via PowerShell remoting to an Arc-enabled server.

### Generate the SSH config file

#### [Azure CLI](#tab/azure-cli)

```bash
az ssh config --resource-group <myRG> --name <myMachine> --local-user <localUser> --resource-type Microsoft.HybridCompute --file <SSH config file>
```

#### [Azure PowerShell](#tab/azure-powershell)

 ```powershell
Export-AzSshConfig -ResourceGroupName <myRG> -Name <myMachine> -LocalUser <localUser> -ResourceType Microsoft.HybridCompute/machines -ConfigFilePath <SSH config file>
```

 ---

### Find the newly created entry in the SSH config file

Open the created or modified SSH config file. The entry should have a similar format to the following sample file:

```powershell
Host <myRG>-<myMachine>-<localUser>
    HostName <myMachine>
    User <localUser>
    ProxyCommand "<path to proxy>\.clientsshproxy\sshProxy_windows_amd64_1_3_022941.exe" -r "<path to relay info>\az_ssh_config\<myRG>-<myMachine>\<myRG>-<myMachine>-relay_info"
```

### Use the `-Options` parameter

Using the [`-Options`](/powershell/module/microsoft.powershell.core/new-pssession#-options) parameter allows you to specify a hashtable of SSH options used when connecting to a remote SSH-based session.

Create the hashtable using the format of the following sample. Be mindful of the locations of quotation marks.

```powershell
$options = @{ProxyCommand = '"<path to proxy>\.clientsshproxy\sshProxy_windows_amd64_1_3_022941.exe -r <path to relay info>\az_ssh_config\<myRG>-<myMachine>\<myRG>-<myMachine>-relay_info"'}
```

Next, use the `-Options` hashtable in a PowerShell remoting command:

```powershell
New-PSSession -HostName <myMachine> -UserName <localUser> -Options $options
```

## Next steps

- Learn about [OpenSSH for Windows](/windows-server/administration/openssh/openssh_overview).
- Learn about troubleshooting [SSH access to Azure Arc-enabled servers](ssh-arc-troubleshoot.md).
- Learn about troubleshooting [agent connection issues](troubleshoot-agent-onboard.md).
