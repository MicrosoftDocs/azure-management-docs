---
title: Run command on Azure Arc-enabled servers (preview)
description: The Run command lets you remotely and securely execute scripts or commands on Azure Arc-enabled servers.
ms.date: 02/23/2026
ms.topic: overview
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: "As a system administrator managing hybrid infrastructure, I want to remotely execute scripts on Arc-enabled servers, so that I can efficiently perform administrative tasks and enhance security without needing to log into each VM directly."
---

# What is Run command on Azure Arc-enabled servers (preview)?

The Run command on Azure Arc-enabled servers (preview) lets you remotely and securely execute scripts or commands on virtual machines (VMs) connected to Azure Arc, without requiring a direct connection through Remote Desktop Protocol or SSH.

Because you don't have to sign in to each VM individually, Run command lowers the overhead and effort to perform administrative tasks like installing or updating software, configuring firewall rules, running health checks, or troubleshooting problems.

One key use case is using Run command to enhance your security posture. You can use Run command to remotely apply security patches, enforce compliance policies, or remediate vulnerabilities on your Azure Arc-enabled servers. You can also automate common security tasks, such as rotating passwords, encrypting data, or auditing logs. Through Azure Arc, you can perform these tasks consistently across your hybrid, multicloud, and edge environments, helping reduce operational overhead and response time.

> [!NOTE]
> Although there are some differences, the Run command on Azure Arc-enabled servers is similar to the [Run command functionality you can use on Azure VMs](/azure/virtual-machines/run-command-overview), including the restrictions outlined. As an example of the differences, the Run command on Azure Arc-enabled servers isn't currently available in the Azure portal.

## How it works

The Run command is built in to the Connected Machine agent (starting with version 1.33) and supports the ability to run scripts and to centralize script management across creation, update, deletion, sequencing, and listing operations.

When you use the Run command to execute a script or command from the Azure CLI, PowerShell, or REST API, Azure directs the Connected Machine agent installed on the VM to complete the specified action. You don't have to install any other extensions to the VM.

While the Run command on Azure Arc-enabled servers is free to use, scripts you store in Azure incur billing charges.

> [!Important]
> Run command on Azure Arc-enabled servers doesn't currently support authenticating blobs by using managed identities.

## Supported configurations

The Run command is available across many configurations:

- **Experiences:** Azure CLI, PowerShell, and REST API
- **Operating systems:** Windows and Linux
- **Environments:** Non-Azure environments connected to Azure Arc, including on-premises, VMware, SCVMM, AWS, GCP, and OCI  

> [!NOTE]
> On Linux, Run command doesn't accept names longer than 36 characters.

## Use the Run command

To use the Run command, create a script that contains the commands you want to run on the VM. Then, execute the script by using [Azure PowerShell](/powershell/module/az.connectedmachine), the [Azure CLI](/cli/azure/connectedmachine/run-command), or [REST API](/rest/api/hybridcompute/machine-run-commands). This section includes examples of how to use the Run command with each experience.

### [Azure PowerShell](#tab/azure-powershell)

The following examples use the [Az.ConnectedMachine module](/powershell/module/az.connectedmachine) for Azure PowerShell to execute scripts or commands on an Arc-enabled server.

#### Execute a script on a machine

This command sends the script to the machine, runs it, and returns the captured output.

```powershell-interactive
New-AzConnectedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine" -Location "eastus" -RunCommandName "RunCommandName" –SourceScript "echo Hello World!"
```

> [!NOTE]
> You can add multiple commands in the `-SourceScript` parameter. Use `;` to separate each command. For example: `–SourceScript "id; echo Hello World!"`

#### Execute a script on the machine using a script file in storage

This command directs the Connected Machine agent to a shared access signature (SAS) URI for a storage blob where a script was uploaded. Then, it directs the agent to execute the script and return the captured output.

```powershell-interactive
New-AzConnectedMachineRunCommand -ResourceGroupName "MyRG0" -MachineName "MyMachine" -RunCommandName "MyRunCommand" -Location "eastus" -SourceScriptUri “< SAS URI of a storage blob with read access or public URI>”
```

> [!NOTE]
> The `scriptUri` is a shared access signature (SAS) URI for the storage blob, and it must provide read access to the blob. An expiration time of 24 hours is suggested for the SAS URI. You can generate a SAS URI on Azure portal by using blobs options or generate a SAS token by using `New-AzStorageBlobSASToken`. If you generate a SAS token by using `New-AzStorageBlobSASToken`, the SAS URI format is: `base blob URL + "?"` + the SAS token from `New-AzStorageBlobSASToken`.

