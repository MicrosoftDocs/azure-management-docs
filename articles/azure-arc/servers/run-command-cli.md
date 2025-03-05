---
title: CLI
description: Learn how to use the Azure CLI to remotely execute scripts and commands on Arc-enabled servers.
ms.date: 02/28/2025
ms.topic: conceptual
ms.custom: devx-track-azurecli, devx-track-azurepowershell
---
# Azure CLI requests for Run command on Azure Arc-enabled servers (Preview)

Using the Azure CLI command, [az connectedmachine run-command](/cli/azure/connectedmachine/run-command), you can securely execute scripts or commands on Arc-enabled virtual machines without connecting directly to them through Remote Desktop Protocol or SSH. 

This article provides examples that use `az connectedmachine run-command` to help you understand how to use the Azure CLI to execute scripts or commands on your Arc-enabled server.

## Prerequisites

- The Connected Machine agent version on the Arc-enabled server must be 1.33 or higher.

## Azure CLI sample requests

The following examples use [az connectedmachine run-command](/cli/azure/connectedmachine/run-command) to run a shell script on an Arc-enabled server.

### Execute a script with the machine

This command delivers the script to the machine, executes it, and returns the captured output.

```azurecli
az connectedmachine run-command create --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG" --script "Write-Host Hello World!"
```

### List all deployed RunCommand resources on a machine

This command returns a full list of previously deployed run commands along with their properties.

```azurecli
az connectedmachine run-command list --machine-name "myMachine" --resource-group "myRG"
```

### Get execution status and results

This command retrieves current execution progress, including latest output, start/end time, exit code, and terminal state of the execution.

```azurecli
az connectedmachine run-command show --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
```

> [!NOTE]
> Output and error fields in `instanceView` is limited to the last 4 KB. To access the full output and error, you can forward the output and error data to storage append blobs using `-outputBlobUri` and `-errorBlobUri` parameters while executing Run Command.
> 

### Delete RunCommand resource from the machine

Remove the RunCommand resource previously deployed on the machine. If the script execution is still in progress, execution is terminated.

```azurecli
az connectedmachine run-command delete --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
```

## Related content
- [PowerShell requests for Run command on Azure Arc-enabled servers](run-command-powershell.md)
- [REST API requests for Run command on Azure Arc-enabled servers](run-command-rest.md)
- [What is Run command on Azure Arc-enabled servers?](run-command.md)
