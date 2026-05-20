---
title: "Quickstart: Create an Azure Linux Virtual Machine in the Azure portal"
description: Learn how to quickly create an Azure Linux 4.0 virtual machine in the Azure portal and connect to it over SSH.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.topic: quickstart
ms.date: 04/26/2026
---

# Quickstart: Create an Azure Linux virtual machine (VM) in the Azure portal

Azure Linux for Virtual Machines provides a Microsoft-supported, Azure-optimized Linux distribution for scalable web apps, infrastructure services, and AI workloads. It integrates natively with Azure services, extensions, and tooling, and includes Microsoft-managed security response and lifecycle support. For more information, see [Azure Linux for Virtual Machines](./azure-linux-vm-vmss-overview.md).

In this quickstart, you learn how to:

> [!div class="checklist"]
>
> - Create an Azure Linux 4.0 VM in the Azure portal.
> - Configure SSH access and inbound port rules.
> - Connect to the VM over SSH.
> - Clean up resources.

[!INCLUDE [azure linux 4.0 preview](./includes/azure-linux-4-preview.md)]

## Prerequisites

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/free/) before you begin.

## Create a virtual machine

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Search for **Virtual machines** and select **Create** > **Virtual machine**.

### Configure the Basics tab

1. Under **Project details**, select your **Subscription** and create a new **Resource group** named `myResourceGroup`.
1. Under **Instance details**, configure the following settings:

    | Setting | Value |
    | ------- | ----- |
    | **VM name** | `myVM` |
    | **Availability options** | No infrastructure redundancy required |
    | **Security type** | Standard |
    | **Image** | Azure Linux 4.0 |
    | **Size** | Use defaults (varies by region) |

1. Under **Administrator account**, configure the following settings:

    | Setting | Value |
    | ------- | ----- |
    | **Authentication type** | SSH public key |
    | **Username** | `azureuser` |
    | **SSH key source** | Generate new key pair |
    | **Key pair name** | `myKey` |

1. Under **Inbound port rules**, set **Public inbound ports** to **Allow selected ports** and select **SSH (22)** and **HTTP (80)**.
1. Select **Review + create** > **Create**.

### Download the SSH key

When prompted, select **Download private key and create resource** and save the `myKey.pem` file to an accessible location.

### Get the public IP address of the VM

1. After deployment completes, select **Go to resource**.
1. On the VM's **Overview** page, copy the **Public IP address**.

## Connect to the VM

1. Set permissions on the key file (macOS/Linux):

    ```bash
    chmod 400 ~/Downloads/myKey.pem
    ```

1. SSH into the VM. Replace `<PUBLIC_IP>` with the IP address you copied:

    ```bash
    ssh -i ~/Downloads/myKey.pem azureuser@<PUBLIC_IP>
    ```

    > [!TIP]
    > You can reuse this SSH key for future VMs by selecting **Use a key stored in Azure** when creating a new VM.

## Clean up resources

When you no longer need the VM, delete the resource group to remove all associated resources using the following steps:

1. Open the **Overview** page for the VM in the Azure portal.
1. Select the **Resource group** link.
1. Select **Delete resource group**.
1. Enter the resource group name to confirm, and then select **Delete**.

> [!TIP]
> To avoid charges without deleting the VM, you can enable **Auto-shutdown** under **Operations** > **Auto-shutdown** on the VM page in the portal.

## Related content

For more information about Azure Linux, see the following resources:

- [Azure Linux Virtual Machines (VM) / Virtual Machine Scale Sets (VMSS) in Azure](./azure-linux-vm-vmss-overview.md)
- [Overview of Azure Linux security and compliance](./security-overview.md)
