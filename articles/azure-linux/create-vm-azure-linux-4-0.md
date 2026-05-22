---
title: Create an Azure Virtual Machine (VM) using Azure Linux 4.0
description: This article shows you how to create an Azure Linux 4.0 virtual machine (VM) on Azure using the Azure portal, Azure CLI, an ARM template, or Terraform.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: how-to
ms.date: 04/27/2026
---

# Create an Azure virtual machine (VM) using Azure Linux 4.0

This article shows you how to create an Azure Linux 4.0 virtual machine (VM) on Azure using the Azure portal, the Azure CLI, an ARM template, or Terraform.

## Prerequisites

- An Azure subscription. If you don't have one, [create a free account](https://azure.microsoft.com/free/).
- For CLI-based methods:
  - Install the [Azure CLI](/cli/azure/install-azure-cli).
  - Sign in using the [`az login`](/cli/azure/authenticate-azure-cli) command.
- For Terraform:
  - [Terraform](https://www.terraform.io/) and the [Azure provider](/azure/developer/terraform/get-started-cloud-shell-bash?tabs=bash) configured for your subscription.

## Create an Azure Linux 4.0 VM

### [Azure portal](#tab/azure-portal)

For a guided, UI-driven walkthrough, go to the [Launch an Azure Linux VM](./create-vm-azure-linux-4-0.md) quickstart.

### [Azure CLI](#tab/azure-cli)

1. Create an Azure resource group using the [`az group create`](/cli/azure/group#az_group_create) command. Replace the placeholder values with your own values.

    ```azurecli-interactive
    az group create --name <resource-group-name> --location <location>
    ```

2. Create an Azure VM using an Azure Linux 4.0 Marketplace image using the [`az vm create`](/cli/azure/vm#az_vm_create) command. Replace the placeholder values with your own values.

    ```azurecli-interactive
    az vm create \
      --resource-group <resource-group-name> \
      --name <vm-name> \
      --image microsoftazurelinux:azurelinux-4:4:latest \
      --admin-username azureuser \
      --generate-ssh-keys
    ```

### [ARM template](#tab/arm-template)

Deploy an Azure Linux 4.0 VM with an ARM template by setting the `storageProfile` of your `Microsoft.Compute/virtualMachines` resource to reference an Azure Linux 4.0 Marketplace image. For example:

```json
"storageProfile": {
  "imageReference": {
    "publisher": "microsoftazurelinux",
    "offer": "azurelinux-4",
    "sku": "4",
    "version": "latest"
  },
  "osDisk": {
    "createOption": "fromImage"
  }
}
```

For a complete ARM Template example and step-by-step deployment instructions, see the [Create and deploy your first ARM template](/azure/azure-resource-manager/templates/template-tutorial-create-first-template?tabs=azure-powershell) tutorial.

### [Terraform](#tab/terraform)

Deploy an Azure Linux 4.0 VM with Terraform by setting the `source_image_reference` to reference an Azure Linux 4.0 Marketplace image. For example:

```hcl
  source_image_reference {
    publisher = "microsoftazurelinux"
    offer     = "azurelinux-4"
    sku       = "4"
    version   = "latest"
  }
```

For a complete `main.tf` example and step-by-step deployment instructions, see the [Create a Linux VM with Terraform](/azure/developer/terraform/create-linux-virtual-machine-with-infrastructure) quickstart.

---

## Connect to your Azure Linux 4.0 VM

Connect to your Azure Linux VM over SSH using the guidance in [Connect to a Linux VM in Azure](/azure/virtual-machines/linux-vm-connect).

## Related content

- [Customize images with Image Customizer](./customize-images.md)
- [Package management on Azure Linux](./package-management-overview.md)
