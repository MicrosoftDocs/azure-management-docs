---
title: Connect machines to Azure Arc at scale using Ansible
description: In this article, you learn how to onboard Azure Arc–enabled servers at scale using Ansible Core and Ansible Automation Platform (AAP), including recommended authentication options such as managed identity and service principal.
ms.date: 12/30/2024
ms.topic: how-to
ms.custom: template-how-to, devx-track-ansible
# Customer intent: "As a cloud administrator, I want to onboard multiple machines to Azure Arc using Ansible, so that I can efficiently manage and monitor my hybrid infrastructure at scale."
---

# Connect machines to Azure Arc at scale with Ansible
 
This article shows how to connect Linux and Windows machines to Azure Arc at scale using the Azure Arc Ansible role included in the [azure.azcollection](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/). The Azure Arc Ansible role automates installation of the Azure Connected Machine agent and registers each target machine with Azure Arc. This enables consistent, repeatable onboarding across large environments without requiring interactive login or manual installation steps. 

The Azure Arc Ansible role can be used with Ansible Core or Ansible Automation Platform (AAP). This article covers both options and explains how to use each to onboard machines to Azure Arc at scale. 

## Overview 
The **azure_arc role** installs the Azure Connected Machine agent and registers each host with Azure Arc in the specified resource group and subscription. Once registered, the machine appears in Azure as an Arc-enabled server and can be managed using Azure services such as Azure Policy, Update Manager, and Monitor. The Azure Arc Ansible role is used to onboard machines to Azure Arc. It does not discover machines automatically or replace Azure Policy–based management after onboarding. 

## Prerequisites 
Before using the Azure Arc Ansible role, ensure the following requirements are met: 

### Azure permissions 
The identity used by Ansible (Azure CLI user, service principal, or managed identity) must be assigned **the Azure Connected Machine Onboarding role** at the subscription or resource group scope.

In addition, the subscription must have the following [resource providers registered](/azure/azure-resource-manager/management/resource-providers-and-types):
- Microsoft.HybridCompute
- Microsoft.GuestConfiguration 

### Ansible prerequisites 
Before running the Azure Arc Ansible role, ensure:
- Network connectivity from the machines you’d like to Arc-enable to the required Azure Arc URLs. Review and address applicable [network requirements](network-requirements.md). To reduce the number of Azure Arc URLs that must be allowlisted in your network or firewall from ~20 URLs to ~7 URLs, you can [use the Arc gateway](arc-gateway.md). 
- The account that Ansible uses to connect to target machines must have administrator permissions to install software and manage system services (for example, root or sudo access on Linux).

## Role variables
The Azure Arc role supports the following variables:

| Variable | Description |
|---|---|
| `azure_arc_resource_group` | Resource group where the machine will be registered |
| `azure_arc_subscription_id` | Azure subscription ID |
| `azure_arc_tenant_id` | Azure tenant ID |
| `azure_arc_location` | Azure region for the Arc resource (defaults to the resource group location) |
| `azure_arc_cloud` | Azure cloud to use (default: AzureCloud) |
| `azure_arc_tags` | Tags applied to the Arc resource (default: `{ archost: "true" }`) |
| `proxy` | Optional proxy configuration (hostname and port). Example: http://proxy.contoso.com:8080 |

## Authentication options
 
The Azure Arc role supports multiple authentication methods for automation scenarios. This article covers the most common options, including Azure CLI–based authentication and managed identity.

## Azure CLI credentials
 
Use the currently signed-in Azure CLI identity on the Ansible control node. This option is recommended for interactive use and development environments. Set the following in your playbook:

```yaml
auth_source: cli
```

Before running your playbook, sign in to Azure with this Azure CLI command (this will open a new browser window for login):
```azure cli
az login
``` 
If you do not have a browser for login, use this Azure CLI command to log into Azure using your device:
```
az login --use-device-code
```
After you sign in, you can verify and/or set the subscription:
```
az account show
az account set --subscription "<subscription-id-or-name>"
```

## Managed identity authentication
 
