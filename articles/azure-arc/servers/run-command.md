---
title: How to remotely and securely configure servers using Run command (Preview)
description: Learn how to remotely and securely configure servers using Run Command.
ms.date: 02/07/2024
ms.topic: how-to
ms.custom: devx-track-azurecli, devx-track-azurepowershell
---

# Remotely and securely configure servers using Run command (Preview)

Use the Run command on Azure Arc-enabled servers (Public Preview) to securely execute scripts or commands on virtual machines (VMs) without connecting to them directly through Remote Desktop Protocol or SSH. 

Because you don't have to log into each VM individually, the Run command lowers the overhead and effort to perform administrative tasks like installing software, restarting services, or troubleshooting VM performance issues.   

When you use the Run command to execute a script or command from the Azure CLI or PowerSShell, Azure directs the Connected Machine agent installed on the VM to complete the specified action. You don't have to install any additional extensions to the VM.

## Prerequisites

- The Connected Machine agent version on the VM must be 1.33 or higher.


## Supported environments

The Run command is available across many configurations: 

- **Experiences:** Azure CLI and PowerShell 

- **Operating Systems:** Windows and Linux

- **Environments:** Non-Azure environments including on-premises, VMware, SCVMM, AWS, GCP, and OCI  

- **Cost:** Free of charge. However, storage of scripts in Azure may incur billing. 


## Run Command operations

Run Command on Azure Arc-enabled servers supports the following operations:

|Operation  |Description  |
|---------|---------|
|Create |The operation to create a run command. This runs the run command. |
|Delete |The operation to delete a run command. If it's running, delete will also stop the run command. |
|Get |The operation to get a run command. |
|List |The operation to get all the run commands of an Azure Arc-enabled server. |
|Update |The operation to update the run command. This stops the previous run command. |
 
> [!NOTE]
> Output and error blobs are overwritten each time the run command script executes.


## Azure CLI sample requests

The following examples use [az connectedmachine run-command](/cli/azure/connectedmachine/run-command) to run a shell script on an Azure Windows machine.

- Execute a script and return the captured output
  ```azurecli
  az connectedmachine run-command create --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG" --script "Write-Host Hello World!"
  ```
- List all previously deployed Run commands and their properties
  ```azurecli
  az connectedmachine run-command list --machine-name "myMachine" --resource-group "myRG"
  ```
- Get execution progress, including the latest output, start/end time, exit code, and terminal state of the execution.
  ```azurecli
  az connectedmachine run-command show --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
  ```
  > [!NOTE]
  > Output and error fields in `instanceView` is limited to the last 4KB. To access the full output and error, you can forward the output and error data to storage append blobs using `-outputBlobUri` and `-errorBlobUri` parameters while executing Run Command.
- Remove the script, deployed by the Run command, from a machine. If the script is exectuing, the script is terminated.
  ```azurecli
  az connectedmachine run-command delete --name "myRunCommand" --machine-name "myMachine" --resource-group "myRG"
  ```

## PowerShell sample requests

The following examples use [New-AzConnectedMachineRunCommand](/powershell/module/az.connectedmachine/new-azconnectedmachineruncommand) to run a shell script on an Azure Windows machine.

- Execute a script and return the captured output
  ```powershell
  New-AzConnectedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine" -Location "EastUS" -RunCommandName "RunCommandName" –SourceScript "echo Hello World!"
  ``` 
- Execute a script on the machine using SourceScriptUri parameter.`OutputBlobUri` and `ErrorBlobUri` are optional parameters.
  ```powershell
  New-AzConnectedMachineRunCommand -ResourceGroupName -MachineName -RunCommandName -SourceScriptUri “< SAS URI of a storage blob with read access or public URI>” -OutputBlobUri “< SAS URI of a storage append blob with read, add, create, write access>” -ErrorBlobUri “< SAS URI of a storage append blob with read, add, create, write access>”
  ```
- Returns a full list of previously deployed Run Commands along with their properties.
  ```powershell
  Get-AzConnectedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine"
  ```
- Get execution progress, including the latest output, start/end time, exit code, and terminal state of the execution.
  ```powershell
  Get-AzConnectedMachineRunCommand -ResourceGroupName "myRG" - MachineName "myMachine" -RunCommandName "RunCommandName"
  ```
- Create or update Run Command on a Windows machine using a SAS URL of a storage blob that contains a PowerShell script. `SourceScriptUri` can be a storage blob’s full SAS URL or public URL.
  ```powershell
  New-AzConnectedMachineRunCommand -ResourceGroupName MyRG0 -MachineName MyMachine -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri>
  ```
  > [!NOTE]
  > SAS URL must provide read access to the blob. An expiration time of 24 hours is suggested for SAS URL. SAS URLs can be generated on the Azure portal using blob options, or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, your SAS URL = "base blob URL" + "?" + "SAS token from `New-AzStorageBlobSASToken`"
