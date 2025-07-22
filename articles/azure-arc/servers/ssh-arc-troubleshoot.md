---
title: Troubleshoot SSH access to Azure Arc-enabled servers
description: Learn how to troubleshoot and resolve issues with SSH access to Arc-enabled servers.
ms.date: 06/12/2025
ms.topic: troubleshooting
# Customer intent: As a system administrator, I want to troubleshoot SSH access issues for Azure Arc-enabled servers, so that I can ensure successful connections and maintain operational efficiency.
---

# Troubleshoot SSH access to Azure Arc-enabled servers

[SSH for Arc-enabled servers](./ssh-arc-overview.md) enables SSH based connections to Arc-enabled servers without requiring a public IP address or additional open ports. This article provides information to help you troubleshoot issues that may occur while attempting to connect to Azure Arc-enabled servers via SSH.

## Client-side issues

Use the information in this section to help resolve issues caused by errors that occur on the machine that you're connecting from.

### Unable to locate client binaries

This issue occurs when the client side SSH binaries required to connect aren't found. Error messages related to this issue include:

- `Failed to create ssh key file with error: \<ERROR\>.`
- `Failed to run ssh command with error: \<ERROR\>.`
- `Failed to get certificate info with error: \<ERROR\>.`
- `Failed to create ssh key file with error: [WinError 2] The system cannot find the file specified.`
- `Failed to create ssh key file with error: [Errno 2] No such file or directory: 'ssh-keygen'.`

To resolve this issue:

- Provide the path to the folder that contains the SSH client executables by [using the `--ssh-client-folder` parameter](/cli/azure/ssh).
- Ensure that the folder is in the PATH environment variable for Azure PowerShell.

### Azure PowerShell module version mismatch

If the installed version of [Az.Ssh](/powershell/module/az.ssh/) doesn't support the installed Azure PowerShell module [Az.Ssh.ArcProxy](https://aka.ms/PowerShellGallery-Az.Ssh.ArcProxy), you see the following error:

- `This version of Az.Ssh only supports version 1.x.x of the Az.Ssh.ArcProxy PowerShell Module. The Az.Ssh.ArcProxy module {ModulePath} version is {ModuleVersion}, and it is not supported by this version of the Az.Ssh module. Check that this version of Az.Ssh is the latest available.`

To resolve this issue, update the Az.Ssh and Az.Ssh.ArcProxy modules to the latest versions by running the following commands:

```powershell
Update-Module -Name Az.Ssh
Update-Module -Name Az.Ssh.ArcProxy
```

### Az.Ssh.ArcProxy not installed

If the Az.Ssh.ArcProxy module isn't installed on the client machine, you see the following error:

- `Failed to find the PowerShell module Az.Ssh.ArcProxy installed in this machine. You must have the Az.Ssh.Proxy PowerShell module installed in the client machine in order to connect to Azure Arc resources. You can find the module in the PowerShell Gallery (see: https://aka.ms/PowerShellGallery-Az.Ssh.ArcProxy).`

