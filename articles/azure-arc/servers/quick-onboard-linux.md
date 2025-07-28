---
title: Quickstart - Connect a Linux machine with Azure Arc-enabled servers (package-based installation)
description: In this quickstart, you connect and register a Linux machine to Azure Arc using a package-based installation method.
ms.topic: quickstart
ms.date: 07/23/2025
# Customer intent: "As an IT administrator, I want to connect and register Linux machines with Azure management tools, so that I can effectively manage and oversee my on-premises, edge, and multicloud environments."
---

# Quickstart: Connect a Linux machine with Azure Arc-enabled servers (package-based installation)

Get started with [Azure Arc-enabled servers](overview.md) to manage and govern your Linux machines hosted across on-premises, edge, and multicloud environments. Once your Linux machine is Arc-enabled, you can use Azure services on your on-premises machine, such as Azure Policy, Azure Monitor, Microsoft Defender, and Azure Update Manager.

In this quickstart, you deploy and configure the Azure Connected Machine agent on a Linux machine hosted outside of Azure. This quickstart provides a manual option to onboard to Arc-enabled servers with your package manager. If you prefer, you can [use an Azure portal onboarding script](quick-enable-hybrid-vm.md) to automate these steps. The onboarding script configures the Microsoft package repository on your machine, installs the agent using your package manager and onboards the server.

While you can repeat the steps in this article as needed to onboard additional machines, we also provide other options for deploying the agent, including several methods designed to onboard machines at scale. For more information, see [Azure Connected Machine agent deployment options](deployment-options.md).

> [!TIP]
> If you prefer to try out things in a sample/practice experience, get started quickly with [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_servers).

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Root permissions on the Linux machine to install and configure the Connected Machine agent.
- Review the [Connected Machine agent prerequisites](prerequisites.md) and verify the following requirements:
  - These [resource providers are registered](prerequisites.md#azure-resource-providers) on your subscription:
    - Microsoft.HybridCompute
    - Microsoft.GuestConfiguration
    - Microsoft.HybridConnectivity
    - Microsoft.AzureArcData  
  - Your target machine is running a supported [operating system](prerequisites.md#supported-operating-systems).
  - Your account has the [required Azure built-in roles](prerequisites.md#required-permissions).
  - The machine is in a [supported region](overview.md#supported-regions).
  - The Linux hostname or Windows computer name doesn't use a [reserved word or trademark](/azure/azure-resource-manager/templates/error-reserved-resource-name).
  - If the machine connects through a firewall or proxy server to communicate over the Internet, make sure the URLs [listed](network-requirements.md#urls) aren't blocked. You can also use [Azure Arc Gateway (preview)](/azure/azure-arc/servers/arc-gateway?tabs=cli) to reduce the number of required endpoints.

## Deploy the Connected Machine agent using package manager

Follow these steps to install the Azure Connected Machine agent by using your distribution's package manager.

1. Configure the [Microsoft package repository](/linux/packages) on your machine.

1. Install the Connected Machine agent using your package manager.

   For example, for Ubuntu 24.04, perform the following steps:

   1. Download `packages-microsoft-prod.deb`. This is the Debian package that configures your system to use the Microsoft package repository.
   1. Install the package: `sudo dpkg -i packages-microsoft-prod.deb`
   1. Install the agent: `sudo apt update && sudo apt install azcmagent`
   
1. Onboard your Linux machine to Azure by using the [azcmagent connect](azcmagent-connect.md) command:

      ```sh
   sudo azcmagent connect --resource-group "<resource_group_name>" --tenant-id "<tenant_id>" --location "<azure_region>" --subscription-id "<subscription_id>" --cloud "AzureCloud" --tags 'ArcSQLServerExtensionDeployment=Disabled'
   ```

      Adjust the parameters as needed:

   - `--tenant-id`: an Azure globally unique identifier (GUID) assigned to your organization's Azure AD tenant. To find your tenant ID, run this Azure CLI command: `az account show --query tenantId --output tsv`
   - `--subscription-id`: an Azure unique identifier (GUID) assigned to each Azure subscription. To find your subscription ID, run this Azure CLI command: `az account show --query id --output tsv`
   - `--location`: The Azure region in which to create your Arc-enabled server in Azure. The region should match or be near the actual machine location.
   - `--resource-group`: An Azure logical container that holds related resources for an Azure solution, created in the same region as your Arc-enabled server resource. To create a new resource group, run this Azure CLI command: `az group create --name <rg-name> --location <Azure-region>`
      
   - `--cloud`: Keep the default value, `AzureCloud`, unless you're using a [different Azure cloud environment](azcmagent-connect.md#flags).
   - `--tags`: Used to organize your Azure resources. Keep the tag 'ArcSQLServerExtensionDeployment=Disabled' and add any other flags if desired.

      > [!TIP]
      > You can optionally use [Azure Arc Gateway (preview)](/azure/azure-arc/servers/arc-gateway?tabs=cli#create-the-arc-gateway-resource) to reduce the number of required endpoints. If so, include `--gateway-id` and provide the ID of your gateway resource. To find this ID, run this Azure CLI command: `azcmagent gateway show`.

## Verify the connection with Azure Arc

After you install the agent, verify that the server was successfully connected to Azure Arc. To do so, run `azcmagent show` and ensure the agent status is **Connected**.

This command also provides a link directly to the Azure Arc server resource in the Azure portal.

Alternately, you can go to the [Azure portal page for hybrid machines](https://aka.ms/hybridmachineportal) and confirm that the machine has a connected status.

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="Screenshow showing a successful machine connection in the Azure portal." border="false":::

## Next steps

Now that your Linux machine is Arc-enabled, you can enable Azure services, like Microsoft Defender, Azure Monitor, Azure Policy and Microsoft Sentinel, to manage and secure your Arc-enabled machines.

- [Onboard Azure Arc-enabled servers to Microsoft Sentinel](scenario-onboard-azure-sentinel.md)
- [Deploy and configure Azure Monitor Agent using Azure Policy](deploy-ama-policy.md)
- [Connect your non-Azure machines to Microsoft Defender for Cloud](/azure/defender-for-cloud/quickstart-onboard-machines?toc=%2Fazure%2Fazure-arc%2Fservers%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fservers%2Fbreadcrumb%2Ftoc.json)
- [Tutorial: Create a policy assignment to identify noncompliant resources](tutorial-assign-policy-portal.md)
- [Tutorial: Monitor a hybrid machine with VM insights](tutorial-enable-vm-insights.md)