#### List all deployed Run command resources on a machine

This command returns a full list of previously deployed Run Commands along with their properties.

```powershell-interactive
Get-AzConnectedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine"
```

#### Get execution status and results

This command retrieves current execution progress for a Run command, including latest output, start and end time, exit code, and terminal state of the execution.

```powershell-interactive
Get-AzConnectedMachineRunCommand -ResourceGroupName "myRG" - MachineName "myMachine" -RunCommandName "RunCommandName"
```

#### Get status information for a Run command through Instance View

This command gets status information for a Run command on machine with Instance View. Instance View contains the execution state of the Run command (succeeded, failed, and so on), exit code, standard output, and standard error generated by executing the script. A nonzero exit code indicates an unsuccessful execution.

```powershell-interactive
Get-AzConnectedMachineRunCommand -ResourceGroupName "MyRG" -MachineName "MyMachine" -RunCommandName "MyRunCommand"
```

Along with other information, the response returns these fields:

- `InstanceViewExecutionState`: Indicates whether your script was successful or not.
- `ProvisioningState`: Indicates whether the extension platform was able to trigger the Run command script or not.

#### Create or update Run Command on a machine 

This command creates or updates a Run command on a machine and streams standard output and standard error messages to output and error AppendBlobs.

```powershell-interactive
New-AzConnectedMachineRunCommand -ResourceGroupName "MyRG0" - MachineName "MyMachine" -RunCommandName "MyRunCommand3" -Location "eastus" -SourceScript "id; echo HelloWorld" -OutputBlobUri <OutPutBlobUrI> -ErrorBlobUri <ErrorBlobUri>
```

> [!NOTE]
> Output and error blobs must be the AppendBlob type and their SAS URIs must provide read, append, create, and write access to the blob. An expiration time of 24 hours is suggested for SAS URI. If the output or error blob doesn't exist, a blob of type AppendBlob is created. You can generate a SAS URIs on the Azure portal by using blob's options or generate a SAS token by using `New-AzStorageBlobSASToken`.

#### Create or update Run Command on a machine as a different user

This command creates or updates a Run command on a machine as a different user by using the `RunAsUser` and `RunAsPassword` parameters.

Before using this command:

- Contact the administrator of the machine and make sure the user has access to the machine.
- Make sure the user has access to the resources accessed by the Run command, such as directories, files, and network resources.
- On a Windows machine, make sure 'Secondary Logon' is running.

```powershell-interactive
New-AzMachineRunCommand -ResourceGroupName "MyRG0" -MachineName "MyMachine" -RunCommandName "MyRunCommand" -Location "eastus" -SourceScript "id; echo HelloWorld" -RunAsUser myusername -RunAsPassword mypassword
```

#### Create or update Run command on a machine with a local script file

This command creates or updates a Run command on a machine by using a local script file on the client machine where `cmdlet` runs.

```powershell-interactive
New-AzConnectedMachineRunCommand -ResourceGroupName "MyRG0" -VMName "MyMachine" -RunCommandName "MyRunCommand" -Location "eastus" -ScriptLocalPath "C:\MyScriptsDir\MyScript.ps1"
```

#### Create or update Run command on a machine while passing sensitive inputs to the script

This command creates or updates Run command with `ProtectedParameter` specified to pass sensitive inputs to a script, such as passwords or keys.

```azurepowershell-interactive
$privateParametersArray = @{name='inputText';value='privateParam1value'}

New-AzConnectedMachineRunCommand -MachineName "MyMachine" -ResourceGroupName "MyRG0" -RunCommandName "MyRunCommand" -Location "eastus" -SourceScriptUri <SourceScriptUri> -ProtectedParameter $privateParametersArray 
```

Sample script for capturing inputText:

```azurepowershell-interactive
param ([string]$inputText)
Write-Output $inputText
```

You can also pass public parameters in a similar way by using `Parameter`.

- For Windows: Parameter and ProtectedParameter are passed to a script similar to the following example: `myscript.ps1 -publicParam1 publicParam1value -publicParam2 publicParam2value -secret1 secret1value -secret2 secret2value`

- For Linux: A named `Parameter` and its values are set to environment config, which should be accessible within the PowerShell script. For nameless arguments, pass an empty string to name input. Nameless arguments are passed to script similar to the following example: `myscript.sh publicParam1value publicParam2value secret1value secret2value`

