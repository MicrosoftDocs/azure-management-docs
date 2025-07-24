---
title: Quickstart - Connect a Linux machine with Azure Arc-enabled servers (package-based installation)
description: In this quickstart, you connect and register a Linux machine to Azure Arc using a package-based installation method.
ms.topic: quickstart
ms.date: 07/23/2025
# Customer intent: "As an IT administrator, I want to connect and register Linux machines with Azure management tools, so that I can effectively manage and oversee my on-premises, edge, and multicloud environments."
---

# Quickstart: Connect a Linux machine with Azure Arc-enabled servers (package-based installation)

Get started with [Azure Arc-enabled servers](overview.md) to manage and govern your Linux machines hosted across on-premises, edge, and multicloud environments. Once your Linux machine is Arc-enabled, you can use Azure services on your on-premises machine, such as Azure Policy, Azure Monitor, Microsoft Defender and Azure Update Manager.

In this quickstart, you'll deploy and configure the Azure Connected Machine agent on a Linux machine hosted outside of Azure. This is a manual option to onboard to Arc Server with your package manager. Another onboarding option is to use the installation script that automates these steps. The installation script will configure the Microsoft package repository on your machine and install the agent using your package manager.

While you can repeat the steps in this article as needed to onboard additional machines, we also provide other options for deploying the agent, including several methods designed to onboard machines at scale. For more information, see [Azure Connected Machine agent deployment options](deployment-options.md).

> [!TIP]
> If you prefer to try out things in a sample/practice experience, get started quickly with Azure Arc Jumpstart.

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
  - If the machine connects through a firewall or proxy server to communicate over the Internet, make sure the URLs [listed](network-requirements.md#urls) aren't blocked.

## Package-based installation of the Connected Machine agent

Follow these steps to install the Azure Connected Machine agent by using your distribution's package manager:

1. Configure the Microsoft package repository on your machine.

1. Install the agent using your package manager.

   For example, for Ubuntu 24.04, perform the following steps:

   1. Download packages-microsoft-prod.deb. This is the debian package that configures your system to use the Microsoft package repository.
   1. Install the package: sudo dpkg -i packages-microsoft-prod.deb
   1. Install the agent: sudo apt update && sudo apt install azcmagent

   Repository configuration packages:

   - Ubuntu 24.04 azcmagent package
   - RHEL, Rocky, Oracle, and Alma 9 azcmagent package
   - SLES 15 azcmagent package
   - Amazon Linux 2023 azcmagent package
   - Debian12 azcmagent package

   Other supported Linux packages are available at Microsoft Package Repository.

1. Onboard your Linux machine to Azure by using the following command:

   ```sh
   sudo azcmagent connect --resource-group "<resource_group_name>" --tenant-id "<tenant_id>" --location "<azure_region>" --subscription-id "<subscription_id>" --cloud "AzureCloud" --gateway-id "<gatewayId>" --tags 'ArcSQLServerExtensionDeployment=Disabled'
   ```

   Parameters Description:

   1. Tenant ID – an Azure globally unique identifier (GUID) assigned to your organization's Azure AD tenant. To find your Tenant ID, run this Azure CLI command: az account show --query tenantId --output tsv
   1. Subscription ID – an Azure unique identifier (GUID) assigned to each Azure subscription. To find your Sub ID, run this Azure CLI command: az account show --query id --output tsv
   1. Location – The Azure region for your Arc machine location. This should match or be near the actual machine location.
   1. Resource group - Azure logical container that holds related resources for an Azure solution. The location should be the same as your Arc machine. To create a new resource group, run this Azure CLI command: az group create --name \<rg-name\> --location \<Azure-region\>
   1. Cloud – Keep the default value, AzureCloud.
   1. Gateway ID – Optional; Arc Gateway resource reduces the number of required Arc-enabled Server URLs to allow if you have a proxy or firewall.
   1. Tags – Organize your Azure resources as categories defined by tags. Please keep the tag 'ArcSQLServerExtensionDeployment=Disabled'.

   For more information about this command, see azcmagent-connect CLI.

## Verify the connection with Azure Arc

After you install the agent and configure it to connect to Azure Arc-enabled servers, you should verify that the server has successfully connected. You can do this with command line or with Azure Portal.

1. Check that the Arc agent status of the Arc-enabled machine is "Connected". The show command also provides a link directly to the Azure Arc server resource in Azure Portal:

   ```bash
   azcmagent show
   ```

1. Go to the Azure portal page for Azure Arc machines.

   > [!TIP]
   > You can also reach this page in the portal by searching for and selecting "Machines - Azure Arc".

1. Confirm the machine has a connected status.

## Next steps

Now that you've Arc-enabled your Linux machine, you can enable Azure services, like Microsoft Defender, Azure Monitor, Azure Policy and Microsoft Sentinel, to manage and secure your Arc-enabled machines.

Onboard Azure Arc-enabled servers to Microsoft Sentinel
Deploy and configure Azure Monitor Agent using Azure Policy
Connect your non-Azure machines to Microsoft Defender for Cloud
Tutorial: Create a policy assignment to identify non-compliant resources
Tutorial: Monitor a hybrid machine with VM insights
