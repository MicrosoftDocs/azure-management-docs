---
title: Quickstart - Connect hybrid machine with Azure Arc-enabled servers
description: In this quickstart, you connect and register a hybrid machine with Azure Arc-enabled servers.
ms.topic: quickstart
ms.date: 04/23/2025
ms.custom: mode-other
---

# Quickstart: Connect hybrid machines with Azure Arc-enabled servers

Get started with [Azure Arc-enabled servers](../overview.md) to manage and govern your Windows and Linux machines hosted across on-premises, edge, and multicloud environments.

In this quickstart, you'll deploy and configure the Azure Connected Machine agent on a Windows or Linux machine hosted outside of Azure, so that the machine can be managed through Azure Arc-enabled servers. 

While you can repeat the steps in this article as needed to onboard additional machines, we also provide other options for deploying the agent, including several methods designed to onboard machines at scale. For more information, see [Azure Connected Machine agent deployment options](../deployment-options.md).

> [!TIP]
> If you prefer to try out things in a sample/practice experience, get started quickly with [Azure Arc Jumpstart](https://azurearcjumpstart.com/azure_arc_jumpstart/azure_arc_servers).

## Prerequisites

* An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Administrator permissions to install and configure the Connected Machine agent. 
  * Linux—Use the root account. 
  * Windows—Use an account that's a member of the Local Administrators group.
* Review the [Connected Machine agent prerequisites](../prerequisites.md) and verify the following requirements:
  * These [resource providers are registered](../prerequisites.md#azure-resource-providers) on your subscription:
    * Microsoft.HybridCompute
    * Microsoft.GuestConfiguration
    * Microsoft.HybridConnectivity
    * Microsoft.AzureArcData  
  * Your target machine is running a supported [operating system](../prerequisites.md#supported-operating-systems).
  * Your account has the [required Azure built-in roles](../prerequisites.md#required-permissions).
  * The machine is in a [supported region](../overview.md#supported-regions).
  * The Linux hostname or Windows computer name doesn't use a [reserved word or trademark](/azure/azure-resource-manager/templates/error-reserved-resource-name).
  * If the machine connects through a firewall or proxy server to communicate over the Internet, make sure the URLs [listed](../network-requirements.md#urls) aren't blocked.

## Generate installation script

Use the Azure portal to create a script that automates the agent download and installation and establishes the connection with Azure Arc. You'll install this script, in a later step, to the hybrid machine you want to onboard to Azure Arc.

<!--1. Launch the Azure Arc service in the Azure portal by searching for and selecting **Servers - Azure Arc**.

   :::image type="content" source="media/quick-enable-hybrid-vm/search-machines.png" alt-text="Search for Azure Arc-enabled servers in the Azure portal.":::

1. On the **Servers - Azure Arc** page, select **Add** near the upper left.-->

1. [Go to the Azure portal page for adding servers with Azure Arc](https://portal.azure.com/#view/Microsoft_Azure_HybridCompute/HybridVmAddBlade). Select the **Add a single server** tile, then select **Generate script**.

    :::image type="content" source="media/quick-enable-hybrid-vm/add-single-server.png" alt-text="Screenshot of Azure portal's add server page." lightbox="media/quick-enable-hybrid-vm/add-single-server.png":::

   > [!TIP]
   > In the portal, you can also reach this page by searching for and selecting "Servers - Azure Arc" and then selecting **+Add**.

1. On the **Basics** page, complete the following steps:

    1. Select the subscription and resource group where you want the machine to be managed within Azure.
    1. For **Region**, choose the Azure region in which the server's metadata will be stored.
    1. For **Operating system**, select the operating system of the server you want to connect.
    1. For **Connectivity method**, choose how the Azure Connected Machine agent should connect to the internet: 
        * **Public endpoint**
        * **Private endpoint**—Select an existing private link scope and endpoint or create a new one.
        * **Proxy server**—Enter the proxy server IP address or the name and port number that the machine will use in the format `http://<proxyURL>:<proxyport>`.

    1. Select **Next**.

1. On the **Tags** page, review the default **Physical location tags** suggested and enter a value, or specify one or more **Custom tags** to support your standards. Then select **Next**.

1. In the **Download and run script** section, complete the following steps:
   1. Review the script. If you want to make any changes, use the **Previous** button to go back and update your selections. 
   1. Select **Download** to save the script file.

## Install the agent using the script

Now that you've generated the script, the next step is to run it on the server that you want to onboard to Azure Arc. The script will download the Connected Machine agent from the Microsoft Download Center, install the agent on the server, create the Azure Arc-enabled server resource, and associate it with the agent.

Complete the following steps for the operating system of your server.

### Windows agent

1. Log in to the server.

1. Open an elevated 64-bit PowerShell command prompt.

1. Change to the folder or share that you copied the script to, then execute it on the server by running the `./OnboardingScript.ps1` script.

### Linux agent

Install the Linux agent on the target machine in one of the following ways:

* On target machines that can directly communicate to Azure, run the following command:

    ```bash
    bash ~/Install_linux_azcmagent.sh
    ```

* On target machines that communicate to Azure through a proxy server, run the following command:

    ```bash
    bash ~/Install_linux_azcmagent.sh --proxy "{proxy-url}:{proxy-port}"
    ```

## Verify the connection with Azure Arc

After you install the agent and configure it to connect to Azure Arc-enabled servers, go to the Azure portal to verify that the server has successfully connected. 

1. Go to the [Azure portal page for hybrid machines](https://aka.ms/hybridmachineportal).
   > [!TIP]
   > You can also reach this page in the portal by searching for and selecting "Machines - Azure Arc".
 
1. Confirm the machine has a connected status.

:::image type="content" source="./media/quick-enable-hybrid-vm/enabled-machine.png" alt-text="A successful machine connection." border="false":::

## Next steps

Now that you've enabled your Linux or Windows hybrid machine and successfully connected to the service, you're ready to enable Azure Policy to understand compliance in Azure.

> [!div class="nextstepaction"]
> [Create a policy assignment to identify non-compliant resources](tutorial-assign-policy-portal.md)
