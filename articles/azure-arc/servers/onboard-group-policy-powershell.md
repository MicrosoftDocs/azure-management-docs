---
title: Connect machines at scale using Group Policy with a PowerShell script
description: In this article, you learn how to create a Group Policy Object to onboard Active Directory-joined Windows machines to Azure Arc-enabled servers.
ms.date: 08/22/2025
ms.topic: how-to
ms.custom: template-how-to
# Customer intent: As an IT administrator, I want to onboard Active Directory-joined Windows machines to Azure Arc-enabled servers using Group Policy and PowerShell scripts, so that I can efficiently manage and scale my server environment within Azure.
---

# Connect machines at scale using Group Policy

You can onboard Active Directory–joined Windows machines to [Azure Arc-enabled servers](overview.md) at scale using Group Policy.

First, you set up a remote share that hosts the Connected Machine agent, and then modify a script specifying the Arc-enabled server's landing zone within Azure. Next, you run  a script that generates a Group Policy Object (GPO) to onboard a group of machines to Azure Arc-enabled servers. This Group Policy Object can be applied at the site, domain, or organizational level. The assignment can also use Access Control List (ACL) and other security filtering native to Group Policy. All machines in the scope of the Group Policy will be onboarded to Azure Arc-enabled servers, so scope your GPO to only include those machines that you want to onboard to Azure Arc.

Before you get started, ensure your environment meets the [Connected Machine Agent prerequisites](prerequisites.md) and the [networking requirements for deploying Azure Arc-enabled servers](network-requirements.md). For information about supported regions and other related considerations, see [supported Azure regions](overview.md#supported-regions). To understand more about design and deployment criteria, review our [at-scale planning guide](plan-at-scale-deployment.md).

If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

[!INCLUDE [sql-server-auto-onboard](includes/sql-server-auto-onboard.md)]

## Prepare a remote share and create a service principal

The Group Policy Object, which is used to onboard Azure Arc-enabled servers, requires a remote share with the Connected Machine agent.

1. Prepare a remote share to host the Azure Connected Machine agent package for Windows and the configuration file. You need to be able to add files to the distributed location. The network share should provide Domain Controllers, and Domain Computers with Change permissions, and Domain Admins with Full Control permissions.

1. Follow the steps to [create a service principal for onboarding at scale](onboard-service-principal.md#create-a-service-principal-for-onboarding-at-scale).

    * Assign the Azure Connected Machine Onboarding role to your service principal. Limit the scope of the role to the target Azure landing zone.
    * Make a note of the Service Principal Secret; you'll need this value later.

1. Download and unzip the folder **ArcEnabledServersGroupPolicy_vX.X.X** from [https://github.com/Azure/ArcEnabledServersGroupPolicy/releases/latest/](https://github.com/Azure/ArcEnabledServersGroupPolicy/releases/latest/). This folder contains the ArcGPO project structure with the scripts `EnableAzureArc.ps1`, `DeployGPO.ps1`, and `AzureArcDeployment.psm1`. These assets are used for onboarding the machine to Azure Arc-enabled servers.

1. Download the latest version of the [Azure Connected Machine agent Windows Installer package](https://aka.ms/AzureConnectedMachineAgent) from the Microsoft Download Center and save it to the remote share.

1. Execute the deployment script `DeployGPO.ps1`, modifying the run parameters for the DomainFQDN, ReportServerFQDN, ArcRemoteShare, Service Principal secret, Service Principal Client ID, Subscription ID, Resource Group, Region, Tenant, and AgentProxy (if applicable). Details about these values can be found in the script comments.

   For example, the following command deploys the GPO to the contoso.com domain and copies the onboarding script `EnableAzureArc.ps1` to the remote share `AzureArcOnBoard` in the `Server.contoso.com` server:

   ```powershell
   .\DeployGPO.ps1 -DomainFQDN contoso.com -ReportServerFQDN Server.contoso.com -ArcRemoteShare AzureArcOnBoard -ServicePrincipalSecret $ServicePrincipalSecret -ServicePrincipalClientId $ServicePrincipalClientId -SubscriptionId $SubscriptionId -ResourceGroup $ResourceGroup -Location $Location -TenantId $TenantId [-AgentProxy $AgentProxy]
    ```

## Apply the Group Policy Object

On the Group Policy Management Console (GPMC), right-click on the desired Organizational Unit (OU) and link the GPO named **[MSFT] Azure Arc Servers (datetime)**. This GPO has a scheduled task to onboard the machines. Within 20 minutes, the GPO is replicated to the respective domain controllers. For more information about creating and managing group policy in Microsoft Entra Domain Services, see [Administer Group Policy in a Microsoft Entra Domain Services managed domain](/azure/active-directory-domain-services/manage-group-policy).

## Verify successful onboarding

After you install and configure the agent, verify that the servers in your OU were successfully connected to Azure Arc. You can do so by ensuring that they appear in the [Azure portal](https://aka.ms/hybridmachineportal) under **Azure Arc - Machines**.

> [!IMPORTANT]
> After confirming that your servers have successfully onboarded to Azure Arc, disable the Group Policy Object. Doing so prevents the PowerShell commands in the scheduled task from executing again when the system reboots or when the group policy is updated.

## Next steps

* Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
* Review connection troubleshooting information in the [Troubleshoot Connected Machine agent guide](troubleshoot-agent-onboard.md).
* Learn how to manage your machine using [Azure Policy](/azure/governance/policy/overview) for such things as [Azure Machine configuration](/azure/governance/machine-configuration/overview), verifying that the machine is reporting to the expected Log Analytics workspace, enabling monitoring with [VM insights](/azure/azure-monitor/vm/vminsights-enable-policy), and much more.
* Learn more about [Group Policy](/troubleshoot/windows-server/group-policy/group-policy-overview).
