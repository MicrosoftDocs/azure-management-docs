---
title: REST API Requests for the Run Command on Azure Arc-enabled Servers (Preview)
description: Learn how to use the REST API to remotely execute scripts and commands on Arc-enabled servers.
ms.date: 02/28/2025
ms.topic: concept-article
ms.custom: devx-track-azurecli, devx-track-azurepowershell
# Customer intent: As a cloud engineer managing Arc-enabled servers, I want to execute scripts remotely via a REST API, so that I can configure system settings without needing direct access to the servers.
---

# REST API requests for Run command on Azure Arc-enabled servers (Preview)

Using the [REST API](/rest/api/hybridcompute/machine-run-commands) for Run command on Azure Arc-enabled servers (Preview), you can remotely and securely execute scripts or commands on Arc-enabled virtual machines without connecting directly to them through Remote Desktop Protocol or SSH. 

This article provides supported REST API operations and an example scenario to help you understand how to use the REST API.

## Prerequisites

- The Connected Machine agent version on the Arc-enabled server must be 1.33 or higher.
- You need an access token to use the REST API. See [Getting Started with REST](/rest/api/azure/).

## Supported REST API operations

You can use the following REST API operations to execute Run commands on an Azure Arc-enabled server.

|Operation  |Description  |
|---------|---------|
|[Create or update](/rest/api/hybridcompute/machine-run-commands/create-or-update) |The operation to create or update a Run command. Runs the Run command. |
|[Delete](/rest/api/hybridcompute/machine-run-commands/delete) |The operation to delete a Run command. If it's running, delete also stops the Run command. |
|[Get](/rest/api/hybridcompute/machine-run-commands/get) |The operation to get a Run command. |
|[List](/rest/api/hybridcompute/machine-run-commands/list) |The operation to get all the Run commands of an Azure Arc-enabled server. |
|[Update](/rest/api/hybridcompute/machine-run-commands/update) |The operation to update the Run command. Stops the previous Run command. |
 
> [!NOTE]
> Output and error blobs are overwritten each time the Run command script executes.

## Example REST API scenario

The following requests walk you through using the Run command to remotely configure a firewall rule that allows access to the endpoint `www.microsoft.com/pkiops/certs`. The example scenario uses each of the available REST operations.

The code samples use the following information:

- **Subscription ID** - `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`
- **Arc-enabled server name** -  `2012DatacenterServer1`
- **Resource group** - `ContosoRG`
- **Server's operating system** - Windows Server 2012 / R2 servers


### Step 1: Create endpoint access with a Run Command

First, create a Run command script to provide access to the `www.microsoft.com/pkiops/certs` endpoint on your target Arc-enabled server using the PUT operation.

You can either provide the script inline or you can link to the script file. 

- To provide the script in line:

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

- To link to the script file: 
    >[!NOTE]
    >This sample assumes you prepared a file named `newnetfirewallrule.ps1`, containing the in-line script and that you uploaded the script to blob storage.

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
    
    - The `scriptUri` is a shared access signature (SAS) URI for the storage blob, and it must provide read access to the blob. An expiry time of 24 hours is suggested for the SAS URI. You can generate a SAS URI on Azure portal using blobs options or SAS token using `New-AzStorageBlobSASToken`. If generating SAS token using `New-AzStorageBlobSASToken`, the SAS URI format is: `base blob URL + "?"` + the SAS token from `New-AzStorageBlobSASToken`. 

    - Output and error blobs must be the AppendBlob type and their SAS URIs must provide read, append, create, and write access to the blob. An expiration time of 24 hours is suggested for SAS URI. You can generate SAS URIs on Azure portal using blob's options, or SAS token from using `New-AzStorageBlobSASToken`.

    To learn more about shared access signatures, see [Grant limited access to Azure Storage resources using shared access signatures (SAS)](/azure/storage/common/storage-sas-overview).

### Step 2: Verify Run Command details

Verify that you correctly provisioned the Run command using the GET operation.

```rest
GET https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

### Step 3: Update the Run Command

You can update the existing Run command with new parameters using the PATCH operation. The following example opens  access to another endpoint, `*.waconazure.com,` for connectivity to Windows Admin Center.


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

### Step 4: List Run Commands

Before you delete the Run command for endpoint access, make sure there are no other Run commands for the Arc-enabled server. You can use the LIST operation to get all of the Run commands on the Arc-enabled server:

```rest
LIST https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands?api-version=2024-11-10-preview
```

### Step 5: Delete a Run Command

When you no longer need the Run command extension, you can delete it using the following DELETE operation:

```rest
DELETE https://management.azure.com/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/ContosoRG/providers/Microsoft.HybridCompute/machines/2012DatacenterServer1/runCommands/EndpointAccessCommand?api-version=2024-11-10-preview
```

## Related content
- [Azure CLI requests for Run command on Azure Arc-enabled servers](run-command-cli.md)
- [PowerShell requests for Run command on Azure Arc-enabled servers](run-command-powershell.md)
- [What is Run command on Azure Arc-enabled servers?](run-command.md)
