---
title:  Terraform based SCVMM VM management
description: This article describes how to programmatically perform lifecycle operations on the SCVMM managed on-premises virtual machines using the Terraform templates.
ms.topic: how-to 
ms.date: 03/05/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
# Customer intent: "As a cloud infrastructure engineer, I want to manage SCVMM on-premises virtual machines using Terraform, so that I can automate lifecycle operations and enhance efficiency in resource management."
---

# Terraform based SCVMM VM management

This article describes how to programmatically perform lifecycle operations on the SCVMM managed on-premises virtual machines using the Terraform templates.

[Hashicorp Terraform](https://www.terraform.io/) is an open-source IaC (Infrastructure-as-Code) tool to configure and deploy cloud infrastructure. Terraform enables the management of your SCVMM based virtual infrastructure by using AzAPI Terraform providers. The complete set of Terraform templates available with Azure Arc enabled SCVMM can be accessed [here](/azure/templates/microsoft.scvmm/virtualmachineinstances?pivots=deployment-language-terraform).

The following scenarios are covered in this article:

- Create a new SCVMM-managed on-premises Virtual Machine from Azure
- Enable a SCVMM-managed on-premises Virtual Machine for management in Azure 
- Remove the Azure based management capability for a SCVMM-managed on-premises Virtual Machine

## Best practices

- **Use version control**: Keep your Terraform configuration files under version control (for example, Git) to track changes over time. 
- **State management**: Regularly back up your Terraform state files to avoid data loss. 

# [Scenario 1: Create a VM](#tab/scenario1)

### Prerequisites

Ensure to have the following prerequisites before you create a new virtual machine:

- An Azure subscription and resource group where you have Arc SCVMM VM Contributor role or a custom RBAC role with the required permissions to perform lifecycle operations on a Virtual Machine.
- An Arc-enabled SCVMM server with the Azure Arc resource bridge in a Running state.
- A workstation machine with Terraform installed.
- An Azure-enabled cloud resource on which you have Arc SCVMM Private Cloud Resource User role.
- An Azure-enabled virtual machine template resource on which you have Arc SCVMM Private Cloud Resource User role.
- An Azure-enabled virtual network resource on which you have Arc SCVMM Private Cloud Resource User role.

### Step 1: Define the VM variables in a *variables.tf* file

Here is a sample `variables.tf` file with placeholders.

```terraform
variable "subscription_id" {
  description = "The subscription ID for the Azure account."
  type        = string
}

variable "resource_group_name" {
  description = "The name of the resource group."
  type        = string
}

variable "location" {
  description = "The location/region where the resources will be created."
  type        = string
}

variable "machine_name" {
  description = "The name of the machine."
  type        = string
}

variable "inventory_item_id" {
  description = "The ID of the Inventory Item for the VM."
  type        = string
}

variable "vm_username" {
  description = "The admin username for the VM."
  type        = string
}

variable "vm_password" {
  description = "The admin password for the VM."
  type        = string
}

variable "cloud_id" {
    description = "The ID of the cloud."
    type        = string
}

variable "template_id" {
  description = "The ID of the template."
  type        = string
}

variable "virtual_network_id" {
  description = "The ID of the virtual network."
  type        = string
}

variable "availability_set_name" {
  description = "The name of the availability set."
  type        = string
  
}

variable "vmmserver_id" {
  description = "The ID of the SCVMM server."
  type        = string
}

variable "custom_location_id" {
  description = "The ID of the custom location."
  type        = string
}
```

### Step 2: Define the metadata and credentials in a *tfvars* file 

Here is a sample `createscvmmVM.tfvars` file with placeholders.

```terraform
subscription_id      = "your-subscription-id"
resource_group_name  = "your-resource-group"
location             = "eastus"
machine_name         = "vm-name"
vm_username          = "Administrator"
vm_password          = "your_vm_password"
vmmserver_id         = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.SCVMM/vmmServers/your-vmmserver-name"
availability_set_name = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.SCVMM/AvailabilitySets/your-availabilityset-name" #this parameter is optional
cloud_id             = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.SCVMM/Clouds/your-vmmcloud-name"
template_id          = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.SCVMM/VirtualMachineTemplates/your-template-name"
virtual_network_id   = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.SCVMM/VirtualNetworks/your-vmnetwork-name"
custom_location_id   = "/subscriptions/your-subscription-id/resourcegroups/your-resource-group/providers/Microsoft.extendedlocation/customlocations/your-customlocation-name"
```

### Step 3: Define the VM configuration in a *main.tf* file

Here is a sample `main.tf` file with placeholders.
```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
    azapi = {
      source  = "azure/azapi"
      version = ">= 1.0.0"
    }
  }
}

# Configure the AzureRM provider with the subscription ID
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Configure the AzAPI provider with the subscription ID
provider "azapi" {
  subscription_id = var.subscription_id
}

# Retrieve the resource group details
data "azurerm_resource_group" "example" {
  name = var.resource_group_name
}

# Create a Hybrid machine resource in Azure
resource "azapi_resource" "test_machine02" {
  schema_validation_enabled = false
  parent_id = data.azurerm_resource_group.example.id
  type = "Microsoft.HybridCompute/machines@2024-07-10"
  name = var.machine_name
  location = var.location
  body = {
      kind = "SCVMM"
      identity = {
        type = "SystemAssigned"
      }
  }
}

# Create a Virtual Machine instance using the Hybrid machine and Inventory Item ID
resource "azapi_resource" "test_inventory_vm001" {
  schema_validation_enabled = false
  type = "Microsoft.SCVMM/VirtualMachineInstances@2024-06-01"
  name = "default"
  parent_id = azapi_resource.test_machine01.id
  body = {
    properties = {
      infrastructureProfile = {
        templateId   = var.template_id
        cloudId      = var.cloud_id
        vmName       = var.machine_name
      }
    # Availability sets, OS profile and hardware profile are optional
      availabilitySets = [
      {
      name = var.availability_set_name
      }
      ]
    # Override VM template to customize VM creation 
      osProfile = {
        computerName    = "myVM"
        adminPassword   = var.vm_password
        domainName      = "cdmlab"
        domainUsername  = "cdmlabuser"
        domainPassword  = "your_domain_password"
        workgroup       = "myWorkgroup"
        runOnceCommands = "echo Hello; echo World"
      }
      hardwareProfile = {
        cpuCount = 4
        memoryMB = 3072
        limitCpuForMigration = "true"
      }
      networkProfile = {
        networkInterfaces = [
          {
            name            = "nic1"
            macAddress      = "00:0A:95:9D:68:16"
            virtualNetworkId = var.virtual_network_id
            ipv4AddressType = "Dynamic"
          }
        ]
      }
      storageProfile = {
        disks = [
          {
            name       = "disk1"
            diskId     = "disk-unique-id-1"
            diskSizeGB = 30
            bus        = 0
            lun        = 0
            busType    = "SCSI"
            vhdType    = "Dynamic"
          }
        ]
      }
    }
    extendedLocation = {
      type = "CustomLocation"
      name = var.custom_location_id
    }
  }
  depends_on = [azapi_resource.test_machine01]
}

# Install Arc agent on the VM
resource "azapi_resource" "guestAgent" {
  type      = "Microsoft.SCVMM/virtualMachineInstances/guestAgents@2024-06-01"
  parent_id = azapi_resource.test_inventory_vm001.id
  name      = "default"
  body = {
    properties = {
      credentials = {
        username = var.vm_username
        password = var.vm_password
      }
      provisioningAction = "install"
    }
  }
  schema_validation_enabled = false
  ignore_missing_property   = false
  depends_on = [azapi_resource.test_inventory_vm001]
}
```

### Step 4: Run Terraform commands

Use the -var-file flag to pass the *.tfvars* file during Terraform commands.

- Initialize Terraform (if not already initialized): `terraform init`
- Validate the configuration: `terraform validate -var-file="createscvmmVM.tfvars"`
- Plan the changes: `terraform plan -var-file="createscvmmVM.tfvars"`
- Apply the changes: `terraform apply -var-file="createscvmmVM.tfvars"`

Confirm the prompt by entering yes to apply the changes.

# [Scenario 2: Enable a VM for Azure management](#tab/scenario2)

### Prerequisites

Ensure to have the following prerequisites before you create a new virtual machine: 

- An Azure subscription and resource group where you have Arc SCVMM VM Contributor role or a custom RBAC role with the required permissions to perform lifecycle operations on a Virtual Machine.
- An Arc-enabled SCVMM server with the Azure Arc resource bridge in a Running state. 
- A workstation machine with Terraform installed. 

### Step 1: Define the VM variables in a *variables.tf* file 

Here is a sample `variables.tf` file with placeholders. 

```terraform
variable "subscription_id" {
  description = "The subscription ID for the Azure account."
  type        = string
}

variable "resource_group_name" {
  description = "The name of the resource group."
  type        = string
}

variable "location" {
  description = "The location/region where the resources will be created."
  type        = string
}

variable "machine_name" {
  description = "The name of the machine."
  type        = string
}

variable "inventory_item_id" {
  description = "The ID of the Inventory Item for the VM."
  type        = string
}

variable "vm_username" {
  description = "The admin username for the VM."
  type        = string
}

variable "vm_password" {
  description = "The admin password for the VM."
  type        = string
}

variable "vmmserver_id" {
  description = "The ID of the SCVMM server."
  type        = string
}

variable "custom_location_id" {
  description = "The ID of the custom location."
  type        = string
}
```

### Step 2: Define the metadata and credentials in a *tfvars* file 

Here is a sample `createscvmmVM.tfvars` file with placeholders. 

```terraform
subscription_id      = "your-subscription-id"
resource_group_name  = "your-resource-group"
location             = "eastus"
machine_name         = "vm-name"
vm_username          = "Administrator"
vm_password          = "your_vm_password"
inventory_item_id    = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.ScVmm/vmmServers/arcscvmm-777-2-vmmserver/InventoryItems/b3b8c107-9a7a-4ac2-b0aa-010ba7caba64"
vmmserver_id         = "/subscriptions/your-subscription-id/resourceGroups/your-resource-group/providers/Microsoft.ScVmm/vmmServers/arcscvmm-777-2-vmmserver"
custom_location_id   = "/subscriptions/your-subscription-id/resourcegroups/your-resource-group/providers/microsoft.extendedlocation/customlocations/arcscvmm-777-2-cl"
``` 

>[!NOTE]
>`vm_username` and `vm_password` are optional parameters. They are needed only to install the Azure Arc agent on the VM.

### Step 3: Define the VM configuration in a *main.tf* file 

Here is a sample `main.tf` file with placeholders. 

```terraform
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = ">= 3.0"
    }
    azapi = {
      source  = "azure/azapi"
      version = ">= 1.0.0"
    }
  }
}

# Configure the AzureRM provider with the subscription ID
provider "azurerm" {
  features {}
  subscription_id = var.subscription_id
}

# Configure the AzAPI provider with the subscription ID
provider "azapi" {
  subscription_id = var.subscription_id
}

# Retrieve the resource group details
data "azurerm_resource_group" "example" {
  name = var.resource_group_name
}

# Create a Hybrid Machine resource in Azure
resource "azapi_resource" "vm-name" {
  schema_validation_enabled = false
  parent_id = data.azurerm_resource_group.example.id
  type = "Microsoft.HybridCompute/machines@2024-07-10"
  name = var.machine_name
  location = var.location
  body = {
      kind = "SCVMM"
      identity = {
        type = "SystemAssigned"
      }
  }
}

# Enable a Virtual Machine instance using the Hybrid machine and Inventory Item ID
resource "azapi_resource" "test_inventory_vm" {
  schema_validation_enabled = false
  type = "Microsoft.SCVMM/VirtualMachineInstances@2024-06-01"
  name = "default"
  parent_id = azapi_resource.vm-name.id
  body = {
    properties = {
      infrastructureProfile = {
        inventoryItemId = var.inventory_item_id
        vmmServerId   = var.vmmserver_id
      }
    }
    extendedLocation = {
      type = "CustomLocation"
      name = var.custom_location_id
    }
  }
  depends_on = [azapi_resource.vm-name]
}

# Install Arc agent on the VM (optional step)
 resource "azapi_resource" "guestAgent" {
   type      = "Microsoft.SCVMM/virtualMachineInstances/guestAgents@2024-06-01"
   parent_id = azapi_resource.test_inventory_vm.id
   name      = "default"
   body = {
     properties = {
       credentials = {
         username = var.vm_username
         password = var.vm_password
       }
       provisioningAction = "install"
     }
   }
   schema_validation_enabled = false
   ignore_missing_property   = false
   depends_on = [azapi_resource.test_inventory_vm]
 }
```

### Step 4: Run Terraform commands 

Use the -var-file flag to pass the *.tfvars* file during Terraform commands. 

- Initialize Terraform (if not already initialized): `terraform init`
- Validate the configuration: `terraform validate -var-file="enablescvmmVM.tfvars"`
- Plan the changes: `terraform plan -var-file="enablescvmmVM.tfvars"`
- Apply the changes: `terraform apply -var-file="enablescvmmVM.tfvars"`

Confirm the prompt by entering yes to apply the changes.

# [Scenario 3: Remove from Azure](#tab/scenario3)

### Prerequisites 

Ensure to have the following prerequisites before you create a new virtual machine: 

- An Azure subscription and resource group where you have Arc SCVMM VM Contributor role or a custom RBAC role with the required permissions to perform lifecycle operations on a Virtual Machine.
- An Arc-enabled SCVMM server with the Azure Arc resource bridge in a Running state. 
- A workstation machine with Terraform installed. 
- The virtual machine that is to be removed from Azure based management must be enabled for management in Azure and must have an Azure resource. 

Following are the considerations before you remove the VM from Azure:

- This action removes the Azure Resource Manager representation of the machine. The machine needs to be enabled in Azure again to manage it from Azure. Before you proceed, ensure that the VM doesn't break any dependencies and take backups of any important data, if needed. 

- To remove a SCVMM-managed VM from Azure-based management, remove the VM definition (or comment out) from your Terraform configuration and run: 

   ```terraform
   terraform plan 
   ```

   Terraform shows that the VM is planned for removal from Azure because it no longer exists in the configuration. If you agree with the changes, run: 

   ```terraform
   terraform apply 
   ```

   This destroys the Azure representation of the VM if it’s no longer in the configuration.

---

## Next steps

- [Perform operations on SCVMM VMs from Azure](/azure/azure-arc/system-center-virtual-machine-manager/perform-vm-ops-on-scvmm-through-azure).
- Manage [Arc-enabled SCVMM VM extensions](/azure/azure-arc/servers/manage-vm-extensions).