#### Delete Run command resource from the machine

This command removes the Run command resource previously deployed on the machine. If the script execution is still in progress, execution terminates.

```powershell-interactive
Remove-AzConnectedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine" -RunCommandName "RunCommandName"
```

### [Azure CLI](#tab/azure-cli)

The following examples use [`az connectedmachine run-command`](/cli/azure/connectedmachine/run-command) to run a shell script on an Arc-enabled server.

#### Execute a script on a machine

This command sends the script to the machine, runs it, and returns the captured output.

```azurecli-interactive
az connectedmachine run-command create --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG" --script "Write-Host Hello World!"
```

#### List all deployed Run command resources on a machine

This command returns a full list of previously deployed Run commands along with their properties.

```azurecli-interactive
az connectedmachine run-command list --machine-name "myMachine" --resource-group "myRG"
```

#### Get execution status and results

This command retrieves current execution progress for a Run command, including latest output, start and end time, exit code, and terminal state of the execution.

```azurecli-interactive
az connectedmachine run-command show --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
```

> [!NOTE]
> The output and error fields in `instanceView` are limited to the last 4 KB. To access the full output and error, you can forward the output and error data to storage append blobs by using `-outputBlobUri` and `-errorBlobUri` parameters while executing the Run command.

#### Delete the Run command resource from a machine

This command removes the Run command resource previously deployed on the machine. If the script execution is still in progress, execution is terminated.

```azurecli-interactive
az connectedmachine run-command delete --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
```

### [REST API](#tab/rest-api)

The example commands in this section walk you through using the Run command to remotely configure a firewall rule that allows access to the endpoint `www.microsoft.com/pkiops/certs`. The example scenario uses each of the available REST operations.

These examples use the following values:

- **Subscription ID** - `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`
- **Arc-enabled server name** -  `2012DatacenterServer1`
- **Resource group** - `ContosoRG`
- **Server's operating system** - Windows Server 2012 / R2 servers

> [!TIP]
> You need an access token to use the REST API. See [Getting Started with REST](/rest/api/azure/).

#### Create endpoint access with a Run Command

First, create a Run command script to provide access to the `www.microsoft.com/pkiops/certs` endpoint on your target Arc-enabled server using the PUT operation.

You can either provide the script inline or you can link to the script file.

To provide the script inline:

```rest
PUT https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

```json
{
  "location": "eastus2",
  "properties": {
    "source": {
      "script": "New-NetFirewallRule -DisplayName $ruleName -Direction Outbound -Action Allow -RemoteAddress $endpoint -RemotePort $port -Protocol $protocol"
    },
    "parameters": [
      {
        "name": "ruleName",
        "value": "Allow access to www.microsoft.com/pkiops/certs"
      },
      {
        "name": "endpoint",
        "value": "www.microsoft.com/pkiops/certs"
      },
      {
        "name": "port",
        "value": 433
      },
      {
        "name": "protocol",
        "value": "TCP"
      }
    ],
    "asyncExecution": false,
    "runAsUser": "contoso-user1",
    "runAsPassword": "Contoso123!"
    "timeoutInSeconds": 3600,
    "outputBlobUri": "https://mystorageaccount.blob.core.windows.net/myscriptoutputcontainer/MyScriptoutput.txt",
    "errorBlobUri": "https://mystorageaccount.blob.core.windows.net/mycontainer/MyScriptError.txt"
  }
}
```

To link to the script file:

```rest
PUT https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

```json
{
  "location": "eastus2",
  "properties": {
    "source": {
      "scriptUri": "https://mystorageaccount.blob.core.windows.net/myscriptoutputcontainer/newnetfirewallrule.ps1"
    },
    "parameters": [
      {
        "name": "ruleName",
        "value": " Allow access to www.microsoft.com/pkiops/certs"
      },
      {
        "name": "endpoint",
        "value": "www.microsoft.com/pkiops/certs"
      },
      {
        "name": "port",
        "value": 433
      },
      {
        "name": "protocol",
        "value": "TCP"
      }
    ],
    "asyncExecution": false,
    "runAsUser": "contoso-user1",
    "runAsPassword": "Contoso123!"
    "timeoutInSeconds": 3600,
    "outputBlobUri": "https://mystorageaccount.blob.core.windows.net/myscriptoutputcontainer/MyScriptoutput.txt",
    "errorBlobUri": "https://mystorageaccount.blob.core.windows.net/mycontainer/MyScriptError.txt"
  }
}
```

