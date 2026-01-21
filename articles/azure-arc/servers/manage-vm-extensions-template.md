---
title: Enable VM Extensions Using Azure Resource Manager Template
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments by using an Azure Resource Manager template.
ms.date: 06/19/2025
ms.topic: how-to
ms.custom:
  - devx-track-arm-template
  - build-2025
# Customer intent: As a cloud administrator, I want to deploy virtual machine extensions to Azure Arc-enabled servers using an ARM template, so that I can efficiently manage and enhance the capabilities of my hybrid cloud infrastructure.
---

# Enable Azure VM extensions by using an ARM template

This article shows you how to use an Azure Resource Manager template (ARM template) to deploy [virtual machine (VM) extensions](manage-vm-extensions.md) to Azure Arc-enabled servers (Arc-enabled machines).

To deploy extensions to Arc-enabled servers with an ARM template, you add extensions to the template and execute them with the template deployment. You can deploy the extensions on Linux or Windows machines connected to Azure Arc by using Azure PowerShell.

This article shows how to deploy several different VM extensions to an Arc-enabled servers by using a template file, along with a separate parameter file for some extensions. Replace the example values in the samples with your own values before deploying.

## Deployment commands

These sample PowerShell commands install an extension on all the connected Arc-enabled servers within a resource group, based on the information in your ARM template. The command uses the `TemplateFile` parameter to specify the template. If a parameters file is required, the `TemplateParameterFile` parameter is included to specify a file that contains parameters and parameter values. Replace the placeholders with the appropriate values for your deployment.

To deploy an ARM template and parameter file, use the following command, replacing the example values with your own:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```

For example:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "D:\Azure\Templates\AzureMonitorAgent.json" -TemplateParameterFile "D:\Azure\Templates\AzureMonitorAgentParams.json"
```

To deploy an ARM template without a parameter file, use the following command, replacing the example values with your own:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>"
```

For example:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "D:\Azure\Templates\DependencyAgent.json"
```

## Deploy the Azure Monitor Agent VM extension

To deploy the [Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-overview) to an Arc-enabled server, use one of the following sample templates to install the agent on either Linux or Windows.

> **Resource type**
>
> All Arc-enabled server VM extensions use the resource type `Microsoft.HybridCompute/machines/extensions` (mapped from the Azure VM extension resource type).

### Azure Monitor Agent template file for Linux (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `workspaceId` (string, example: Log Analytics workspace ID)
- `workspaceKey` (string, example: Log Analytics primary key)

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "workspaceId": {
            "type": "string"
        },
        "workspaceKey": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/AzureMonitorLinuxAgent')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2025-06-01",
            "properties": {
                "publisher": "Microsoft.Azure.Monitor",
                "type": "AzureMonitorLinuxAgent",
                "enableAutomaticUpgrade": true,
                "settings": {
                    "workspaceId": "[parameters('workspaceId')]"
                },
                "protectedSettings": {
                    "workspaceKey": "[parameters('workspaceKey')]"
                }
            }
        }
    ]
}
```

### Azure Monitor Agent template file for Windows (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `workspaceId` (string, example: Log Analytics workspace ID)
- `workspaceKey` (string, example: Log Analytics primary key)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "workspaceId": {
            "type": "string"
        },
        "workspaceKey": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/AzureMonitorWindowsAgent')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2025-06-01",
            "properties": {
                "publisher": "Microsoft.Azure.Monitor",
                "type": "AzureMonitorWindowsAgent",
                "autoUpgradeMinorVersion": true,
                "enableAutomaticUpgrade": true,
                "settings": {
                    "workspaceId": "[parameters('workspaceId')]"
                },
                "protectedSettings": {
                    "workspaceKey": "[parameters('workspaceKey')]"
                }
            }
        }
    ]
}
```

### Azure Monitor Agent parameter file (Arc-enabled server)

This parameter file can be used for both Linux and Windows.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "value": "<vmName>"
        },
        "location": {
            "value": "<region>"
        },
        "workspaceId": {
            "value": "<MyWorkspaceID>"
        },
        "workspaceKey": {
            "value": "<MyWorkspaceKey>"
        }
    }
}
```

Save the template and parameter file, and edit the parameter file with the appropriate values for your deployment.

### Deploy Azure Monitor Agent

1. **Prerequisites**: Arc-enabled server is connected and a Log Analytics workspace exists.
2. **Deploy**:
   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "AzureMonitorAgent.json" -TemplateParameterFile "AzureMonitorAgentParams.json"
   ```
3. **Verify**: In the Azure portal, confirm the extension status is **Succeeded** on the Arc-enabled server.

## Deploy the Custom Script Extension