- Get a Run command for machine with Instance View. Instance View contains the execution state of run command (Succeeded, Failed, etc.), exit code, standard output, and standard error generated by executing the script using Run Command. A non-zero ExitCode indicates an unsuccessful execution.
  ```powershell
  Get-AzConnectedMachineRunCommand -ResourceGroupName MyRG -MachineName MyMachine -RunCommandName MyRunCommand
  ```
- 


### Create or update Run Command on a machine using SourceScript (script text)

Create or update Run Command on a machine passing the script content directly to `-SourceScript` parameter. Use `;` to separate multiple commands.

```powershell
New-AzConnectedMachineRunCommand -ResourceGroupName MyRG0 -MachineName MyMachine -RunCommandName MyRunCommand2 -Location EastUS2EUAP -SourceScript "id; echo HelloWorld"
```

### Create or update Run Command on a machine using OutputBlobUri, ErrorBlobUri to stream standard output and standard error messages to output and error Append blobs

Create or update Run Command on a machine and stream standard output and standard error messages to output and error Append blobs.

```powershell
New-AzConnectedMachineRunCommand -ResourceGroupName MyRG0 - MachineName MyMachine -RunCommandName MyRunCommand3 -Location EastUS2EUAP -SourceScript "id; echo HelloWorld"-OutputBlobUri <OutPutBlobUrI> -ErrorBlobUri <ErrorBlobUri>
```

> [!NOTE]
> Output and error blobs must be the AppendBlob type and their SAS URLs must provide read, append, create, write access to the blob. An expiration time of 24 hours is suggested for SAS URL. If output or error blob does not exist, a blob of type AppendBlob will be created. SAS URLs can be generated on Azure portal using blob's options, or SAS token from using `New-AzStorageBlobSASToken`.
> 

### Create or update Run Command on a machine as a different user using RunAsUser and RunAsPassword parameters

Create or update Run Command on a machine as a different user using `RunAsUser` and `RunAsPassword` parameters. For RunAs to work properly, contact the administrator the of machine and make sure user is added on the machine, user has access to resources accessed by the Run Command (directories, files, network etc.), and in case of Windows machine, 'Secondary Logon' service is running on the machine.

```powershell
New-AzMachineRunCommand -ResourceGroupName MyRG0 -MachineName MyMachine -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScript "id; echo HelloWorld" -RunAsUser myusername -RunAsPassword mypassword
```

### Create or update Run Command on a machine resource using SourceScriptUri (storage blob SAS URL)

Create or update Run Command on a Windows machine resource using a SAS URL of a storage blob that contains a PowerShell script.


```powershell
New-AzMachineRunCommand -ResourceGroupName MyRG0 -MachineName MyMachine -RunCommandName MyRunCommand -Location EastUS2EUAP -SourceScriptUri <SourceScriptUri>
```

> [!NOTE]
> SAS URL must provide read access to the blob. An expiry time of 24 hours is suggested for SAS URL. SAS URLs can be generated on Azure portal using blob options or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, the SAS URL format is: base blob URL + "?" + the SAS token from `New-AzStorageBlobSASToken`.


### Create or update Run Command on a machine using ScriptLocalPath (local script file)
Create or update Run Command on a machine using a local script file that is on the client machine where cmdlet is executed.

```powershell
New-AzMachineRunCommand -ResourceGroupName MyRG0 -VMName MyMachine -RunCommandName MyRunCommand -Location EastUS2EUAP -ScriptLocalPath "C:\MyScriptsDir\MyScript.ps1"
```

### Create or update Run Command on a machine instance using Parameter and ProtectedParameter parameters (Public and Protected Parameters to script)

Use ProtectedParameter to pass any sensitive inputs to script such as passwords, keys etc.

- Windows: Parameters and ProtectedParameters are passed to script as arguments are passed to script and run like this: `myscript.ps1 -publicParam1 publicParam1value -publicParam2 publicParam2value -secret1 secret1value -secret2 secret2value`

- Linux: Named Parameters and its values are set to environment config, which should be accessible within the .sh script. For Nameless arguments, pass an empty string to name input. Nameless arguments are passed to script and run like this: `myscript.sh publicParam1value publicParam2value secret1value secret2value`

### Delete RunCommand resource from the machine

Remove the RunCommand resource previously deployed on the machine. If the script execution is still in progress, execution will be terminated.

```powershell
Remove-AzConnetedMachineRunCommand -ResourceGroupName "myRG" -MachineName "myMachine" -RunCommandName "RunCommandName"
```