If the Ansible control node runs on an Azure virtual machine or an Azure Arc–enabled server, you can authenticate using a system-assigned managed identity. Each Azure Arc–enabled server automatically receives its own system-assigned managed identity when it is connected to Azure Arc. This identity is machine-specific and represents that server when authenticating to Azure services. By default, managed identities have no Azure RBAC permissions. Before using managed identity authentication with the Azure Arc Ansible role, you must explicitly grant the managed identity permission to onboard machines. Managed identity is the recommended authentication method for production automation as it removes the need to store credentials or secrets. 

## Assign the managed identity to the onboarding role
The Ansible control node’s managed identity must be assigned **the Azure Connected Machine Onboarding role** at the subscription or resource group scope before running the playbook.

### Option 1: Azure portal 
Use the Azure portal to assign the required role to the Ansible control node’s managed identity. This option applies to Azure VM Managed Identity _or_ Arc-enabled server Managed Identity. This flow is supported for Arc-enabled server identities because Arc creates a service principal in Microsoft Entra ID for the machine identity. 

1. Go to [Azure portal](https://portal.azure.com/) 
1. Navigate to the **Subscription** or **Resource Group** where you want to onboard machines. 

1. Select **Access control (IAM).**

1. Select **Add → Add role assignment.**

1. Select the role: **Azure Connected Machine Onboarding.**

1. For **Assign access to**, choose **Managed identity.**

1. Select **Virtual machine** (if control node is an Azure VM) or **Azure Arc–enabled server.**

1. Pick the control node resource.

1. Click Save.

### Option 2: Azure CLI 
Use Azure CLI commands to assign the required role to the Ansible control node’s managed identity. 

#### Get the managed identity principal ID
The managed identity principal ID is the Microsoft Entra object ID (principal ID) of the system‑assigned managed identity for the Ansible control node.

If the Ansible control node is an Arc-enabled server, use this Azure CLI command:
```
az connectedmachine show \
--name \<arc-server-name\> \
--resource-group \<rg-name\> \
--query identity.principalId \
-o tsv
```

If the Ansible control node is an Azure VM, use this Azure CLI command:
```
az vm show \
--name \<vm-name\> \
--resource-group \<rg-name\> \
--query identity.principalId \
-o tsv
```

#### Assign the role to the managed identity

Based on the scope of your Arc onboarding experience, you need to assign the role for either the subscription or resource group. You need the managed identity principal ID from the previous step as the `--assignee` value. Below are the corresponding Azure CLI commands based on the selected scope:

**Subscription scope** 
This sets the Azure Connected Machine Onboarding role at the subscription level:

```
az role assignment create \
--assignee \<identity-principal-id> \
--role "Azure Connected Machine Onboarding" \
--scope /subscriptions/<subscription-id> 
```
**Resource group scope** 
This sets the Azure Connected Machine Onboarding role at the resource group level:

```
az role assignment create \
--assignee \<identity-principal-id> \
--role "Azure Connected Machine Onboarding" \
--scope /subscriptions/<subscription-id>/resourceGroups/<rg-name>
```

### Set the playbook to use managed identity
Once the Ansible control node’s managed identity has the required role, you can use it as your authentication source in your playbook - no interactive login, secrets, or environment variables are required. To use managed identity authentication in your playbook, set the authentication source as follows: 

```yaml
auth_source: msi
```

### How the managed identity authentication works

When the playbook runs, the Azure SDK authenticates by requesting a token from the local metadata service. Azure virtual machines use the Azure Instance Metadata Service. Azure Arc–enabled servers use the Azure Arc Hybrid Instance Metadata Service. The token represents the managed identity of the control node. Azure then evaluates whether that identity has permission to create and register Azure Arc resources.

## Deploy Azure Arc at scale using Ansible
 
The Azure Arc Ansible role can be used with Ansible Core or Ansible Automation Platform to perform at scale Arc-enable machines. This section covers how to use the Azure Arc role in both scenarios. The Azure Arc role is idempotent. You can rerun the playbook as new machines are added to inventory. Machines already connected to Azure Arc are skipped.

## Deploy Azure Arc at scale using Ansible Core
 
When using Ansible Core, at scale onboarding is achieved by defining groups of machines in inventory and running the same playbook across those groups. The Azure Arc Ansible role is executed once per host and can be safely run across large numbers of machines. 

### Choose an authentication method
Before running the playbook, decide which authentication method your control node will use. The examples in the following steps show options for Azure CLI (recommended for development) and managed identity (recommended for production). 

### Define your inventory
Group machines logically by environment or operating system. Inventory files can be static (INI or YAML) or dynamically generated using scripts or inventory plugins. We suggest grouping your machines to easily target them. 
Example inventory file: 
```yaml
[arc_linux] 
linux-01 
linux-02 
linux-03 
[arc_windows] 
win-01 
win-02 
```
### Configure role variables
Create a group variables file to define the Azure environment your machines will onboard into. This keeps your playbook clean and reusable across environments. 
Example: group\_vars/arc\_linux.yml 
```yaml
azure_arc_resource_group: "my-arc-rg" 
azure_arc_subscription_id: "<your-subscription-id\>" 
azure_arc_tenant_id: "<your-tenant-id\>" 
azure_arc_location: "eastus" 
azure_arc_tags: 
environment: "production" 
archost: "true"
```
### Write the playbook
Choose the authentication option that matches your environment. 

#### Option A: Azure CLI authentication (recommended for development)
Use the signed-in Azure CLI identity on the Ansible control node. Before running the playbook, you need to sign in to Azure CLI.

Example: arc-onboard.yml 

```yaml
- name: Connect Linux machines to Azure Arc (CLI auth)

  hosts: arc_linux
  gather_facts: true

  vars:
    auth_source: cli

  roles:
    - role: azure.azcollection.azure_arc
      vars:
        azure_arc_resource_group: "{{ azure_arc_resource_group }}"
        azure_arc_subscription_id: "{{ azure_arc_subscription_id }}"
        azure_arc_tenant_id: "{{ azure_arc_tenant_id }}"
        azure_arc_location: "{{ azure_arc_location }}"
        azure_arc_tags: "{{ azure_arc_tags }}"
```

 
#### Option B: Managed Identity authentication (recommended for production)
If your Ansible control node is an Azure VM or Arc-enabled server with a managed identity assigned, no additional login step is required. 
Example: arc-onboard.yml 
```yaml
- name: Connect Linux machines to Azure Arc (Managed Identity auth)

  hosts: arc_linux
  gather_facts: true

  vars:
    auth_source: msi

  roles:
    - role: azure.azcollection.azure_arc
      vars:
        azure_arc_resource_group: "{{ azure_arc_resource_group }}"
        azure_arc_subscription_id: "{{ azure_arc_subscription_id }}"
        azure_arc_tenant_id: "{{ azure_arc_tenant_id }}"
        azure_arc_location: "{{ azure_arc_location }}"
        azure_arc_tags: "{{ azure_arc_tags }}"
```
Run the playbook: 
```bash
ansible-playbook -i inventory.ini arc-onboard.yml
```
## Deploy Azure Arc at scale using Ansible Automation Platform
 
When using **Ansible Automation Platform (AAP)**, inventories and host groups are managed centrally in the platform. With AAP, you do not need to maintain local inventory files. AAP is recommended for large or dynamic environments. AAP supports inventories that automatically discover machines from external sources such as virtualization platforms, cloud providers, or CMDB systems. 

### Set up your AAP credential for Azure authentication
AAP manages credentials centrally. Create an Azure Resource Manager credential in AAP with one of the following authentication methods. 

**AAP Authentication Methods**
The following table summarizes supported AAP authentication methods:

| Authentication Method | When to Use | Notes |
| --- | --- | --- |
| Managed Identity | Production; AAP execution environment runs on an Azure VM or Arc-enabled server | Recommended for production |
| Service Principal | Production; general purpose or AAP outside Azure | Requires client ID and secret |
 
Azure CLI credentials are not applicable in AAP. Use Managed Identity or a Service Principal for AAP-based automation. To create an Azure credential in AAP: 
1. In the **AAP web UI**, go to **Credentials** and select **Add**. 
1. Set **Credential Type** to **Microsoft Azure Resource Manager**. 
1. For **Managed Identity**: enable **Use the environment's managed identity**. For **Service Principal**: enter your Subscription ID, Client ID, Client Secret, and Tenant ID. 
1. Select **Save**.
### Configure your inventory
AAP inventories can be static or dynamically sourced. For large or dynamic environments, use a constructed inventory to group machines automatically based on host metadata. 
#### Option A: Static inventory with host groups
1. In AAP, go to **Inventories > Add > Inventory**, then add hosts manually or import from a file. 
1. Add your Linux hosts to a group named arc_linux: 
```yaml
linux-server-01 
linux-server-02 
linux-server-03 
```
#### Option B: Dynamic inventory with constructed groups (recommended for scale) 
Use an inventory source (for example, VMware vSphere, AWS, or Azure) and define constructed groups based on host variables. For example, to automatically group all Linux machines: 
Example: constructed.yml 
```yaml
plugin: constructed 
groups: 
arc_linux: ansible_os_family == "RedHat" or ansible_os_family == "Debian"
```
Newly added machines that match the group condition are automatically included in subsequent job runs. 
### Create a project 
In AAP, a Project links to a source control repository containing your playbook and role requirements file. 
1. Go to Projects > Add. 
1. Set SCM Type to Git. 
1. Enter your repository URL. 
1. Select Save. 
1. Ensure your repository includes a requirements.yml file at the root to install the azure.azcollection: 
Example: requirements.yml 
```yaml
collections: 
- name: azure.azcollection 
source: https://galaxy.ansible.com
```
### Write the Arc onboarding playbook
Store this playbook in your project repository. The Azure credential configured earlier is injected automatically by AAP at runtime — you do not need to hardcode secrets in the playbook. 
#### Option A: Managed Identity

In the example below, the `auth_source` is set to managed identity.

```yaml
- name: Connect machines to Azure Arc (AAP - Managed Identity)

  hosts: arc_linux
  gather_facts: true

  vars:
    auth_source: msi
    azure_arc_resource_group: "my-arc-rg"
    azure_arc_subscription_id: "<your-subscription-id>"
    azure_arc_tenant_id: "<your-tenant-id>"
    azure_arc_location: "eastus"
    azure_arc_tags:
      environment: "production"
      archost: "true"

  roles:
    - role: azure.azcollection.azure_arc
```

#### Option B: Service Principal

When `auth_source: env` is set, the role uses the Azure credential configured in the AAP job template. The service principal values are injected automatically by AAP — you do not need to hardcode them in the playbook. 

```yaml
- name: Connect machines to Azure Arc (AAP - Service Principal)

  hosts: arc_linux
  gather_facts: true

  vars:
    auth_source: env
    azure_arc_resource_group: "my-arc-rg"
    azure_arc_subscription_id: "<your-subscription-id>"
    azure_arc_tenant_id: "<your-tenant-id>"
    azure_arc_location: "eastus"
    azure_arc_tags:
      environment: "production"
      archost: "true"

  roles:
    - role: azure.azcollection.azure_arc
```
### Create and run a job template
1. In AAP, go to **Templates > Add > Job Template**. 
1. Configure the following fields, then save and launch the job: 

**AAP Job Template Fields**

| Field | Value |
| --- | --- |
| Name | Connect machines to Azure Arc |
| Job Type | Run |
| Inventory | Your inventory (for example, the arc\_linux group) |
| Project | Your project (linked to your repository) |
| Playbook | arc-onboard.yml |
| Credentials | Your Azure Resource Manager credential |
| Limit | arc\_linux (or leave blank to target all hosts) |
 


## Continuous and large-scale onboarding

Because AAP inventories are dynamically maintained: 
- Newly discovered machines are automatically included in subsequent job runs. 
- Machines already connected to Azure Arc are skipped (the role is idempotent). 
The same job template can be scheduled to run on a recurring basis to continuously onboard new machines as they are added to your environment. Schedule the job template to run on a recurring basis (for example, nightly) to automatically onboard new machines as they are added to your environment. 

## Verify the connection with Azure Arc 
After installing the agent and configuring it to connect to Azure Arc-enabled servers, go to the Azure portal to verify that the servers in your target hosts successfully connected. View your machines in the [Azure portal](https://portal.azure.com/). 

## Next steps 
- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring. 
- Review connection troubleshooting information in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md). 
- Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview) for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying that the machine is reporting to the expected Log Analytics workspace, enabling monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.