The `scriptUri` is a shared access signature (SAS) URI for the storage blob, and it must provide read access to the blob. An expiration time of 24 hours is suggested for the SAS URI. You can generate a SAS URI on Azure portal using blobs options or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, the SAS URI format is: `base blob URL + "?"` + the SAS token from `New-AzStorageBlobSASToken`.

Output and error blobs must be the AppendBlob type and their SAS URIs must provide read, append, create, and write access to the blob. An expiration time of 24 hours is suggested for SAS URI. You can generate SAS URIs on Azure portal using blob's options, or SAS token from using `New-AzStorageBlobSASToken`.

To learn more about shared access signatures, see [Grant limited access to Azure Storage resources using shared access signatures (SAS)](/azure/storage/common/storage-sas-overview).

#### Verify Run command details

Verify that you correctly provisioned the Run command by using the GET operation:

```rest
GET https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

### Update the Run command

You can update the existing Run command with new parameters by using the PATCH operation. The following example adds access to another endpoint, `*.waconazure.com`, for connectivity to Windows Admin Center.

```rest
PATCH https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

```json
{
  "location": "eastus2",
  "properties": {
    "source": {
      "script": "New-NetFirewallRule -DisplayName $ruleName -Direction Outbound -Action Allow -RemoteAddress $endpoint -RemotePort $port -Protocol $protocol"
    },
    "parameters": [
      {
        "name": "ruleName",
        "value": "Allow access to WAC endpoint"
      },
      {
        "name": "endpoint",
        "value": "*.waconazure.com"
      },
      {
        "name": "port",
        "value": 433
      },
      {
        "name": "protocol",
        "value": "TCP"
      }
    ],
    "asyncExecution": false,
    "runAsUser": "contoso-user1",
    "runAsPassword": "Contoso123!",
    "timeoutInSeconds": 3600,
    "outputBlobUri": "https://mystorageaccount.blob.core.windows.net/myscriptoutputcontainer/MyScriptoutput.txt",
    "errorBlobUri": "https://mystorageaccount.blob.core.windows.net/mycontainer/MyScriptError.txt"
  }
}
```

### List Run commands

Before you delete the Run command for endpoint access, make sure the Arc-enabled server has no other Run commands. Use the LIST operation to get all of the Run commands on the Arc-enabled server:

```rest
LIST https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands?api-version=2024-11-10-preview
```

### Delete a Run command

When you no longer need the Run command extension, delete it by using the following DELETE operation:

```rest
DELETE https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```
---

## Limit access to Run command (Preview)

While remote access enabled by the Run command lowers overhead for performing certain tasks on a virtual machine (VM), you can also limit remote access.

### Manage access to Run command by using Azure role-based access control (Azure RBAC)

Use Azure RBAC to control which users can execute commands and scripts by using the Run command.

The following table describes a Run command action, the permission needed to perform the action, and the RBAC role that grants the permission.

| Action | Permission | RBAC with permission |
|---------|---------|---------|
| List Run commands or show details of the command | `Microsoft.HybridCompute/machines/runCommands/read` | Built-in [Reader](/azure/role-based-access-control/built-in-roles) role and higher |
| Run a command | `Microsoft.HybridCompute/machines/runCommands/write` | [Azure Connected Machine Resource Administrator](/azure/role-based-access-control/built-in-roles) role and higher |

To control access to the Run command functionality, use one of the [built-in roles](/azure/role-based-access-control/built-in-roles) or create a [custom role](/azure/role-based-access-control/custom-roles) that grants a Run command permission.

### Block run commands locally

You can control whether the Connected Machine agent allows access to the VM through Run commands by adding the Run command extension to an allowlist (inclusive) or a blocklist (exclusive).

For more information, see [Extension allowlists and blocklists](security-extensions.md#allowlists-and-blocklists).

The following example adds the Run command extension to a blocklist on a Windows VM:

```azurecli
azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerwindows"
```

This example adds the Run command extensions to an allowlist on a Linux VM:

```azurecli
azcmagent config set extensions.allowlist "microsoft.cplat.core/runcommandhandlerlinux"`
```

## Next steps

To learn how to use Run command, see the following resources:
- [Azure CLI requests for Run command on Azure Arc-enabled servers](run-command-cli.md)
- [PowerShell requests for Run command on Azure Arc-enabled servers](run-command-powershell.md)
- [REST API requests for Run command on Azure Arc-enabled servers](run-command-rest.md)


