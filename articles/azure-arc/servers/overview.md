---
title: Azure Arc-enabled servers Overview
description: Learn how to use Azure Arc-enabled servers to manage servers hosted outside of Azure like an Azure resource.
ms.date: 02/05/2026
ms.topic: overview
# Customer intent: As an IT administrator managing hybrid environments, I want to connect and manage physical and virtual servers outside of Azure using Azure Arc, so that I can apply consistent governance, security, and monitoring across all my resources.
---

# What is Azure Arc-enabled servers?

Azure Arc-enabled servers lets you manage Windows and Linux physical servers and virtual machines hosted *outside* of Azure, on your corporate network or with another cloud provider. With Azure Arc, these machines that you host outside of Azure are considered [hybrid machines](https://azure.microsoft.com/resources/cloud-computing-dictionary/what-is-hybrid-cloud-computing), with a representation of each machine in Azure. You manage these hybrid machines in Azure Arc the same way you manage native Azure virtual machines.

When you connect a machine to Azure Arc, it's treated as a resource in Azure. Each connected machine has an Azure Resource ID, so you can include it in an Azure resource group along with other native Azure resources.

To connect hybrid machines to Azure, install the [Azure Connected Machine agent](agent-overview.md) on the machine. You can install the Connected Machine agent manually or at scale on multiple machines by using the [deployment method](deployment-options.md) that works best for your scenario.

> [!NOTE]
> For additional guidance regarding the different services Azure Arc offers, see [Choosing the right Azure Arc service for machines](../choose-service.md).

## Supported cloud operations

When you connect your machine to Azure Arc-enabled servers, you can perform many operational functions, just as you would with native Azure virtual machines. The following list describes some of the key supported actions for connected machines.

* **Govern**:
  * Assign [Azure machine configurations](/azure/governance/machine-configuration/overview) to audit settings inside the machine. For cost information, see the Azure Policy [pricing guide](https://azure.microsoft.com/pricing/details/azure-policy/).
* **Protect**:
  * Protect non-Azure servers by using [Microsoft Defender for Endpoint](/microsoft-365/security/defender-endpoint/microsoft-defender-endpoint), included through [Microsoft Defender for Cloud](/defender-for-cloud/defender-for-cloud-introduction), for threat detection,  vulnerability management, and to proactively monitor for potential security threats. Microsoft Defender for Cloud presents the alerts and remediation suggestions from the threats detected.
  * Use [Microsoft Sentinel](scenario-onboard-azure-sentinel.md) to collect security-related events and correlate them with other data sources.
* **Configure**:
  * Use [Azure Automation](/azure/automation/extension-based-hybrid-runbook-worker-install?tabs=windows) for frequent and time-consuming management tasks by using PowerShell and Python [runbooks](/azure/automation/automation-runbook-execution). Assess configuration changes for installed software, Microsoft services, Windows registry and files, and Linux daemons by using the Azure Monitor agent for [change tracking and inventory](/azure/automation/change-tracking/overview-monitoring-agent?tabs=win-az-vm).
  * Use [Azure Update Manager](/azure/update-manager/overview) to manage operating system updates for your Windows and Linux servers.
  * Perform post-deployment configuration and automation tasks by using supported [Arc-enabled servers VM extensions](manage-vm-extensions.md) for your non-Azure Windows or Linux machine.
* **Monitor**:
  * Monitor operating system performance and discover application components to monitor processes and dependencies with other resources by using [VM insights](/azure/azure-monitor/vm/vminsights-overview).
  * Collect other log data, such as performance data and events, from the operating system or workloads running on the machine by using the [Azure Monitor Agent](/azure/azure-monitor/agents/azure-monitor-agent-overview). This data is stored in a [Log Analytics workspace](/azure/azure-monitor/logs/log-analytics-workspace-overview) and contains properties specific to the machine, such as a Resource ID, to support [resource-context log access](/azure/azure-monitor/logs/manage-access#access-mode).

To learn more about Azure monitoring, security, and update services across hybrid and multicloud environments, watch the following video.

> [!VIDEO https://www.youtube.com/embed/mJnmXBrU1ao]

[!INCLUDE [azure-lighthouse-supported-service](~/reusable-content/ce-skilling/azure/includes/azure-lighthouse-supported-service.md)]

## Agent status

You can view the status for a connected machine in the Azure portal under **Azure Arc > Machines**.

The Connected Machine agent sends a regular heartbeat message to the service every five minutes. If the service stops receiving these heartbeat messages from a machine, the service considers that machine offline, and the status will change to **Disconnected** within 15 to 30 minutes. When the service receives a subsequent heartbeat message from the Connected Machine agent, the status automatically changes back to **Connected**.

If a machine remains disconnected for 45 days, its status might change to **Expired**. An expired machine can't be managed through Azure Arc until a server administrator disconnects and then reconnects it to Azure. The expiration date of the managed identity's credential determines the exact date upon which a machine expires. The credential is valid for up to 90 days and renews every 45 days.

If a machine receives 429 error messages or shows intermittent connection statuses, it might be an incorrectly cloned machine. For more information, see [Cloning guidelines](agent-overview.md#cloning-guidelines).

## Supported regions

For a list of supported regions with Azure Arc-enabled servers, see the [Azure products by region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/table) page.

In most cases, you should select the Azure region geographically closest to your machine's location when you create the installation script. Data at rest is stored within the Azure geography containing the region you specify, which might affect your choice of region if you have [data residency requirements](#data-residency).

## Supported environments

Azure Arc-enabled servers supports the management of physical servers and virtual machines hosted *outside* of Azure. For specific details about supported environments, see the [connected Machine agent prerequisites](prerequisites.md#supported-environments).

> [!NOTE]
> Azure Arc-enabled servers isn't designed or supported to enable management of virtual machines running in Azure.

## Service limits

There's no limit to [the number of Arc-enabled servers and VM extensions](/azure/azure-resource-manager/management/resources-without-resource-group-limit#microsofthybridcompute) you can deploy in a resource group or subscription. The standard 800 instance limit per resource group limit does apply to the [Azure Arc Private Link Scope](private-link-security.md) resource type.

## Data residency

Azure Arc-enabled servers stores customer data. By default, customer data stays within the region the customer deploys the service instance in. For regions with data residency requirements, customer data is always kept within the same region.  For example, if you register the machine with Azure Arc using the East US region, data is stored in East US.

For example, [instance metadata information](agent-overview.md#instance-metadata) about the connected machine is collected and stored in this region. This metadata includes the following information:

* Operating system name and version
* Computer name
* Computer fully qualified domain name (FQDN)
* Connected Machine agent version

## Next steps

* Before evaluating or enabling Azure Arc-enabled servers across multiple hybrid machines, review the [Connected Machine agent overview](agent-overview.md) to understand requirements, technical details about the agent, and deployment methods.
* Try out Arc-enabled servers by using the [Azure Arc Jumpstart](https://jumpstart.azure.com/azure_arc_jumpstart/azure_arc_servers).
* Review the [Planning and deployment guide](plan-at-scale-deployment.md) to plan for deploying Azure Arc-enabled servers at any scale and implement centralized management and monitoring.
* Explore the [Cloud Adoption Framework Unified hybrid and multicloud operations guide](/azure/cloud-adoption-framework/scenarios/hybrid/strategy) and [Identity and access management for Azure Arc-enabled servers](/azure/cloud-adoption-framework/scenarios/hybrid/arc-enabled-servers/eslz-identity-and-access-management).
