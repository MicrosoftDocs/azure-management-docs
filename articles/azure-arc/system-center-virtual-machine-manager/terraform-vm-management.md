---
title:  Terraform based VM management
description: This article describes how to programmatically perform lifecycle operations on the SCVMM managed on-premises virtual machines using the Terraform templates.
ms.topic: how-to 
ms.date: 01/08/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
author: PriskeyJeronika-MS
ms.author: v-gjeronika
manager: jsuri
---

# Terraform based VM management

This article describes how to programmatically perform lifecycle operations on the SCVMM managed on-premises virtual machines using the Terraform templates.

[Hashicorp Terraform](https://www.terraform.io/) is an open-source IaC (Infrastructure-as-Code) tool to configure and deploy cloud infrastructure. It codifies infrastructure in configuration files that describe the desired state for your topology. Terraform enables the management of your SCVMM based virtual infrastructure by using AzAPI Terraform providers.

The complete set of Terraform templates available with Azure Arc enabled SCVMM can be accessed [here](/azure/templates/microsoft.scvmm/virtualmachineinstances?pivots=deployment-language-terraform).

The following scenarios are covered in this article:

- Create a new SCVMM-managed on-premises Virtual Machine from Azure

## Scenario 1: Create a new Virtual Machine

### Prerequisites

Ensure to have the following prerequisites before you create a new virtual machine:

- An Azure subscription and resource group where you have Arc SCVMM VM Contributor role.
- An Arc-enabled SCVMM server with the Azure Arc resource bridge in a Running state.
- A workstation machine with Terraform installed.
- An Azure-enabled cloud resource on which you have Arc SCVMM Private Cloud Resource User role.
- An Azure-enabled virtual machine template resource on which you have Arc SCVMM Private Cloud Resource User role.
- An Azure-enabled virtual network resource on which you have Arc SCVMM Private Cloud Resource User role.

### Step 1: Define the VM variables in a *variables.tf* file

Here is a sample `variables.tf` file with placeholders.

Add a Terraform code snippet - [variables.tf](https://microsoftapc.sharepoint.com/:u:/t/AzureCoreIDC/EfSDnWAJi9NIlMkDKQ1ka-UBzPV3Nnot78IPlg4bKG7JtQ?e=XuHxsZ)

### Step 2: Define the metadata and credentials in a *tfvars* file 

Here is a sample `createscvmmvm.tfvars` file with placeholders.

Add a Terraform code snippet - [createVMsample.tfvars](https://microsoftapc.sharepoint.com/:u:/t/AzureCoreIDC/ES3yFDw7QMRPs2ROYI7Ta4AB-IN-jSecrOPElnG5Sv7RDQ?e=G3y0dR)

### Step 3: Define the VM configuration in a *main.tf* file

Here is a sample `main.tf` file with placeholders.

Add a Terraform code snippet - [main.tf](https://microsoftapc.sharepoint.com/:u:/t/AzureCoreIDC/EYkIiQzIU8NAntr6_46gfngBu986tujzClzZohbBMCRa9Q?e=0gMj84)

### Step 4: Run Terraform commands

Use the -var-file flag to pass the *.tfvars* file during Terraform commands.

- Initialize Terraform (if not already initialized): `terraform init`
- Validate the configuration: `terraform validate -var-file="createVMsample.tfvars"`
- Plan the changes: `terraform plan -var-file="createVMsample.tfvars"`
- Apply the changes: `terraform apply -var-file="createVMsample.tfvars"`

Confirm the prompt by entering yes to apply the changes.

## Best practices

- **Use version control**: Keep your Terraform configuration files under version control (for example, Git) to track changes over time.
- **State management**: Regularly back up your Terraform state files to avoid data loss.