## Example scenarios

Suppose you have an Azure Arc-enabled server called “2012DatacenterServer1” in resource group “ContosoRG” with Subscription ID “aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa”. Consider a scenario where you need to provide remote access to an endpoint for Windows Server 2012 / R2 servers. Access to Extended Security Updates enabled by Azure Arc requires access to the endpoint `www.microsoft.com/pkiops/certs`. You need to remotely configure a firewall rule that allows access to this endpoint. Use Run Command in order to allow connectivity to this endpoint.

### Example 1: Endpoint access with Run Command

Start off by creating a Run Command script to provide endpoint access to the `www.microsoft.com/pkiops/certs` endpoint on your target Arc-enabled server using the PUT operation.

To directly provide the script in line, use the following operation:

```rest
PUT https://management.azure.com/subscriptions/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2023-10-03-preview
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

To instead link to the script file, you can use the Run Command operation’s ScriptURI option. For this it's assumed you have prepared a `newnetfirewallrule.ps1` file containing the in-line script and uploaded this script to blob storage.

```rest
PUT https://management.azure.com/subscriptions/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2023-10-03-preview
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

SAS URL must provide read access to the blob. An expiry time of 24 hours is suggested for SAS URL. SAS URLs can be generated on Azure portal using blobs options or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, the SAS URL format is: `base blob URL + "?"` + the SAS token from `New-AzStorageBlobSASToken`. 

Output and error blobs must be the AppendBlob type and their SAS URLs must provide read, append, create, write access to the blob. An expiration time of 24 hours is suggested for SAS URL. SAS URLs can be generated on Azure portal using blob's options, or SAS token from using `New-AzStorageBlobSASToken`.

### Example 2: Get Run Command details

To verify that you've correctly provisioned the Run Command, use the GET command to retrieve details on the provisioned Run Command:

```rest
GET https://management.azure.com/subscriptions/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2023-10-03-preview
```

### Example 3: Update the Run Command

Let’s suppose you want to open up access to an additional endpoint `*.waconazure.com` for connectivity to Windows Admin Center. You can update the existing Run Command with new parameters:


```rest
PATCH https://management.azure.com/subscriptions/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2023-10-03-preview
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


### Example 4: List Run Commands

Ahead of deleting the Run Command for Endpoint Access, make sure there are no other Run Commands for the Arc-enabled server. You can use the list command to get all of the Run Commands:

```rest
LIST https://management.azure.com/subscriptions/aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/
```

### Example 5: Delete a Run Command

If you no longer need the Run Command extension, you can delete it using the following command:

```rest
DELETE https://management.azure.com/subscriptions/ aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2023-10-03-preview
```

## Maintain VM security

While remote access enabled by the Run command lowers overhead for performing certain tasks on a VM, there are a few ways you can make sure the VM also stays secure.

- Limit access to the Run command in a subscription
- Allow or block certain Run commands locally 

### Limit access to Run Command using role-based access (RBAC)

You can use RBAC to control what roles in a subscription are able to execute commands and scripts with the Run command. The following table describes action you can take with the Run command, the permission needed to perform the action, and the RBAC role that grants the permission.

|Action  |Permission  | RBAC with permission |
|---------|---------|---------|
|- List Run commands - Show details of command|`Microsoft.HybridCompute/machines/runCommands/read`|Built-in [Reader](/azure/role-based-access-control/built-in-roles) role and higher|
|Run a command|`Microsoft.HybridCompute/machines/runCommands/write`|[Azure Connected Machine Resource Administrator](/azure/role-based-access-control/built-in-roles) role and higher|

To control access to the Run command functionality, use one of the [built-in roles](/azure/role-based-access-control/built-in-roles) or create a [custom role](/azure/role-based-access-control/custom-roles) that grants a Run command permission.

### Block run commands locally

You can control whether the Connected Machine agent allows access to the VM through Run commands by adding the Run command extension to an allowlist (inclusive) or a blocklist (exclusive). See [Extension allowlists and blocklists](security-extensions.md#allowlists-and-blocklists) to learn more.


#### Windows
The following example adds the Run command extension to a blocklist on a Windows VM.

`azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerwindows"`

#### Linux
The following example add the Run command extensions to an allowlist on a Linux VM.

`azcmagent config set extensions.allowlist "microsoft.cplat.core/runcommandhandlerlinux"`

## Disabling Run Command

To disable the Run Command on Azure Arc-enabled servers, open an administrative command prompt and run the following commands. These commands use the local agent configuration capabilities for the Connected Machine agent in the Extension blocklist.

**Windows**

`azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerwindows"`

**Linux**

`sudo azcmagent config set extensions.blocklist "microsoft.cplat.core/runcommandhandlerlinux"`
