---
title: Enable VM Extensions Using Azure Resource Manager Template
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments by using an Azure Resource Manager template.
ms.date: 05/12/2025
ms.topic: how-to
ms.custom: devx-track-arm-template
---

# Enable Azure VM extensions by using an ARM template

This article shows you how to use an Azure Resource Manager template (ARM template) to deploy Azure [virtual machine (VM) extensions](manage-vm-extensions.md) to Azure Arc-enabled servers.

To deploy extensions to Arc-enabled servers with an ARM template, you add extensions to the template and execute them with the template deployment. You can deploy the extensions on Linux or Windows connected machines by using Azure PowerShell. Each sample that follows includes a template file and a parameter file with sample values to provide to the template.

> [!NOTE]
> Although multiple extensions can be batched and processed together, they're installed serially. After installation of the first extension is complete, the next extension is installed.

## Deploy the Azure Monitor Agent VM extension

To deploy the Azure Monitor Agent, use one of the following samples to install the agent on either Linux or Windows.

### Template file for Linux

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
            "type": "Microsoft.Compute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2021-11-01",
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

### Template file for Windows

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
            "type": "Microsoft.Compute/machines/extensions",
            "location": "[parameters('location')]",
            "apiVersion": "2021-11-01",
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

### Parameter file

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

Save the template and parameter file, and edit the parameter file with the appropriate values for your deployment. You can then install the extension on all the connected machines within a resource group by using the following command. The command uses the `TemplateFile` parameter to specify the template and the `TemplateParameterFile` parameter to specify a file that contains parameters and parameter values. Replace the placeholders with the appropriate values for your deployment.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "<resource-group-name>" -TemplateFile "<template-filename.json>" -TemplateParameterFile "<parameter-filename.json>"
```

## Deploy the Custom Script Extension

To use the Custom Script Extension, run one of the following samples for Linux and Windows. If you're unfamiliar with the Custom Script Extension, see [Custom Script Extension for Linux](/azure/virtual-machines/extensions/custom-script-linux) or [Custom Script Extension for Windows](/azure/virtual-machines/extensions/custom-script-windows). There are few differing characteristics that you should understand when you're using this extension with hybrid machines:

* The list of supported operating systems with the Azure VM Custom Script Extension is not applicable to Azure Arc-enabled servers. See the [list of supported operating systems for Azure Arc-enabled servers](prerequisites.md#supported-operating-systems).

* Configuration details regarding Azure virtual machine scale sets or VMs created through the classic deployment model are not applicable.

* If your machines need to download a script externally and can communicate only through a proxy server, you need to [configure the Connected Machine agent](manage-agent.md#update-or-remove-proxy-settings) to set the proxy server's environmental variable.

The Custom Script Extension configuration specifies things like script location and the command to be run. This configuration is specified in the following templates.

### Template file for Linux

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
      "apiVersion": "2022-03-10",
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

### Template file for Windows

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

### Parameter file

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.1.2-preview",
  "parameters": {
    "basics": [
      {}
    ],
    "steps": [
      {
        "name": "customScriptExt",
        "label": "Add Custom Script Extension",
        "elements": [
          {
            "name": "fileUris",
            "type": "Microsoft.Common.FileUpload",
            "label": "Script files",
            "toolTip": "The script files that will be downloaded to the virtual machine.",
            "constraints": {
              "required": false
            },
            "options": {
              "multiple": true,
              "uploadMode": "url"
            },
            "visible": true
          },
          {
            "name": "commandToExecute",
            "type": "Microsoft.Common.TextBox",
            "label": "Command",
            "defaultValue": "sh script.sh",
            "toolTip": "The command to execute, for example: sh script.sh",
            "constraints": {
              "required": true
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "vmName": "[vmName()]",
      "location": "[location()]",
      "fileUris": "[steps('customScriptExt').fileUris]",
      "commandToExecute": "[steps('customScriptExt').commandToExecute]"
    }
  }
}
```

## Deploy the Dependency agent extension

To use the Azure Monitor Dependency agent extension, run one of the following samples for Linux and Windows. If you're unfamiliar with the Dependency agent, see [Overview of Azure Monitor agents](/azure/azure-monitor/vm/vminsights-dependency-agent-maintenance).

### Template file for Linux

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "vmName": {
      "type": "string",
      "metadata": {
        "description": "The name of existing Linux machine."
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
        "type": "DependencyAgentLinux",
        "enableAutomaticUpgrade": true
      }
    }
  ],
  "outputs": {
  }
}
```

### Template file for Windows

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

### Template deployment

Save the template file to disk. You can then deploy the extension to the connected machine by using the following command:

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "D:\Azure\Templates\DependencyAgent.json"
```

## Deploy the Azure Key Vault VM extension (preview)

The following JSON shows the schema for the Key Vault VM extension (preview). The extension does not require protected settings, because all its settings are considered public information. The extension requires a list of monitored certificates, the polling frequency, and the destination certificate store.

### Template file for Linux

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

### Template file for Windows

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
> Your observed certificate URLs should be of the form `https://myVaultName.vault.azure.net/secrets/myCertName`. The reason is that the `/secrets` path returns the full certificate, including the private key, whereas the `/certificates` path doesn't. You can find more information about certificates in [Azure Key Vault keys, secrets, and certificates overview](/azure/key-vault/general/about-keys-secrets-certificates).

### Template deployment

Save the template file to disk. You can then deploy the extension to the connected machine by using the following command.

> [!NOTE]
> The VM extension would require a system-assigned identity to be assigned to authenticate to Key Vault. See [Authenticate against Azure resources with Azure Arc-enabled servers](managed-identity-authentication.md) for Linux and Windows Azure Arc-enabled servers.

```powershell
New-AzResourceGroupDeployment -ResourceGroupName "ContosoEngineering" -TemplateFile "D:\Azure\Templates\KeyVaultExtension.json"
```

## Related content

* You can deploy, manage, and remove VM extensions by using [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or the [Azure CLI](manage-vm-extensions-cli.md).
* You can find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
