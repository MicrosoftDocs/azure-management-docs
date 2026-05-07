---
title: Enable VM Extensions Using Red Hat Ansible
description: This article describes how to deploy virtual machine extensions to Azure Arc-enabled servers running in hybrid cloud environments by using Red Hat Ansible automation.
ms.date: 09/03/2024
ms.topic: how-to
ms.custom:
  - devx-track-ansible
  - build-2025
# Customer intent: "As an IT administrator managing Azure Arc-enabled servers, I want to deploy and automate VM extensions using Red Hat Ansible, so that I can efficiently manage and scale my hybrid cloud environment."
---

# Deploy and manage Arc extensions using Ansible

This article shows how to deploy, update, validate, and remove Azure Arc machine extensions using the Azure Ansible collection's (azure.azcollection) [azure_rm_arcmachineextensions](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/content/module/azure_rm_arcmachineextensions/?keywords=arc) module.

You learn how to:

- Deploy extensions to Arc-enabled servers
- Update or remove extensions
- Validate extension deployment and health
- Choose the right authentication method for at-scale automation
- Operate extension deployment as a continuous compliance workflow

## Overview

Azure Arc–enabled servers support [virtual machine (VM) extensions](/azure/azure-arc/servers/manage-vm-extensions#extensions) that provide post-deployment configuration and automation capabilities for Windows and Linux machines across hybrid, multi-cloud environments.

Common use cases include:

- Monitoring and observability with Azure Monitor
- Security with Defender for Endpoint
- Compliance with Azure Policy
- Configuration management with Machine Configuration
- Post-deployment scripting with Custom Scripting

The [azure_rm_arcmachineextensions](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/content/module/azure_rm_arcmachineextensions/?keywords=arc) module enables you to manage the lifecycle of extensions:

- Install extensions
- Update configuration or version
- Remove extensions

The [azure_rm_arcmachineextensions_info](https://galaxy.ansible.com/ui/repo/published/azure/azcollection/content/module/azure_rm_arcmachineextensions_info/?keywords=arc) module complements this by enabling validation and troubleshooting.

In most environments, extension management is a **day-2 operations workflow**:

- Configure machines with required extensions
- Validate their state
- Remediate drift over time

## Prerequisites

### Azure requirements

- Machines already connected to Azure Arc

### Permissions

To deploy and manage extensions, your identity must have permissions such as:

- Read Arc-enabled machines
- Read extensions
- Create/update/delete extensions

The **Azure Connected Machine Resource Administrator** role includes these permissions.

### Ansible requirements

- Ansible Core or Ansible Automation Platform (AAP)
- [Azure collection](https://galaxy.ansible.com/ui/repo/published/azure/azcollection) installed

## Authentication guidance for at-scale deployment

Choosing the right authentication method is critical for security and scalability when deploying extensions across many machines.

### Authentication options summary

| Method | Best for | Recommendation |
|--------|----------|---------------|
| Azure CLI (`auth_source: cli`) | Local development and testing | Use for dev only |
| Managed identity (`auth_source: msi`) | Production automation from Azure or Arc control nodes | Recommended |
| Environment variables (`auth_source: env`) | AAP, CI/CD, or external automation | When MSI not available |

### Managed identity

Use managed identity when your Ansible control node runs on an Azure VM or an Azure Arc–enabled server. Follow the steps below to assign the Ansible control node managed identity with the [**Azure Connected Machine Resource Administrator** role](/azure/azure-arc/servers/manage-vm-extensions#extension-deployment-prerequisites).

#### Azure portal

Use the Azure portal to assign the required role to the Ansible control node's managed identity. This option applies to Azure VM Managed Identity *or* Arc-enabled server Managed Identity.

1. Go to [Azure portal](https://portal.azure.com/).
1. Navigate to the **Subscription** or **Resource Group** where you want to onboard machines.
1. Select **Access control (IAM)**.
1. Select **Add → Add role assignment**.
1. Select the role: **Azure Connected Machine Resource Administrator**.
1. For **Assign access to**, choose **Managed identity**.
1. Select **Virtual machine** (if control node is an Azure VM) or **Azure Arc–enabled server**.
1. Pick the control node resource.
1. Select **Save**.


Once the Ansible control node's managed identity has the required role, you can use it as your authentication source in your playbook—no interactive login, secrets, or environment variables are required. 

To use managed identity authentication in your playbook, set the authentication source as follows:

```yaml
auth_source: msi
```

### Azure CLI authentication

 This option is recommended for interactive use and development environments. Use the currently signed-in Azure CLI identity on the Ansible control node. Set the following in your playbook:

```yaml
auth_source: cli
```

Before running your playbook, sign in to Azure with this Azure CLI command (this opens a new browser window for login):

```azurecli
az login
```

If you do not have a browser for login, use this Azure CLI command to log into Azure using your device:

```azurecli
az login --use-device-code
```

After you sign in, you can verify and/or set the subscription:

```azurecli
az account show
az account set --subscription "<subscription-id-or-name>"
```

## Arc extensions

The following table highlights commonly used extensions for Arc-enabled servers. You can view the [full list of extensions](/azure/azure-arc/servers/manage-vm-extensions#extensions).

| Extension | Publisher | Type | Use case |
|-----------|-----------|------|----------|
| Azure Monitor Agent | Microsoft.Azure.Monitor | AzureMonitorLinuxAgent / AzureMonitorWindowsAgent | Collect logs and metrics |
| Dependency Agent | Microsoft.Azure.Monitoring.DependencyAgent | DependencyAgentLinux / Windows | Dependency mapping |
| Custom Script | Microsoft.Compute | CustomScriptExtension | Run scripts for configuration |
| Key Vault | Microsoft.Azure.KeyVault | KeyVaultForLinux / Windows | Retrieve certificates and secrets |
| Defender for Cloud | Microsoft.Azure.Security | MDE.* | Endpoint protection and security |

## Deploy an extension

Use the `azure_rm_arcmachineextensions` module to install and configure extensions on Arc-enabled servers. Each extension defines a specific capability—such as monitoring, security, or configuration—and is deployed to a target machine using a combination of publisher, type, and settings. The module is idempotent, meaning it can be safely re-run to ensure the desired state is applied across your environment. For production scenarios, use managed identity authentication to enable secure, non-interactive deployments at scale.

The example playbook below installs the Azure Monitor Agent (AMA) extension on an Arc-enabled Linux server. The extension is deployed using managed identity authentication, enabling secure, non-interactive automation without requiring stored credentials. The playbook specifies the extension's publisher and type to identify the monitoring capability, and uses `state: present` to ensure the extension is installed. Because the module is idempotent, this playbook can be safely re-run to maintain the desired monitoring baseline across your environment.

```yaml
- name: Deploy Azure Monitor Agent extension
  hosts: localhost
  connection: local
  vars:
    auth_source: msi
  tasks:
    - name: Install extension
      azure.azcollection.azure_rm_arcmachineextensions:
        resource_group: "arc-rg"
        machine_name: "my-arc-server"
        name: "AzureMonitorLinuxAgent"
        publisher: "Microsoft.Azure.Monitor"
        type: "AzureMonitorLinuxAgent"
        type_handler_version: "1.0"
        settings: {}
        state: present
```

## Update an extension

You can [enable automatic extension updates](/azure/azure-arc/servers/manage-automatic-vm-extension-upgrade?tabs=azure-portal) using Azure portal, Azure CLI, or PowerShell.

If you would like to manually manage the extension updates, use the `azure_rm_arcmachineextensions` module to update an existing extension's configuration or version on an Arc-enabled server. By specifying updated values—such as a newer `type_handler_version` or modified settings—the module ensures the extension reflects the desired configuration.

This example playbook updates an existing extension by specifying a new version or configuration. When run, the module compares the current state of the extension with the desired configuration and applies changes only if needed. This enables safe, repeatable updates across multiple machines without affecting already compliant systems.

```yaml
- name: Update extension
  azure.azcollection.azure_rm_arcmachineextensions:
    resource_group: "arc-rg"
    machine_name: "my-arc-server"
    name: "AzureMonitorLinuxAgent"
    publisher: "Microsoft.Azure.Monitor"
    type: "AzureMonitorLinuxAgent"
    type_handler_version: "1.1"
    state: present
```

## Remove an extension

Use the `azure_rm_arcmachineextensions` module with `state: absent` to remove an extension from an Arc-enabled server. This is useful for decommissioning unused extensions, resolving failed deployments, or enforcing a new baseline configuration.

The example playbook below removes an extension from an Arc-enabled server using `state: absent`. This is useful for decommissioning unused extensions or enforcing a revised configuration baseline. The operation is idempotent, so it can be safely re-run even if the extension has already been removed.

```yaml
- name: Remove extension
  azure.azcollection.azure_rm_arcmachineextensions:
    resource_group: "arc-rg"
    machine_name: "my-arc-server"
    name: "AzureMonitorLinuxAgent"
    state: absent
```

## Validate extension deployment

Use the `azure_rm_arcmachineextensions_info` module to retrieve information about extensions deployed on an Arc-enabled server and verify their configuration and status. This allows you to confirm that required extensions are installed and operating as expected by checking properties such as name, version, and provisioning state. Validation steps can be integrated into automation workflows to detect missing, failed, or misconfigured extensions and trigger remediation actions.

This example playbook retrieves extension details from an Arc-enabled server and verifies that the required extension is installed. It checks the returned extension list and fails the playbook if the expected extension is missing, enabling you to enforce compliance and trigger remediation workflows when needed.

```yaml
- name: Validate extension deployment
  hosts: localhost
  connection: local
  vars:
    auth_source: msi
  tasks:
    - name: Get extension details
      azure.azcollection.azure_rm_arcmachineextensions_info:
        resource_group: "arc-rg"
        name: "my-arc-server"
      register: extension_info

    - name: Ensure extension is installed
      fail:
        msg: "Azure Monitor Agent is not installed"
      when: "'AzureMonitorLinuxAgent' not in extension_info.extensions | map(attribute='name') | list"
```

### Detect unhealthy extensions

This example task iterates through all extensions on the target machine and identifies any that are not in a successful provisioning state. It is commonly used to detect failed installations or misconfigured extensions as part of a validation or monitoring workflow.

```yaml
- name: Identify failed extensions
  debug:
    msg: "{{ item.name }} is not healthy"
  loop: "{{ extension_info.extensions }}"
  when: item.provisioningState != "Succeeded"
```

### Remediate drift

This task reapplies the desired extension configuration to ensure that the target machine matches the defined baseline. Because the module is idempotent, it only installs or updates the extension if it is missing or misconfigured, making it safe to run repeatedly for ongoing compliance enforcement.

```yaml
- name: Re-apply extension baseline
  azure.azcollection.azure_rm_arcmachineextensions:
    resource_group: "arc-rg"
    machine_name: "my-arc-server"
    name: "AzureMonitorLinuxAgent"
    publisher: "Microsoft.Azure.Monitor"
    type: "AzureMonitorLinuxAgent"
    state: present
```

## Use Ansible Automation Platform (AAP) for at-scale execution

Extension deployment and validation can be executed using Ansible Automation Platform (AAP) to enable centralized management, scheduling, and repeatability.

AAP allows you to define job templates that run playbooks without requiring manual execution and provides centralized visibility into automation runs.

### Recommended job template structure

Create separate job templates for each stage of the workflow:

| Job template | Purpose |
|--------------|---------|
| Deploy extensions | Install required extensions across machines |
| Validate extensions | Check extension health and compliance |
| Remediate drift (optional) | Reapply desired state for non-compliant machines |

### Configure a deployment job template

Set up a job template with the following values:

| Field | Value |
|-------|-------|
| Job Type | Run |
| Inventory | Target inventory (for example, `arc_linux`) |
| Project | Repository containing your playbooks |
| Playbook | Example: `deploy_extensions.yml` |
| Credentials | Azure Resource Manager credential |
| Limit | Optional group or host filter |

### Configure authentication in AAP

For Azure authentication in AAP:

- Use managed identity when your execution environment runs on Azure VM or Arc-enabled server.

### Enable continuous compliance

To maintain extension compliance at scale:

- Schedule the validation job template to run periodically (for example, hourly or daily).
- Optionally trigger the remediation job template when validation failures are detected.
- Use inventory groups or dynamic inventory to automatically include new machines.

### Operational pattern

Using AAP, your extension workflow becomes:

1. Deploy extensions (baseline configuration)
1. Validate extension state (scheduled compliance checks)
1. Remediate drift (automated correction)

This model enables consistent and repeatable management of extensions across large hybrid environments.

## Related content

- Learn how to deploy, manage, and remove VM extensions using the [Azure CLI](/azure/azure-arc/servers/manage-vm-extensions-cli), [PowerShell](/azure/azure-arc/servers/manage-vm-extensions-powershell), or [Azure Resource Manager templates](/azure/azure-arc/servers/manage-vm-extensions-template).
- Review the [Troubleshoot VM extensions guide](/azure/azure-arc/servers/troubleshoot-vm-extensions).