To use the Custom Script Extension on an Arc-enabled server, deploy one of the following sample templates for Linux or Windows. For more information, see [Custom Script Extension for Linux](/azure/virtual-machines/extensions/custom-script-linux) or [Custom Script Extension for Windows](/azure/virtual-machines/extensions/custom-script-windows).

### Custom script extension template file for Linux (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `fileUris` (array, example: script URLs)
- `commandToExecute` (securestring, example: `sh script.sh`)

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    },
    "location": {
      "type": "string"
    },
    "fileUris": {
      "type": "array"
    },
    "commandToExecute": {
      "type": "securestring"
    }
  },
  "resources": [
    {
      "name": "[concat(parameters('vmName'),'/CustomScript')]",
      "type": "Microsoft.HybridCompute/machines/extensions",
      "location": "[parameters('location')]",
      "apiVersion": "2025-06-01",
      "properties": {
        "publisher": "Microsoft.Azure.Extensions",
        "type": "CustomScript",
        "autoUpgradeMinorVersion": true,
        "settings": {},
        "protectedSettings": {
          "commandToExecute": "[parameters('commandToExecute')]",
          "fileUris": "[parameters('fileUris')]"
        }
      }
    }
  ]
}
```

### Custom script template file for Windows (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `fileUris` (string, example: script URL)
- `arguments` (securestring, optional, example: `-Param1 Value1`)

```json
{
    "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "fileUris": {
            "type": "string"
        },
        "arguments": {
            "type": "securestring",
            "defaultValue": " "
        }
    },
    "variables": {
        "UriFileNamePieces": "[split(parameters('fileUris'), '/')]",
        "firstFileNameString": "[variables('UriFileNamePieces')[sub(length(variables('UriFileNamePieces')), 1)]]",
        "firstFileNameBreakString": "[split(variables('firstFileNameString'), '?')]",
        "firstFileName": "[variables('firstFileNameBreakString')[0]]"
    },
    "resources": [
        {
            "name": "[concat(parameters('vmName'),'/CustomScriptExtension')]",
            "type": "Microsoft.HybridCompute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2022-03-10",
            "properties": {
                "publisher": "Microsoft.Compute",
                "type": "CustomScriptExtension",
                "autoUpgradeMinorVersion": true,
                "settings": {
                    "fileUris": "[split(parameters('fileUris'), ' ')]"
                },
                "protectedSettings": {
                    "commandToExecute": "[concat ('powershell -ExecutionPolicy Unrestricted -File ', variables('firstFileName'), ' ', parameters('arguments'))]"
                }
            }
        }
    ]
}
```

### Deploy Custom Script Extension

1. **Prerequisites**: Script files accessible from the Arc-enabled server.
2. **Deploy**:
   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "CustomScript.json"
   ```

## Deploy the Dependency Agent extension

Use the following templates to deploy the Dependency Agent to an Arc-enabled Linux or Windows machine.

### Dependency Agent template file for Linux (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string"
    }
  },
  "resources": [
    {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "2025-06-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentLinux",
        "enableAutomaticUpgrade": true
      }
    }
  ]
}
```

### Dependency Agent template file for Windows (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Windows machine."
      }
    }
  },
  "resources": [
    {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/DAExtension')]",
      "apiVersion": "2022-03-10",
      "location": "[resourceGroup().location]",
      "dependsOn": [
      ],
      "properties": {
        "publisher": "Microsoft.Azure.Monitoring.DependencyAgent",
        "type": "DependencyAgentWindows",
        "enableAutomaticUpgrade": true
      }
    }
  ],
  "outputs": {
  }
}
```

### Deploy Dependency Agent

1. **Prerequisites**: Azure Monitor enabled for the Arc-enabled server.
2. **Deploy**:
   ```powershell
   New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "DependencyAgent.json"
   ```
3. **Verify**: In the Azure portal, confirm the extension status is **Succeeded** on the Arc-enabled server.

## Deploy the Azure Key Vault extension

The following JSON shows the schema for the Azure Key Vault extension. This extension doesn't require protected settings, because all its settings are considered public information. The extension requires a list of monitored certificates, the polling frequency, and the destination certificate store.

