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

# Enable Azure VM extensions by using Red Hat Ansible automation

This article shows you how to deploy [virtual machine (VM) extensions](manage-vm-extensions.md) to Azure Arc-enabled servers at scale by using Red Hat Ansible Automation Platform. The examples in this article rely on content developed and incubated by Red Hat through the [Ansible Content Lab for Cloud Content](https://cloud.lab.ansible.io/).

This article also uses the [Azure Infrastructure Configuration Demo](https://github.com/ansible-content-lab/azure.infrastructure_config_demos) collection. This collection contains many roles and playbooks that are pertinent to this article, including the following:

|File or folder  |Description  |
|---------|---------|
|playbook_enable_arc_extension.yml  |Playbook that's used as a job template to enable Azure Arc extensions.  |
|playbook_disable_arc-extension.yml  |Playbook that's used as a job template to disable Azure Arc extensions.  |
|roles/arc  |Ansible role that contains the reusable automation that the playbooks use.  |

> [!NOTE]
> The examples in this article target Linux hosts.

## Prerequisites

### Automation Controller 2.x

This article is applicable to both self-managed Ansible Automation Platform and Red Hat Ansible Automation Platform on Microsoft Azure.

### Automation execution environment

To use the examples in this article, you need an automation execution environment with both the Azure collection and the Azure CLI installed. Both are required to run the automation.

If you don't have an automation execution environment that meets these requirements, you can use [this example](https://github.com/scottharwell/cloud-ee).

For more information about building and configuring automation execution environments, see the [Red Hat Ansible documentation](https://docs.ansible.com/ansible/latest/getting_started_ee/index.html).

### Azure Resource Manager credential

A working account credential configured in Ansible Automation Platform for Azure Resource Manager is required. Ansible Automation Platform uses this credential to authenticate operations by using the Azure collection and the Azure CLI.

## Configure the content

To use the [Azure Infrastructure Configuration Demo collection](https://github.com/ansible-content-lab/azure.infrastructure_config_demos) in Automation Controller, follow these steps to set up a project with the repository:

1. Log in to Automation Controller.
1. On the left menu, select **Projects**.
1. Select **Add**, and then complete the fields of the form as follows:

   - **Name**: Enter **Content Lab - Azure Infrastructure Configuration Collection**.

   - **Execution Environment**: Select with the Azure collection and Azure CLI instead.

   - **Source Control Type**: Select **Git**.

   - **Source Control URL**: Enter `https://github.com/ansible-content-lab/azure.infrastructure_config_demos.git`.

   :::image type="content" source="media/migrate-ama/configure-content.png" alt-text="Screenshot of the Projects window to edit details." lightbox="media/migrate-ama/configure-content.png":::

1. Select **Save**.

After you save the details, the project should be synchronized with Automation Controller.

## Create job templates

The project that you created from the Azure Infrastructure Configuration Demo collection contains example playbooks that implement the reusable content in roles. You can learn more about the individual roles in the collection by viewing the [README file](https://github.com/ansible-content-lab/azure.infrastructure_config_demos/blob/main/README.md) included with the collection.

The following mapping within the collection can help you identify which extension you want to enable.

|Extension  |Extension variable name  |
|---------|---------|
|Microsoft Defender for Cloud integrated vulnerability scanner  |`microsoft_defender`  |
|Custom Script Extension  |`custom_script`  |
|Azure Monitor for VMs (insights)  |`azure_monitor_for-vms`  |
|Azure Key Vault virtual machine extension  |`azure_key_vault`  |
|Azure Monitor agent  |`azure_monitor_agent`  |
|Azure Automation Hybrid Runbook Worker extension  |`azure_hybrid_rubook`  |

You need to create templates so that you can enable and disable Azure Arc-enabled server VM extensions, as described in the following sections.

> [!NOTE]
> There are additional VM extensions not included in this collection. They're outlined in [Virtual machine extension management with Azure Arc-enabled servers](manage-vm-extensions.md#extensions).

### Enable Azure Arc VM extensions

This template is responsible for enabling an Azure Arc-enabled server VM extension on the hosts that you identify.

> [!IMPORTANT]
> Azure Arc supports enabling or disabling only a single extension at a time, so this process can take some time. If you try to enable or disable another VM extension with this template before Azure completes this process, the template reports an error.
>
> After the job template runs, it might take minutes to hours for each machine to report that the extension is operational. When the extension is operational, you can run this job template again with another extension without getting an error.

To create the template:

1. On the right menu, select **Templates**.
1. Select **Add**.
1. Select **Add job template**, and then complete the fields of the form as follows:

   - **Name**: Enter **Content Lab - Enable Arc Extension**.

   - **Job Type**: Select **Run**.

   - **Inventory**: Enter **localhost**.

   - **Project**: Enter **Content Lab - Azure Infrastructure Configuration Collection**.

   - **Playbook**: Enter `playbook_enable_arc-extension.yml`.

   - **Credentials**: Enter your Azure Resource Manager credential.

   - **Variables**: Enter the following code:

     ```bash
     ---
      resource_group: <your_resource_group>
      region: <your_region>
      arc_hosts:
      <first_arc_host>
      <second_arc_host>
      extension: microsoft_defender
     ```

     > [!NOTE]
     > Change `<your_resource_group>`, `<your_region>`, `<first_arc_host>`, and `<second_arc_host>` to match the names of your Azure resources. If you have a large number of Azure Arc hosts, use Jinja2 formatting to extract the list from your inventory sources.

1. Select the **Prompt on launch** checkbox for variables so that you can change the extension at runtime.
1. Select **Save**.

### Disable Azure Arc VM extensions

This template is responsible for disabling an Azure Arc-enabled server VM extension on the hosts that you identify. To create the template:

1. On the right menu, select **Templates**.
1. Select **Add**.
1. Select **Add job template**, and then complete the fields of the form as follows:

   - **Name**: Enter **Content Lab - Disable Arc Extension**.

   - **Job Type**: Select **Run**.

   - **Inventory**: Enter **localhost**.

   - **Project**: Enter **Content Lab - Azure Infrastructure Configuration Collection**.

   - **Playbook**: Enter `playbook_disable_arc-extension.yml`.

   - **Credentials**: Enter your Azure Resource Manager credential.

   - **Variables**: Enter the following code:

     ```bash
     ---
      resource_group: <your_resource_group>
      region: <your_region>
      arc_hosts:
      <first_arc_host>
      <second_arc_host>
      extension: microsoft_defender
     ```

     > [!NOTE]
     > Change `<your_resource_group>`, `<your_region>`, `<first_arc_host>`, and `<second_arc_host>` to match the names of your Azure resources. If you have a large number of Azure Arc hosts, use Jinja2 formatting to extract the list from your inventory sources.

1. Select the **Prompt on launch** checkbox for variables so that you can change the extension at runtime.
1. Select **Save**.

### Run the automation

Now that you have the job templates created, you can enable or disable Azure Arc extensions by simply changing the name of the `extension` variable. Azure Arc extensions are mapped in the `arc` role in [this file](https://github.com/ansible-content-lab/azure.infrastructure_config_demos/blob/main/roles/arc/defaults/main.yml).

When you select the launch 🚀 icon, the template asks you to confirm that the variables are accurate. For example, to enable the Microsoft Defender extension, ensure that the extension variable is set to `microsoft_defender`. Then, select **Next** > **Launch** to run the template.

:::image type="content" source="media/manage-vm-extensions-ansible/launch-extension.png" alt-text="Screenshot of the window to launch the Azure Arc extension.":::

If no errors are reported, the extension is enabled and active on the applicable servers after a short period of time. You can then proceed to enable (or disable) other extensions by changing the extension variable in the template.

## Related content

- You can deploy, manage, and remove VM extensions by using [Azure PowerShell](manage-vm-extensions-powershell.md), the [Azure portal](manage-vm-extensions-portal.md), or the [Azure CLI](manage-vm-extensions-cli.md).
- You can find troubleshooting information in the [guide for troubleshooting VM extensions](troubleshoot-vm-extensions.md).
