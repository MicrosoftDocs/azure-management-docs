---
title: Programmatically create Azure Dashboards
description: Use a dashboard in the Azure portal as a template to programmatically create Azure Dashboards. Includes JSON reference.
ms.topic: how-to
ms.custom:
ms.date: 02/02/2026
# Customer intent: "As an Azure user, I want to programmatically create and deploy Azure portal dashboards using JSON templates, so that I can manage and customize multiple dashboards efficiently."
---

# Programmatically create Azure dashboards

This article shows you how to programmatically create and publish Azure dashboards. Use this process to create a deployable template based on any dashboard.

## Overview

Shared dashboards in the [Azure portal](https://portal.azure.com) are [resources](/azure/azure-resource-manager/management/overview), just like virtual machines and storage accounts. You can manage resources programmatically by using [Azure REST API](/rest/api/?preserve-view=true&view=Azure), [Azure CLI](/cli/azure), and [Azure PowerShell](/powershell/azure/get-started-azureps). Each of these APIs and tools provides ways to create, list, retrieve, modify, and delete resources.

Whichever tools you use, to create a dashboard programmatically, you construct a [JSON representation](azure-portal-dashboards-structure.md) of your dashboard object. This object contains information about the tiles on the dashboard. It includes sizes, positions, resources, and customizations.

The most practical way to generate this JSON document is to use the Azure portal to create an initial dashboard with the tiles you want. After you create the dashboard, export the JSON, then use it as a template. You can modify the template further before you deploy it to your Azure environment.

## Fetch the JSON representation of a dashboard

First, download the JSON representation of an existing dashboard. From the dashboard that you want to start with, select **Export** and then select **Download**.

:::image type="content" source="media/azure-portal-dashboards-create-programmatically/download-command.png" alt-text="Screenshot of the command to export the JSON representation of a template in the Azure portal.":::

If you prefer, you can retrieve the representation and other information about the dashboard resource programmatically, such as by using [REST APIs](/rest/api/resources/Resources/Get).

## Create a dashboard template

Azure lets you orchestrate the deployment of multiple resources, such as dashboards. You create a deployment template that expresses the set of resources to deploy and the relationships between them. In this case, you create a template by modifying the JSON you downloaded. 

The JSON format of each deployed template resource is the same as if you were creating them individually by uploading an exported dashboard, except that the template language adds a few concepts like variables, parameters, basic functions, and more. This extended syntax is only supported in the context of a template deployment. For more information, see [Understand the structure and syntax of ARM templates](/azure/azure-resource-manager/templates/syntax).

You can preserve the structure and configuration of each tile, then parameterize the set of Azure resources that the tiles refer to, so they can be reused and adapted for different resources. To do so, use the template's parameter syntax to replace specific resource IDs with parameterized values.

For example, your template might include a hard-coded resource ID for a VM, such as:

```json
resourceId: "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/contoso/providers/Microsoft.Compute/virtualMachines/myVM1"
```

This resource ID can be replaced with a parameterized version that you can use with other VMs, such as:
```json
resourceId: "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
```

Declare required template metadata and the parameters at the top of the JSON template, like this:

```json

{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "virtualMachineName": {
            "type": "string"
        },
        "virtualMachineResourceGroup": {
            "type": "string"
        },
        "dashboardName": {
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        ... rest of template omitted ...
    ]
}
```

## Deploy the dashboard template

Once you configure your template, deploy it by using any of the following methods.

### Deploy the dashboard template in the Azure portal

1. Select the following image to sign in to Azure and open a template.

    :::image type="content" source="~/reusable-content/ce-skilling/azure/media/template-deployments/deploy-to-azure-button.svg" alt-text="Button to deploy the Resource Manager template to Azure." border="false" link="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fquickstarts%2Fmicrosoft.portal%2Fazure-portal-dashboard%2Fazuredeploy.json":::

1. Select or enter the appropriate project and instance details, and then select **Review + create**.

1. Select **Create**. You see a notification when the dashboard is deployed successfully.

### Deploy the dashboard template by using the Azure CLI

1. Run the [az portal dashboard create](/cli/azure/portal/dashboard#az-portal-dashboard-create) command to deploy the template:

   ```azurecli
   az portal dashboard create --resource-group myResourceGroup --name 'Simple VM Dashboard' \
      --input-path portal-dashboard-template-testvm.json --location centralus
   ```

1. Check that the dashboard was created successfully by running the [az portal dashboard show](/cli/azure/portal/dashboard#az-portal-dashboard-show) command:

   ```azurecli
   az portal dashboard show --resource-group myResourceGroup --name 'Simple VM Dashboard'
   ```

To see all the dashboards for the current subscription, use [az portal dashboard list](/cli/azure/portal/dashboard#az-portal-dashboard-list):

```azurecli
az portal dashboard list
```

To update a dashboard, use the [az portal dashboard update](/cli/azure/portal/dashboard#az-portal-dashboard-update) command:

```azurecli
az portal dashboard update --resource-group myResourceGroup --name 'Simple VM Dashboard' \
   --input-path portal-dashboard-template-testvm.json --location centralus
```

### Deploy the dashboard template by using Azure PowerShell

Use the [New-AzPortalDashboard](/powershell/module/az.portal/new-azportaldashboard) cmdlet, which is part of the [Az.Portal module](/powershell/module/az.portal/), to deploy the template directly from PowerShell:

```azurepowershell
$DashboardParams = @{
  DashboardPath = $myPortalDashboardTemplatePath
  ResourceGroupName = $resourceGroupName
  DashboardName = $dashboardName
}
New-AzPortalDashboard @DashboardParams
```

Check that the dashboard was created successfully:

```azurepowershell
Get-AzPortalDashboard -Name $dashboardName -ResourceGroupName $resourceGroupName
```

## Example dashboard

Consider the following example dashboard:

:::image type="content" source="media/azure-portal-dashboards-create-programmatically/sample-dashboard.png" alt-text="Screenshot of a sample dashboard in the Azure portal.":::

The following sections show two JSON representations of this example dashboard. The first is the version exported from the portal that's bound to a specific VM resource. The second is the template version that can be programmatically bound to any VM and deployed using Azure Resource Manager.

### Example JSON representation exported from dashboard

This example template is similar to what you see when you export a dashboard that looks like the example at the beginning of this article. The hard-coded resource identifiers show that this dashboard points at a specific Azure VM.

```json
{
  "properties": {
    "lenses": [
      {
        "order": 0,
        "parts": [
          {
            "position": {
              "x": 0,
              "y": 0,
              "colSpan": 3,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [],
              "type": "Extension/HubsExtension/PartType/MarkdownPart",
              "settings": {
                "content": {
                  "settings": {
                    "content": "## Azure Virtual Machines Overview\r\nNew team members should watch this video to get familiar with Azure Virtual Machines.",
                    "markdownUri": null
                  }
                }
              }
            }
          },
          {
            "position": {
              "x": 3,
              "y": 0,
              "colSpan": 8,
              "rowSpan": 4
            },
            "metadata": {
              "inputs": [],
              "type": "Extension/HubsExtension/PartType/MarkdownPart",
              "settings": {
                "content": {
                  "settings": {
                    "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Create a Linux virtual machine](https://docs.microsoft.com/azure/virtual-machines/linux/quick-create-portal)\r\n1. [Create a Windows virtual machine](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal)\r\n1. [Create a virtual machine scale set](https://docs.microsoft.com/azure/virtual-machine-scale-sets/quick-create-portal)",
                    "title": "Test VM Dashboard",
                    "subtitle": "Contoso",
                    "markdownUri": null
                  }
                }
              }
            }
          },
          {
            "position": {
              "x": 0,
              "y": 2,
              "colSpan": 3,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [],
              "type": "Extension/HubsExtension/PartType/VideoPart",
              "settings": {
                "content": {
                  "settings": {
                    "src": "https://www.youtube.com/watch?v=rOiSRkxtTeU",
                    "autoplay": false
                  }
                }
              }
            }
          },
          {
            "position": {
              "x": 0,
              "y": 4,
              "colSpan": 11,
              "rowSpan": 3
            },
            "metadata": {
              "inputs": [
                {
                  "name": "queryInputs",
                  "value": {
                    "timespan": {
                      "duration": "PT1H"
                    },
                    "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine",
                    "chartType": 0,
                    "metrics": [
                      {
                        "name": "Percentage CPU",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      }
                    ]
                  }
                }
              ],
              "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
            }
          },
          {
            "position": {
              "x": 0,
              "y": 7,
              "colSpan": 3,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [
                {
                  "name": "queryInputs",
                  "value": {
                    "timespan": {
                      "duration": "PT1H"
                    },
                    "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine",
                    "chartType": 0,
                    "metrics": [
                      {
                        "name": "Disk Read Operations/Sec",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      },
                      {
                        "name": "Disk Write Operations/Sec",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      }
                    ]
                  }
                }
              ],
              "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
            }
          },
          {
            "position": {
              "x": 3,
              "y": 7,
              "colSpan": 3,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [
                {
                  "name": "queryInputs",
                  "value": {
                    "timespan": {
                      "duration": "PT1H"
                    },
                    "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine",
                    "chartType": 0,
                    "metrics": [
                      {
                        "name": "Disk Read Bytes",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      },
                      {
                        "name": "Disk Write Bytes",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      }
                    ]
                  }
                }
              ],
              "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
            }
          },
          {
            "position": {
              "x": 6,
              "y": 7,
              "colSpan": 3,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [
                {
                  "name": "queryInputs",
                  "value": {
                    "timespan": {
                      "duration": "PT1H"
                    },
                    "id": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine",
                    "chartType": 0,
                    "metrics": [
                      {
                        "name": "Network In Total",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      },
                      {
                        "name": "Network Out Total",
                        "resourceId": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                      }
                    ]
                  }
                }
              ],
              "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
            }
          },
          {
            "position": {
              "x": 9,
              "y": 7,
              "colSpan": 2,
              "rowSpan": 2
            },
            "metadata": {
              "inputs": [
                {
                  "name": "id",
                  "value": "/subscriptions/aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e/resourceGroups/SimpleWinVMResourceGroup/providers/Microsoft.Compute/virtualMachines/myVirtualMachine"
                }
              ],
              "type": "Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart",
              "asset": {
                "idInputName": "id",
                "type": "VirtualMachine"
              }
            }
          }
        ]
      }
    ],
    "metadata": {
      "model": {}
    }
  },
  "name": "Simple VM Dashboard",
  "type": "Microsoft.Portal/dashboards",
  "location": "INSERT LOCATION",
  "tags": {
    "hidden-title": "Simple VM Dashboard"
  },
  "apiVersion": "2022-12-01-preview"
}
```

### Example parameterized dashboard template

This example shows the template version of the previous dashboard. You can programmatically bind it to any virtual machine and deploy it by using Azure Resource Manager.

The templatized version of the example dashboard defines three parameters called `virtualMachineName`, `virtualMachineResourceGroup`, and `dashboardName`. These parameters let you use the same dashboard template with a different Azure virtual machine every time you deploy. You can programmatically configure and deploy it to point to any Azure virtual machine. To test this feature, copy the following template and paste it into the [Azure portal template deployment page](https://portal.azure.com/#create/Microsoft.Template).

This example deploys one dashboard by itself, but the template language lets you deploy multiple resources and bundle one or more dashboards together.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "virtualMachineName": {
      "type": "string",
      "metadata": {
        "description": "Name of the existing virtual machine to show in the dashboard"
      }
    },
    "virtualMachineResourceGroup": {
      "type": "string",
      "metadata": {
        "description": "Name of the resource group that contains the virtual machine"
      }
    },
    "dashboardName": {
      "type": "string",
      "defaultValue": "[guid(parameters('virtualMachineName'), parameters('virtualMachineResourceGroup'))]",
      "metadata": {
        "Description": "Resource name that Azure portal uses for the dashboard"
      }
    },
    "dashboardDisplayName": {
      "type": "string",
      "defaultValue": "Simple VM Dashboard",
      "metadata": {
        "description": "Name of the dashboard to display in Azure portal"
      }
    },
    "location": {
      "type": "string",
      "defaultValue": "[resourceGroup().location]"
    }
  },
  "resources": [
    {
      "type": "Microsoft.Portal/dashboards",
      "apiVersion": "2020-09-01-preview",
      "name": "[parameters('dashboardName')]",
      "location": "[parameters('location')]",
      "tags": {
        "hidden-title": "[parameters('dashboardDisplayName')]"
      },
      "properties": {
        "lenses": [
          {
            "order": 0,
            "parts": [
              {
                "position": {
                  "x": 0,
                  "y": 0,
                  "rowSpan": 2,
                  "colSpan": 3
                },
                "metadata": {
                  "inputs": [],
                  "type": "Extension/HubsExtension/PartType/MarkdownPart",
                  "settings": {
                    "content": {
                      "settings": {
                        "content": "## Azure Virtual Machines Overview\r\nNew team members should watch this video to get familiar with Azure Virtual Machines."
                      }
                    }
                  }
                }
              },
              {
                "position": {
                  "x": 3,
                  "y": 0,
                  "rowSpan": 4,
                  "colSpan": 8
                },
                "metadata": {
                  "inputs": [],
                  "type": "Extension/HubsExtension/PartType/MarkdownPart",
                  "settings": {
                    "content": {
                      "settings": {
                        "content": "This is the team dashboard for the test VM we use on our team. Here are some useful links:\r\n\r\n1. [Create a Linux virtual machine](https://docs.microsoft.com/azure/virtual-machines/linux/quick-create-portal)\r\n1. [Create a Windows virtual machine](https://docs.microsoft.com/azure/virtual-machines/windows/quick-create-portal)\r\n1. [Create a virtual machine scale set](https://docs.microsoft.com/azure/virtual-machine-scale-sets/quick-create-portal)",
                        "title": "Test VM Dashboard",
                        "subtitle": "Contoso"
                      }
                    }
                  }
                }
              },
              {
                "position": {
                  "x": 0,
                  "y": 2,
                  "rowSpan": 2,
                  "colSpan": 3
                },
                "metadata": {
                  "inputs": [],
                  "type": "Extension/HubsExtension/PartType/VideoPart",
                  "settings": {
                    "content": {
                      "settings": {
                        "src": "https://www.youtube.com/watch?v=rOiSRkxtTeU",
                        "autoplay": false
                      }
                    }
                  }
                }
              },
              {
                "position": {
                  "x": 0,
                  "y": 4,
                  "rowSpan": 3,
                  "colSpan": 11
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "queryInputs",
                      "value": {
                        "timespan": {
                          "duration": "PT1H"
                        },
                        "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                        "chartType": 0,
                        "metrics": [
                          {
                            "name": "Percentage CPU",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          }
                        ]
                      }
                    }
                  ],
                  "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
                }
              },
              {
                "position": {
                  "x": 0,
                  "y": 7,
                  "rowSpan": 2,
                  "colSpan": 3
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "queryInputs",
                      "value": {
                        "timespan": {
                          "duration": "PT1H"
                        },
                        "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                        "chartType": 0,
                        "metrics": [
                          {
                            "name": "Disk Read Operations/Sec",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          },
                          {
                            "name": "Disk Write Operations/Sec",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          }
                        ]
                      }
                    }
                  ],
                  "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
                }
              },
              {
                "position": {
                  "x": 3,
                  "y": 7,
                  "rowSpan": 2,
                  "colSpan": 3
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "queryInputs",
                      "value": {
                        "timespan": {
                          "duration": "PT1H"
                        },
                        "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                        "chartType": 0,
                        "metrics": [
                          {
                            "name": "Disk Read Bytes",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          },
                          {
                            "name": "Disk Write Bytes",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          }
                        ]
                      }
                    }
                  ],
                  "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
                }
              },
              {
                "position": {
                  "x": 6,
                  "y": 7,
                  "rowSpan": 2,
                  "colSpan": 3
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "queryInputs",
                      "value": {
                        "timespan": {
                          "duration": "PT1H"
                        },
                        "id": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]",
                        "chartType": 0,
                        "metrics": [
                          {
                            "name": "Network In Total",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          },
                          {
                            "name": "Network Out Total",
                            "resourceId": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                          }
                        ]
                      }
                    }
                  ],
                  "type": "Extension/Microsoft_Azure_Monitoring/PartType/MetricsChartPart"
                }
              },
              {
                "position": {
                  "x": 9,
                  "y": 7,
                  "rowSpan": 2,
                  "colSpan": 2
                },
                "metadata": {
                  "inputs": [
                    {
                      "name": "id",
                      "value": "[resourceId(parameters('virtualMachineResourceGroup'), 'Microsoft.Compute/virtualMachines', parameters('virtualMachineName'))]"
                    }
                  ],
                  "type": "Extension/Microsoft_Azure_Compute/PartType/VirtualMachinePart",
                  "asset": {
                    "idInputName": "id",
                    "type": "VirtualMachine"
                  }
                }
              }
            ]
          }
        ]
      }
    }
  ]
}
```

## Next steps

- Learn more about the [structure of Azure dashboards](azure-portal-dashboards-structure.md).
- Learn how to [use markdown tiles on Azure dashboards to show custom content](azure-portal-markdown-tile.md).
- Learn how to [manage access for shared dashboards](azure-portal-dashboard-share-access.md).
