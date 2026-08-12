---
title: Enable Essential Machine Management (preview)
description: Describes how to enable Essential Machine Management to automatically configure management for VMs in your subscription.
ms.topic: how-to
ms.date: 07/27/2026
---

# Enable Essential Machine Management (preview)

**Essential Machine Management (EMM)** simplifies the onboarding and configuration of management for Azure virtual machines (VMs) and Azure Arc-enabled servers. When you enable a subscription for Essential Machine Management, all VMs and Azure Arc-enabled servers in that subscription are automatically enrolled and configured with a curated set of management features. This configuration ensures that your machines are consistently set up for monitoring, security, and management.

## Prerequisites

- [Log Analytics workspace](/azure/azure-monitor/logs/quick-create-workspace) to collect log data from VMs.
- [Azure Monitor workspace](/azure/azure-monitor/metrics/azure-monitor-workspace-manage) to collect metrics data from VMs.
- [User assigned managed identity](/entra/identity/managed-identities-azure-resources/manage-user-assigned-managed-identities-azure-portal) as described in the [Managed identity](#managed-identity) section.

## Required permissions

### User

The user performing the enrollment must have the following roles in the subscription being enabled:

- Essential Machine Management Administrator
- Managed Identity Operator roles
- Resource Policy Contributor

If you're using a Log Analytics workspace or Azure Monitor workspace in a different subscription than the one being enabled for Essential Machine Management:

- The user account must also have the **Essential Machine Management Administrator** role in the resource group of the Log Analytics workspace or Azure Monitor workspace.
- The `Microsoft.ManagedOps` resource provider needs to be registered in the subscription of the Log Analytics workspace or Azure Monitor workspace. Use the Azure PowerShell command:  `Register-AzResourceProvider -ProviderNamespace "Microsoft.ManagedOps"`.

### Managed identity

The enrollment requires a [user assigned managed identity](/entra/identity/managed-identities-azure-resources/manage-user-assigned-managed-identities-azure-portal) with **Contributor** permission for the subscription.

The user assigned managed identity also needs to have permissions to act on the following resource providers: `Microsoft.Insights` and `Microsoft.Authorization`. For more information, see [Configure assignment restriction for user-assigned managed identities](/entra/identity/managed-identities-azure-resources/configure-managed-identities-assignment-restriction).

If you're using a Log Analytics workspace or Azure Monitor workspace in a different subscription than the one being enabled for Essential Machine Management, the managed identity must also have **Contributor** permissions in the resource group of the Log Analytics workspace or Azure Monitor workspace.

## Features enabled

Essential machine management enables a standard set of features and allows you to optionally enable additional security features.

### Essentials tier

The following features are part of the essentials tier.

| Feature | Description |
|:---|:---|
| [Azure Monitor](/azure/azure-monitor/vm/monitor-vm) | Monitors and provides insights into VM performance and health. Configures metric-based recommended alerts. |
| [Azure Update Manager](/azure/update-manager/overview) | Automates the deployment of operating system updates to VMs. |
| [Azure Machine Configuration](/azure/governance/machine-configuration/overview/01-overview-concepts) | Audits the Azure security baseline policy. |
| [Azure Change Tracking and Inventory](/azure/azure-change-tracking-inventory/overview-monitoring-agent) | Tracks changes to VM configurations and maintains an inventory of resources. |

#### Essentials tier pricing

> [!NOTE]
> During the initial phase of public preview, the Essential Machine Management features are provided at no extra charge. Logs generated from Change Tracking and Inventory incur a separate charge for both Azure Virtual Machines and Arc-enabled servers.
- For Azure Virtual Machines only, capabilities enabled by Essential Machine Management are provided at no extra charge.
- For Azure Arc-enabled servers with Windows Server Software Assurance, Windows Server PayGo, and Windows Server Extended Security Updates, capabilities enabled by Essential Machine Management are provided at no extra charge.
- For all other Arc-enabled servers, Essential Machine Management is priced at $9 per server per month once billing is enabled at a future date. An announcement and documentation update will be posted when billing begins.

### Security tier

The Essential Machine Management service provides the following security features. You can enable any combination of these features for the enrolled VMs. Features in this section might incur an extra charge.

| Feature | Description | Cost |
|:---|:---|:---|
| Foundational CSPM | Provides foundational cloud security posture management (CSPM) capabilities to assess and improve the security of your cloud resources. | No |
| Defender CSPM | Advanced cloud security posture management (CSPM) capabilities to enhance the security of your cloud resources. | Yes |
| Defender for cloud | Advanced threat protection and security management for VMs. | Yes |

## Enable a subscription

 To enable machine management for a subscription, select **Essential machine management** from the **Configuration** menu, and select **Enable**.

> [!NOTE]
> During public preview, the Azure portal is the only supported method for enabling machine management.

:::image type="content" source="./media/enrollment/machine-enrollment.png" lightbox="./media/enrollment/machine-enrollment.png" alt-text="Screenshot of Essential Machine Management screen with no subscriptions enabled.":::

<details>
<summary>Scope tab</summary>

The **Scope** tab includes the subscription that you want to enable and the managed identity.

| **Setting** | **Description** |
|:---|:---|
| **Select a subscription** | Select the subscription to enable. A list is provided with all subscriptions you have access to and the number of Azure VMs and Arc-enabled VMs in each. |
| **Required user role assignments** | Lists the required roles that your user account must be assigned to. |
| **Current user role assignments** | Lists the roles that are currently assigned to your user account. |
| **User assigned managed identity** | Select the managed identity to use for onboarding VMs in the subscription. |
| **Required identity role assignment** | Lists the required roles the managed identity must be assigned. |
| **Current identity role assignment** | Lists the roles currently assigned to the managed identity. |

</details>


<details>
<summary>Configure tab</summary>

The **Configure** tab includes the Log Analytics workspace and Azure Monitor workspace that collects data from the managed VMs.

| **Setting** | **Description** |
|:---|:---|
| **Log Analytics workspace** | Select the Log Analytics workspace to use for collecting log data from VMs. |
| **Azure Monitor workspace** | Select the Azure Monitor workspace to use for collecting metrics data from VMs. |

</details>


<details>
<summary>Security tab</summary>

The **Security** tab allows you to select additional security services for the managed VMs.

| **Setting** | **Description** |
|:---|:---|
| **Foundational CSPM** | Continuously assess your cloud environment with agentless, risk-prioritized insights. Recommended for all workloads.<br><br>This add-on incurs no additional charge.  |
| **Defender CSPM** | Continuously assess your cloud environment with agentless, risk-prioritized insights. Recommended for all workloads.<br><br>This add-on incurs an additional charge. |
| **Defender for cloud** | Comprehensive server protection with integrated endpoint detection and response (EDR), vulnerability management, file integrity monitoring, and advanced threat detection. Recommended for business-critical workloads.<br><br>This add-on incurs an additional charge. |

</details>

## Existing VMs

When you enable Essential Machine Management for each subscription, it automatically onboards all Azure VMs and Azure Arc-enabled servers in that subscription. Once enabled, any VMs you add to the subscription are enrolled and configured with the selected features. The following behavior applies to existing VMs in the subscription when you enable Essential Machine Management:

- Existing services retain their configuration. For example, if a VM is already using Update Management with a maintenance schedule, it still follows that maintenance schedule.
- After you enable the subscription, [remediation tasks](/azure/governance/policy/how-to/remediate-resources) are created to enable the selected service for all existing VMs in the subscription.

> [!WARNING]
> Use caution with the public preview if you have existing VMs with Change Tracking enabled. In this case, an additional Change Tracking DCR is created and associated with the VM. Since Change Tracking supports only a single DCR though, either DCR could be assigned. If you want to use the ManagedOps DCR, remove the existing DCR.

## Excluding VMs

There is currently no ability to exclude VMs in the enabled subscription. All VMs in the subscription are onboarded and configured with the selected features.

## Disable a subscription

Disable a subscription by selecting it and then select **Offboard**. When you disable a subscription, any VMs added to that subscription are no longer configured with the selected management features. The configuration isn't changed for existing VMs though. They will continue to be managed with the existing features until you manually remove them.

> [!WARNING]
> When you disable a subscription, machines in that subscription no longer use consolidated pricing. Pricing for these machines reverts to standard pricing for each individual service, which most likely increases your costs. Ensure that you disable any unneeded services on existing VMs to avoid extra charges.

## Troubleshooting

For help resolving common issues with Essential Machine Management, see [Troubleshoot Essential Machine Management (preview)](./troubleshoot.md). This article also identifies the objects created during enrollment and how to verify their creation.

## Detailed configuration

The following table describes the specific configuration applied to each VM when you enable Essential Machine Management.

| Feature | Configuration details |
|:---|:---|
| Azure Monitor | - Installs Azure Monitor agent<br>- Collects standard set of performance counters.<br>- Configures metric-based recommended alerts |
| Azure Update Manager |- Installs extension (`Microsoft.CPlat.Core.LinuxPatchExtension` or `Microsoft.CPlat.Core.WindowsPatchExtension`)<br>- [Periodic assessment](/azure/update-manager/assessment-options#periodic-assessment) enabled. |
| Azure Machine Configuration |- Installs extension (`Microsoft.GuestConfiguration.ConfigurationforLinux` or `Microsoft.GuestConfiguration.ConfigurationforWindows`)<br>- Applies the [Linux security baseline](/azure/governance/policy/samples/guest-configuration-baseline-linux) and [Windows security baseline](/azure/governance/policy/samples/guest-configuration-baseline-windows) in **Audit only** mode. |
| Azure Change Tracking and Inventory | - Installs extension (`Microsoft.Azure.ChangeTrackingAndInventory.<br>ChangeTracking-Windows` or `Microsoft.Azure.ChangeTrackingAndInventory.ChangeTracking-Linux`)<br>- Uses Log Analytics workspace specified in onboarding.<br>- Collects basic files and registry keys. |
| [Defender CSPM](/azure/defender-for-cloud/concept-cloud-security-posture-management#cspm-plans) | - All settings on by default. |

## Next steps

- [Troubleshoot Essential Machine Management](./troubleshoot.md).
