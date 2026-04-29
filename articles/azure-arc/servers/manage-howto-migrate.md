---
title:  How to rename Azure Arc-enabled servers and migrate across regions
description: Learn how to rename and migrate an Azure Arc-enabled server from one region to another.
ms.date: 04/27/2026
ms.topic: how-to
# Customer intent: As a system administrator, I want to migrate an Azure Arc-enabled server to another region, so that I can enhance manageability and compliance with governance requirements.
---

# How to rename Azure Arc-enabled servers and migrate across regions

You might want to rename an Azure Arc-enabled server in Azure, or move it from one Azure region to another. For example, you might want to move regions to improve manageability, for governance reasons, or because you realized the machine was originally registered in the wrong region.

When you change the name of a Linux or Windows machine connected to Azure Arc-enabled servers, the new name isn't recognized automatically, because the resource name in Azure is immutable. As with other Azure resources, to use the new name, you must delete the resource in Azure and then recreate it.

You follow the same process for either of these processes: uninstall any VM extensions, delete the resource in Azure, and then recreate it with the new name or in the new region. Before you perform these steps, audit the machine to verify which VM extensions are installed, so that you can redeploy them with the same configuration after the resource is recreated.

> [!NOTE]
> Because your Arc-enabled server resource in Azure is deleted as part of this process, there's a temporary period of downtime until the machine is reconnected to Azure.

## Remove VM extensions

First, list the VM extensions installed on the machine and note their configuration by using the [Azure portal](manage-vm-extensions-portal.md#list-extensions-installed), [Azure CLI](manage-vm-extensions-cli.md#list-extensions-installed), or [Azure PowerShell](manage-vm-extensions-powershell.md#list-extensions-installed).

After noting the configuration of the installed extensions, remove all VM extensions installed on the machine. To remove extensions, use the [Azure portal](manage-vm-extensions-portal.md#remove-extensions), [Azure CLI](manage-vm-extensions-cli.md#remove-extensions), or [Azure PowerShell](manage-vm-extensions-powershell.md#remove-extensions).

## Disconnect from Azure Arc

Use the `azcmagent` tool with the [Disconnect](azcmagent-disconnect.md) parameter to disconnect the machine from Azure Arc and delete the machine resource from Azure. You can run this tool manually while signed in interactively, with a Microsoft identity [access token](/azure/active-directory/develop/access-tokens), or with a [service principal](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

Disconnecting the machine from Azure Arc-enabled servers doesn't remove the Connected Machine agent, and you don't need to remove the agent as part of this process.

## Reconnect to Azure Arc

Run the `azcmagent` tool with the [Connect](azcmagent-connect.md) parameter to re-register the Connected Machine agent on the machine. The agent defaults to using the computer's current hostname, but you can choose a different resource name by using the `--resource-name` parameter. Specify a region by using the `--location` parameter.

After the connection is in place and the machine is visible in Azure, redeploy any VM extensions that were originally deployed to the machine from Azure Arc-enabled servers.

If you deployed the Azure Monitor for VMs (insights) agent by using an Azure Policy definition, the agents are redeployed after the next [evaluation cycle](/azure/governance/policy/how-to/get-compliance-data#evaluation-triggers).

## Next steps

* Get tips for [troubleshooting Connected Machine agent connection issues](troubleshoot-agent-onboard.md).
* Learn how to [manage Connected Machine agent versions on Arc-enabled servers](manage-agent.md).
* Learn how to [configure Azure Connected Machine agent proxy settings](manage-agent-proxy-settings.md).
