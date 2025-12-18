---
title: Connect machines to Azure Arc at scale using Ansible Playbooks
description: In this article, you learn how to connect machines to Azure using Azure Arc-enabled servers using Ansible playbooks.
ms.date: 12/30/2024
ms.topic: how-to
ms.custom: template-how-to, devx-track-ansible
# Customer intent: "As a cloud administrator, I want to onboard multiple machines to Azure Arc using Ansible playbooks, so that I can efficiently manage and monitor my hybrid infrastructure at scale."
---

# Connect machines to Azure Arc at scale using Ansible playbooks

You can onboard Ansible-managed nodes to Azure Arc-enabled servers at scale using Ansible playbooks. To do so, download, modify, and then run the appropriate playbook.

## Prerequisites

Before you get started, ensure you have the following:

- An Azure subscription. If you don't have one, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Review the [general prerequisites](prerequisites.md) and verify that your subscription and resources meet the requirements.
- Familiarity with [supported Azure regions](overview.md#supported-regions) and other related considerations.
- Review the [at-scale planning guide](plan-at-scale-deployment.md) to understand the design and deployment criteria, as well as management and monitoring recommendations.
- An Ansible control node with Ansible installed and configured to manage your target machines.
- Network connectivity from the Ansible control node to both the target machines and Azure endpoints.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Generate a service principal and collect Azure details

Before you can run the script to connect your machines, you'll need to:

1. Follow the steps to [create a service principal for onboarding at scale](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

    * Assign the Azure Connected Machine Onboarding role to your service principal and limit the scope of the role to the target Azure subscription or resource group.
    * Make a note of the Service Principal Secret and Service Principal Client ID; you'll need these values later.

1. Collect the details in the following table about where the Azure Arc-enabled resource will onboard. You'll use this information to update the playbook in the next sections. 

| Parameter | Description | Variable name in playbook |
|-----------|-------------|---------------------------|
| Service Principal Client ID | The application (client) ID of your service principal | `service_principal_id` |
| Service Principal Secret | The client secret value of your service principal | `service_principal_secret` |
| Tenant ID | Your Azure Active Directory tenant ID | `tenant_id` |
| Subscription ID | The Azure subscription where resources will be created | `subscription_id` |
| Resource Group | The name of the resource group for Arc-enabled servers | `resource_group` |
| Region | The Azure region where the Arc-enabled resource will onboard | `location` |

## Download the Ansible playbook

On your Ansible control node, copy the following Ansible playbook template and save it as `arc-server-onboard-playbook.yml`.

```yaml
---
- name: Onboard Linux and Windows Servers to Azure Arc-enabled servers with public endpoint connectivity
  hosts: all
  # vars:
  #   azure:
  #     service_principal_id: 'INSERT-SERVICE-PRINCIPAL-CLIENT-ID'
  #     service_principal_secret: 'INSERT-SERVICE-PRINCIPAL-SECRET'
  #     resource_group: 'INSERT-RESOURCE-GROUP'
  #     tenant_id: 'INSERT-TENANT-ID'
  #     subscription_id: 'INSERT-SUBSCRIPTION-ID'
  #     location: 'INSERT-LOCATION'
  tasks:
  - name: Check if the Connected Machine Agent has already been downloaded on Linux servers
    stat:
      path: /usr/bin/azcmagent
      get_attributes: False
      get_checksum: False
    register: azcmagent_lnx_downloaded
    when: ansible_system == 'Linux'

  - name: Download the Connected Machine Agent on Linux servers
    become: yes
    get_url:
      url: https://aka.ms/azcmagent
      dest: ~/install_linux_azcmagent.sh
      mode: '700'
    when: (ansible_system == 'Linux') and (azcmagent_lnx_downloaded.stat.exists == false)

  - name: Install the Connected Machine Agent on Linux servers
    become: yes
    shell: bash ~/install_linux_azcmagent.sh
    when: (ansible_system == 'Linux') and (not azcmagent_lnx_downloaded.stat.exists)

  - name: Check if the Connected Machine Agent has already been downloaded on Windows servers
    win_stat:
      path: C:\Program Files\AzureConnectedMachineAgent
    register: azcmagent_win_downloaded
    when: ansible_os_family == 'Windows'

  - name: Download the Connected Machine Agent on Windows servers
    win_get_url:
      url: https://aka.ms/AzureConnectedMachineAgent
      dest: C:\AzureConnectedMachineAgent.msi
    when: (ansible_os_family == 'Windows') and (not azcmagent_win_downloaded.stat.exists)

  - name: Install the Connected Machine Agent on Windows servers
    win_package:
      path: C:\AzureConnectedMachineAgent.msi
    when: (ansible_os_family == 'Windows') and (not azcmagent_win_downloaded.stat.exists)

  - name: Check if the Connected Machine Agent has already been connected
    become: true
    command:
     cmd: azcmagent check
    register: azcmagent_lnx_connected
    ignore_errors: yes
    when: ansible_system == 'Linux'
    failed_when: (azcmagent_lnx_connected.rc not in [ 0, 16 ])
    changed_when: False

  - name: Check if the Connected Machine Agent has already been connected on windows
    win_command: azcmagent check
    register: azcmagent_win_connected
    when: ansible_os_family == 'Windows'
    ignore_errors: yes
    failed_when: (azcmagent_win_connected.rc not in [ 0, 16 ])
    changed_when: False

  - name: Connect the Connected Machine Agent on Linux servers to Azure Arc
    become: yes
    shell: azcmagent connect --service-principal-id "{{ azure.service_principal_id }}" --service-principal-secret "{{ azure.service_principal_secret }}" --resource-group "{{ azure.resource_group }}" --tenant-id "{{ azure.tenant_id }}" --location "{{ azure.location }}" --subscription-id "{{ azure.subscription_id }}"
    when:  (ansible_system == 'Linux') and (azcmagent_lnx_connected.rc is defined and azcmagent_lnx_connected.rc != 0)

  - name: Connect the Connected Machine Agent on Windows servers to Azure
    win_shell: '& $env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe connect --service-principal-id "{{ azure.service_principal_id }}" --service-principal-secret "{{ azure.service_principal_secret }}" --resource-group "{{ azure.resource_group }}" --tenant-id "{{ azure.tenant_id }}" --location "{{ azure.location }}" --subscription-id "{{ azure.subscription_id }}"'
    when: (ansible_os_family == 'Windows') and (azcmagent_win_connected.rc is defined and azcmagent_win_connected.rc != 0)
```

## Modify the Ansible playbook

After downloading the Ansible playbook, complete the following steps:

1. Within the Ansible playbook, modify the variables under the **vars section** with the service principal and Azure details collected earlier:

    * Service Principal Client ID (`service_principal_id`)
    * Service Principal Secret (`service_principal_secret`)
    * Resource Group (`resource_group`)
    * Tenant ID (`tenant_id`)
    * Subscription ID (`subscription_id`)
    * Region (`location`)

1. Enter the correct hosts field capturing the target servers for onboarding to Azure Arc. You can employ [Ansible patterns](https://docs.ansible.com/ansible/latest/user_guide/intro_patterns.html#common-patterns) to selectively target which hybrid machines to onboard.

1. This template passes the service principal secret as a variable in the Ansible playbook. Note that [Ansible vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) can be used to encrypt this secret and the variables can be passed through a configuration file.

## Run the Ansible playbook

From the Ansible control node, run the Ansible playbook by invoking the `ansible-playbook` command:

```
ansible-playbook arc-server-onboard-playbook.yml
```

After the playbook runs, **PLAY RECAP** indicates all tasks completed successfully and surfaces any nodes where tasks failed.

## Verify the connection with Azure Arc

After installing the agent and configuring it to connect to Azure Arc-enabled servers, go to the Azure portal to verify that the servers in your target hosts have successfully connected. View your machines in the [Azure portal](https://aka.ms/hybridmachineportal).

## Next steps

- Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
- Review connection troubleshooting information in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).
- Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview) for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying that the machine is reporting to the expected Log Analytics workspace, enabling monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.