To fix this error, install the module from the [PowerShell Gallery](https://aka.ms/PowerShellGallery-Az.Ssh.ArcProxy): `Install-Module -Name Az.Ssh.ArcProxy`

### Insufficient permissions to execute proxy

If your account doesn't have permissions to execute the SSH proxy that is used to connect, you may see the following errors:

- `/bin/bash: line 1: exec: /usr/local/share/powershell/Modules/Az.Ssh.ArcProxy/1.0.0/sshProxy_linux_amd64_1.3.022941: cannot execute: Permission denied`
- `CreateProcessW failed error:5 posix_spawnp: Input/output error`

You can resolve this issue by ensuring that the user account has permissions to execute the SSH proxy on the management machine.

## Server-side issues

Use the information in this section to help resolve issues caused by errors on the Arc-enabled server that you're trying to connect to.

### SSH traffic not allowed on the server

This issue occurs when SSHD isn't running on the server, or SSH traffic isn't allowed on the server. In this case, you may see the following errors:

- `{"level":"fatal","msg":"sshproxy: error copying information from the connection: read tcp 192.168.1.180:60887-\u003e40.122.115.96:443: wsarecv: An existing connection was forcibly closed by the remote host.","time":"2022-02-24T13:50:40-05:00"}`
- `{"level":"fatal","msg":"sshproxy: error connecting to the address: 503 connection to localhost:22 failed: dial tcp [::1]:22: connectex: No connection could be made because the target machine actively refused it.. websocket: bad handshake","proxyVersion":"1.3.022941"}`
- `SSH connection is not enabled in the target port {Port}. `

To resolve this issue:

- Ensure that the SSHD service is running on the Arc-enabled server.
- Ensure that the functionality is enabled on your Arc-enabled server on port 22 (or other nondefault port) by running the following command:

#### [Azure CLI](#tab/azure-cli)

```azurecli
az rest --method put --uri https://management.azure.com/subscriptions/<subscription>/resourceGroups/<resourcegroup>/providers/Microsoft.HybridCompute/machines/<arc enabled server name>/providers/Microsoft.HybridConnectivity/endpoints/default/serviceconfigurations/SSH?api-version=2023-03-15 --body '{\"properties\": {\"serviceName\": \"SSH\", \"port\": 22}}'
```

#### [Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Invoke-AzRestMethod -Method put -Path /subscriptions/<subscription>/resourceGroups/<resourcegroup>/providers/Microsoft.HybridCompute/machines/<arc enabled server name>/providers/Microsoft.HybridConnectivity/endpoints/default/serviceconfigurations/SSH?api-version=2023-03-15 -Payload '{"properties": {"serviceName": "SSH", "port": "22"}}'
```

---

## Azure permissions issues

Use this information to help resolve issues caused by insufficient permissions.

### Incorrect role assignments to enable SSH connectivity

If you don't have the right role assignment to make contributions to the target resource, you see the following error:

- `Client is not authorized to create a Default connectivity endpoint for {Name} in the Resource Group {ResourceGroupName}. This is a one-time operation that must be performed by an account with Owner or Contributor role to allow connections to target resource`

To resolve this error, ensure that you have the Owner or Contributor role on the Arc-enabled server, or ask someone with one of those roles to set up SSH connectivity.

### Incorrect role assignments to connect

This issue occurs if you don't have the proper role assignment on the target resource, specifically a lack of `read` permissions. You may see the following errors:

- `Unable to determine the target machine type as Azure VM or Arc Server`
- `Unable to determine that the target machine is an Arc Server`
- `Unable to determine that the target machine is an Azure VM`
- `Permission denied (publickey).`
- `Request for Azure Relay Information Failed: (AuthorizationFailed) The client '\<user name\>' with object id '\<ID\>' does not have authorization to perform action 'Microsoft.HybridConnectivity/endpoints/listCredentials/action' over scope '/subscriptions/\<Subscription ID\>/resourceGroups/\<Resource Group\>/providers/Microsoft.HybridCompute/machines/\<Machine Name\>/providers/Microsoft.HybridConnectivity/endpoints/default' or the scope is invalid. If access was recently granted, please refresh your credentials.`

To fix this issue, ensure that you have the Virtual Machine Local user Login role on the Arc-enabled server that you're connecting to. If using Microsoft Entra login, ensure you have the Virtual Machine User Login or the Virtual Machine Administrator Login roles and that the Microsoft Entra SSH Login extension is installed on the Arc-Enabled server.

### HybridConnectivity RP not registered

If the HybridConnectivity resource provider isn't registered for the subscription, you may see the following error:

- `Request for Azure Relay Information Failed: (NoRegisteredProviderFound) Code: NoRegisteredProviderFound`

To resolve this issue, register the HybridConnectivity resource provider for the subscription:

- Run ```az provider register -n Microsoft.HybridConnectivity```.
- Confirm success by running ```az provider show -n Microsoft.HybridConnectivity```, and verify that `registrationState` is set to `Registered`.
- Restart the hybrid agent on the Arc-enabled server.

## Next steps

- Learn about SSH access to [Azure Arc-enabled servers](ssh-arc-overview.md).
- Learn about troubleshooting [agent connection issues](troubleshoot-agent-onboard.md).
