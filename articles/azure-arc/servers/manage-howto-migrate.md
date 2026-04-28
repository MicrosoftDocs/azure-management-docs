---
title:  How to rename Azure Arc-enabled servers and migrate across regions
description: Learn how to rename and migrate an Azure Arc-enabled server from one region to another.
ms.date: 04/27/2026
ms.topic: how-to
# Customer intent: As a system administrator, I want to migrate an Azure Arc-enabled server to another region, so that I can enhance manageability and compliance with governance requirements.
---

# How to rename Azure Arc-enabled servers and migrate across regions

You might want to rename an Azure Arc-enabled server or move it from one region to another. For example, you might want to move regions to improve manageability, for governance reasons, or because you realized the machine was originally registered in the wrong region. You might want to rename an Arc-enabled server to better reflect its purpose or naming conventions in your environment.

To migrate an Azure Arc-enabled server from one Azure region to another, or to rename it, uninstall any VM extensions, delete the resource in Azure, and recreate it with the new name or in the new region. Before you perform these steps, audit the machine to verify which VM extensions are installed.

> [!NOTE]
> While installed extensions continue to run and perform their normal operation after this procedure is complete, you won't be able to manage them. If you attempt to redeploy the extensions on the machine, you might experience unpredictable behavior.

## Rename an Azure Arc-enabled server resource

When you change the name of a Linux or Windows machine connected to Azure Arc-enabled servers, the new name isn't recognized automatically, because the resource name in Azure is immutable. As with other Azure resources, to use the new name, you must delete the resource in Azure and then recreate it.

For Azure Arc-enabled servers, before you rename the machine, remove the VM extensions:

1. List the VM extensions installed on the machine and note their configuration by using the [Azure portal](manage-vm-extensions-portal.md#list-extensions-installed), [Azure CLI](manage-vm-extensions-cli.md#list-extensions-installed), or [Azure PowerShell](manage-vm-extensions-powershell.md#list-extensions-installed).

1. Remove all VM extensions installed on the machine by using the [Azure portal](manage-vm-extensions-portal.md#remove-extensions), [Azure CLI](manage-vm-extensions-cli.md#remove-extensions), or [Azure PowerShell](manage-vm-extensions-powershell.md#remove-extensions).

1. Use the **azcmagent** tool with the [Disconnect](azcmagent-disconnect.md) parameter to disconnect the machine from Azure Arc and delete the machine resource from Azure. You can run this tool manually while signed in interactively, with a Microsoft identity [access token](/azure/active-directory/develop/access-tokens), or with a [service principal](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

    Disconnecting the machine from Azure Arc-enabled servers doesn't remove the Connected Machine agent, and you don't need to remove the agent as part of this process.

1. Re-register the Connected Machine agent with Azure Arc-enabled servers. Run the `azcmagent` tool with the [Connect](azcmagent-connect.md) parameter to complete this step. The agent defaults to using the computer's current hostname, but you can choose your own resource name by passing the `--resource-name` parameter to the connect command.

1. Redeploy the VM extensions that were originally deployed to the machine from Azure Arc-enabled servers. If you deployed the Azure Monitor for VMs (insights) agent by using an Azure Policy definition, the agents are redeployed after the next [evaluation cycle](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

## Move an Arc-enabled server to another region

> [!NOTE]
> This operation causes downtime during the migration.

1. Remove any VM extensions that are installed on the machine. To remove extensions, use the [Azure portal](manage-vm-extensions-portal.md#remove-extensions), [Azure CLI](manage-vm-extensions-cli.md#remove-extensions), or [Azure PowerShell](manage-vm-extensions-powershell.md#remove-extensions).

1. Use the **azcmagent** tool with the [Disconnect](azcmagent-disconnect.md) parameter to disconnect the machine from Azure Arc and delete the machine resource from Azure. You can do this step manually while signed in interactively by using a Microsoft identity platform [access token](/azure/active-directory/develop/access-tokens), or by using the service principal you used for onboarding (or a [new service principal that you create](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale)).

    > [!NOTE]
    > Disconnecting the machine from Azure Arc-enabled servers doesn't remove the Connected Machine agent. You don't need to remove the agent as part of this process.

1. Run the `azcmagent` tool with the [Connect](azcmagent-connect.md) parameter to re-register the Connected Machine agent with Azure Arc-enabled servers in the other region.

1. Redeploy the VM extensions that you originally deployed to the machine from Azure Arc-enabled servers.

    If you deployed the Azure Monitor agent by using an Azure Policy definition, the agent is redeployed after the next [evaluation cycle](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

## Next steps

* For troubleshooting information, see [Troubleshoot Connected Machine agent connection issues](troubleshoot-agent-onboard.md).

* Learn how to manage your machine by using [Azure Policy](/azure/governance/policy/overview) for such things as VM [guest configuration](/azure/governance/machine-configuration/overview), verifying the machine is reporting to the expected Log Analytics workspace, enabling monitoring by using [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy) policy, and much more.