### Azure Key Vault template file for Linux (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `autoUpgradeMinorVersion` (bool, example: `true`)
- `pollingIntervalInS` (int, example: `3600`)
- `certificateStoreName` (string, ignored on Linux)
- `certificateStoreLocation` (string, example: `/var/lib/waagent/Microsoft.Azure.KeyVault`)
- `observedCertificates` (string, example: Key Vault certificate URI)
- `msiEndpoint` (string, example: `http://localhost:40342/metadata/identity`)
- `msiClientId` (string, example: managed identity client ID)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "autoUpgradeMinorVersion":{
            "type": "bool"
        },
        "pollingIntervalInS":{
          "type": "int"
        },
        "certificateStoreName":{
          "type": "string"
        },
        "certificateStoreLocation":{
          "type": "string"
        },
        "observedCertificates":{
          "type": "string"
        },
        "msiEndpoint":{
          "type": "string"
        },
        "msiClientId":{
          "type": "string"
        }
},
"resources": [
   {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/KVVMExtensionForLinux')]",
      "apiVersion": "2022-03-10",
      "location": "[parameters('location')]",
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForLinux",
      "enableAutomaticUpgrade": true,
      "settings": {
          "secretsManagementSettings": {
          "pollingIntervalInS": <polling interval in seconds, e.g. "3600">,
          "certificateStoreName": <ignored on linux>,
          "certificateStoreLocation": <disk path where certificate is stored, default: "/var/lib/waagent/Microsoft.Azure.KeyVault">,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net/secrets/mycertificate"
          },
          "authenticationSettings": {
                "msiEndpoint":  "http://localhost:40342/metadata/identity"
        }
      }
    }
  }
 ]
}
```

### Azure Key Vault template file for Windows (Arc-enabled server)

**Parameters**

- `vmName` (string, example: `myArcMachine`)
- `location` (string, example: `eastus`)
- `autoUpgradeMinorVersion` (bool, example: `true`)
- `pollingIntervalInS` (int, example: `3600`)
- `certificateStoreName` (string, example: `MY`)
- `linkOnRenewal` (bool, example: `false`)
- `certificateStoreLocation` (string, example: `LocalMachine`)
- `requireInitialSync` (bool, example: `true`)
- `observedCertificates` (string, example: Key Vault certificate URI)
- `msiEndpoint` (string, example: `http://localhost:40342/metadata/identity`)
- `msiClientId` (string, example: managed identity client ID)

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "vmName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "autoUpgradeMinorVersion":{
            "type": "bool"
        },
        "pollingIntervalInS":{
          "type": "int"
        },
        "certificateStoreName":{
          "type": "string"
        },
        "linkOnRenewal":{
          "type": "bool"
        },
        "certificateStoreLocation":{
          "type": "string"
        },
        "requireInitialSync":{
          "type": "bool"
        },
        "observedCertificates":{
          "type": "string"
        },
        "msiEndpoint":{
          "type": "string"
        },
        "msiClientId":{
          "type": "string"
        }
},
"resources": [
   {
      "type": "Microsoft.HybridCompute/machines/extensions",
      "name": "[concat(parameters('vmName'),'/KVVMExtensionForWindows')]",
      "apiVersion": "2022-03-10",
      "location": "[parameters('location')]",
      "properties": {
      "publisher": "Microsoft.Azure.KeyVault",
      "type": "KeyVaultForWindows",
      "enableAutomaticUpgrade": true,
      "settings": {
        "secretsManagementSettings": {
          "pollingIntervalInS": "3600",
          "certificateStoreName": <certificate store name, e.g.: "MY">,
          "linkOnRenewal": <Only Windows. This feature ensures s-channel binding when certificate renews, without necessitating a re-deployment.  e.g.: false>,
          "certificateStoreLocation": <certificate store location, currently it works locally only e.g.: "LocalMachine">,
          "requireInitialSync": <initial synchronization of certificates e.g.: true>,
          "observedCertificates": <list of KeyVault URIs representing monitored certificates, e.g.: "https://myvault.vault.azure.net"
        },
        "authenticationSettings": {
                "msiEndpoint": "http://localhost:40342/metadata/identity"
        }
      }
    }
  }
 ]
}
```

> [!NOTE]
> Your observed certificate URLs should be of the form `https://myVaultName.vault.azure.net/secrets/myCertName`. The reason is that the `/secrets` path returns the full certificate, including the private key, whereas the `/certificates` path doesn't. For more information about certificates, see [Azure Key Vault keys, secrets, and certificates overview](/azure/key-vault/general/about-keys-secrets-certificates).

Save the template and edit as needed for your environment. Then install the Azure Key Vault extension to your connected machines by running the [PowerShell deployment command](#deployment-commands) found earlier in this article.

> [!TIP]
> The Azure Key Vault extension requires a system-assigned identity to be assigned to authenticate to Key Vault. For more information, see [Authenticate against Azure resources with Azure Arc-enabled servers](managed-identity-authentication.md).

## Related content

- Read more about [VM extensions supported by Azure Arc-enabled servers](manage-vm-extensions.md).
- Learn how to deploy, manage, and remove VM extensions by using [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or the [Azure CLI](manage-vm-extensions-cli.md).
- Explore troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).

**Agent feedback applied**

- Extracted parameter lists for each template.
- Added compact numbered deployment procedures for each extension.
- Standardized resource type to `Microsoft.HybridCompute/machines/extensions`.
- Clarified Arc-enabled machine scope versus Azure VM usage.
