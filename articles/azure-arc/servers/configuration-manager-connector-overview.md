---
title: Configuration Manager connector for Azure Arc (preview)
description: Learn how the Configuration Manager connector makes on-premises machine inventory visible in Azure without installing the Azure Connected Machine agent on each managed machine.
ms.date: 10/22/2026
ms.topic: overview
ai-usage: ai-assisted
# Customer intent: As a Configuration Manager administrator, I want to view inventory for my managed machines in Azure without deploying an agent to each machine, so that I can retain Configuration Manager as my management authority.
---

# Configuration Manager connector for Azure Arc (preview)

The Configuration Manager connector runs on-premises and brings inventory from your Configuration Manager-managed machines into Azure. You select the device collections to include, and the connector uses their inventory data to represent those machines in Azure.

The connector provides connected inventory without installing the Azure Connected Machine agent on each managed machine. Configuration Manager remains the source of inventory and the management authority for those machines.

## How the connector works

The connector uses your Configuration Manager site server to make selected inventory available in Azure:

1. The Configuration Manager site server connects to Azure Arc by using the Azure Connected Machine agent.
1. You select the device collections to include through the Configuration Manager console.
1. The connector makes the selected machines and their inventory visible in Azure.

Connected inventory includes server roles, services, and applications from the data available in Configuration Manager. The managed machines don't need the Connected Machine agent for this inventory representation.

## Inventory and agent deployment

The Configuration Manager connector provides a read-only inventory representation. It doesn't transfer management of the selected machines to Azure or deploy the Connected Machine agent to them.

Use the connector when you want visibility into Configuration Manager inventory in Azure. To use Azure Arc capabilities that require an agent on each machine, use an [agent deployment option](deployment-options.md) instead. Inventory representation alone doesn't enable those capabilities.

The connector is separate from the existing Configuration Manager deployment methods:

- [Connect machines using a Configuration Manager script](onboard-configuration-manager-powershell.md) installs the Connected Machine agent on the target machines.
- [Connect machines using a Configuration Manager custom task sequence](onboard-configuration-manager-custom-task.md) installs the Connected Machine agent as part of a task sequence.

## Deploy the connector

Deploy the connector on the Configuration Manager site server, not on each managed machine. Use the Azure portal to generate the onboarding script. After completing connector setup, approve the console extension and select the device collections to include.

### Prerequisites

Before you begin, prepare the following items:

- An Azure subscription, a resource group, and the Microsoft Entra tenant ID for your deployment.
- An Azure account with permissions to connect the site server to Azure Arc and create, update, and remove connected inventory resources in the target resource group.
- Administrator access to the Configuration Manager site server and access to approve and install Configuration Manager console extensions.
- Azure Connected Machine agent version 1.70 or later on the Configuration Manager site server.
- Windows PowerShell 5.1 on the site server, with access to the Configuration Manager SMS Provider in `root\SMS`.
- Connectivity that meets the [Azure Arc-enabled servers network requirements](network-requirements.md), plus access to PowerShell Gallery and the NuGet package provider download if `Az.Accounts` isn't installed.
- A small device collection for initial validation.

> [!IMPORTANT]
> The portal-generated onboarding script disables Arc VM extension deployment on the site server and disables automatic SQL Server extension deployment through the `ArcSQLServerExtensionDeployment=Disabled` tag. This restriction is separate from the Configuration Manager console extension, which you approve and install after onboarding.

### Generate and run the onboarding script

Generate and run the connector-specific onboarding script:

1. Sign in to the [Azure portal](https://portal.azure.com), and search for and select **Azure Arc**.
1. Under **Supported environments**, select **On-premises connectors**, and then select **Create**.

   :::image type="content" source="media/configuration-manager-connector/on-premises-connectors.png" alt-text="Screenshot of the Azure Arc On-premises connectors page with the Create button and a list of connected site servers.":::

1. On **Onboard a Configuration Manager site server**, complete the **Basics** tab:
   - Select the **Subscription** and **Resource group** for the connector.
   - Select the **Region** in which to create the Azure resource.
   - Set **Operating system** to **Windows**.

   :::image type="content" source="media/configuration-manager-connector/onboard-site-server-basics.png" alt-text="Screenshot of the Basics tab for onboarding a Configuration Manager site server, showing subscription, resource group, region, and operating system fields.":::

1. Select **Next: Tags**, and optionally add physical location or custom tags.
1. Select **Download and run script**. Download or copy the generated script and save it as a `.ps1` file on the Configuration Manager site server.
1. Review the script and confirm that its subscription, resource group, tenant, and location match your intended deployment.
1. Open Windows PowerShell as Administrator on the site server. Run the saved script, and complete the Azure sign-in flow when prompted. For example, if you saved the script as `Onboard-ConfigurationManagerConnector.ps1`:

   ```powershell
   & .\Onboard-ConfigurationManagerConnector.ps1
   ```

1. Wait for the `Console extension published` message. If an error appears, resolve it before continuing; returning to the PowerShell prompt doesn't confirm that onboarding succeeded.

The generated script installs the Azure Connected Machine agent, discovers the Configuration Manager site code, connects the site server as a Configuration Manager connector, and enables automatic agent upgrade. It also installs `Az.Accounts` if needed, disables Arc VM extension deployment on the site server, and publishes the Configuration Manager console extension.

### Approve and install the console extension

After the onboarding script finishes successfully, install the published console extension:

1. In the Configuration Manager console, go to **Administration** > **Overview** > **Updates and Servicing** > **Console Extensions**.
1. Approve the Azure Arc console extension if your hierarchy settings require approval.
1. Install the extension, and then close and restart the Configuration Manager console.
1. Go to **Administration** > **Overview** > **Cloud Services** > **Azure Arc Devices** and confirm that the workspace opens.

### Select device collections

Use the Connected Inventory wizard to choose which managed machines appear in Azure:

1. In the Configuration Manager console, go to **Administration** > **Overview** > **Cloud Services** > **Azure Arc Devices**, and select **Arc Connected Devices** from the ribbon or context menu.

   :::image type="content" source="media/configuration-manager-connector/console-azure-arc-devices.png" alt-text="Screenshot of the Configuration Manager Azure Arc Devices workspace with the Arc Connected Devices ribbon button and device collection inventory status.":::

1. On **Connected Inventory**, review the inventory capabilities and Azure connection details, and then select **Next**.

   :::image type="content" source="media/configuration-manager-connector/console-connected-inventory-wizard.png" alt-text="Screenshot of the Azure Arc setup wizard's Connected Inventory page showing inventory capabilities, Azure connection details, and the Next button.":::

1. On **Choose collections**, select your test device collection. You can select more than one collection.
1. On **Review changes**, confirm the collections to include or remove, and then select **Apply**.
1. Monitor **Apply changes**, and review **Results** for success, no-change, or error information. Resolve errors before continuing.
1. Close the wizard and review the refreshed **Azure Arc Devices** workspace.

> [!IMPORTANT]
> Don't select **All Systems** (`SMS00001`) unless you intend to include every managed client in the site in connected inventory.

### Validate deployment

Confirm the site server connection and the selected inventory:

1. On the site server, run the following commands:

   ```powershell
   azcmagent show
   azcmagent config get extensions.allowlist
   azcmagent config get extensions.enabled
   ```

1. Confirm that the agent status is `Connected` and that the subscription, resource group, tenant, and location match your deployment. The extension settings should be `Allow/None` and `false`, respectively.
1. To confirm that the selected machines appear in Azure, see [View machine inventory](#view-machine-inventory).

## View machine inventory

Use the Azure portal to browse connected machines and their inventory, or Azure Resource Graph to query connectors and managed machines.

Device inventory refreshes daily and can take up to 24 hours to appear. If devices are missing, confirm the collection selection and wizard results in Configuration Manager. Also, check the subscription and filter selections in the Azure portal.

# [Azure portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Go to **Azure Arc** > **On-premises connectors**, and select your site server.

   :::image type="content" source="media/configuration-manager-connector/portal-site-server.png" alt-text="Screenshot of the Azure Arc On-premises connectors page listing a Configuration Manager site server with Connected status.":::

1. Select **Devices** to view the managed machines included in connected inventory.

   :::image type="content" source="media/configuration-manager-connector/portal-site-server-devices.png" alt-text="Screenshot of the site server's Devices page listing managed machines, operating systems, and OS versions.":::

1. Select **Machine inventory (preview)** to explore the available inventory.

   :::image type="content" source="media/configuration-manager-connector/portal-site-server-machine-inventory.png" alt-text="Screenshot of the site server's Machine inventory page showing summary cards for enrolled machines, user passwords, groups, and certificates.":::

# [Azure Resource Graph](#tab/azure-resource-graph)

Use [Azure Resource Graph Explorer](/azure/governance/resource-graph/first-query-portal) to run the following Kusto Query Language (KQL) samples. Select the subscriptions that contain your connector resources. You need read access to the resources you query; a Log Analytics workspace isn't required.

### List Configuration Manager connectors

This query lists Configuration Manager site servers registered as connectors. Use the **Resource ID** value in the queries that require `<site-server-resource-id>`.

```kusto
resources
| where type =~ 'microsoft.hybridcompute/machines'
| where ['kind'] == 'ConfigManagerConnector'
| project
    Name = name,
    ['Resource ID'] = id,
    Platform = tostring(tags.ConnectorType),
    ID = tostring(tags.ConnectorId),
    ['Resource group'] = resourceGroup,
    Location = location
| order by Name asc
```

### List managed machines for a connector

Replace `<site-server-resource-id>` with the connector's Azure resource ID.

```kusto
resources
| where type =~ 'microsoft.hybridcompute/machines'
| where tostring(tags.OnPremisesConnectorResourceId) =~ '<site-server-resource-id>'
| project
    Name = name,
    ['Operating system'] = tostring(properties.osName),
    ['OS version'] = tostring(properties.osVersion),
    ['OS type'] = tostring(properties.osType),
    Inventory = 'Enabled',
    ['Connector ID'] = tostring(tags.ConnectorId),
    ['Site server'] = tostring(tags.ConfigMgrSiteServer),
    ['Resource group'] = resourceGroup,
    Location = location
| order by Name asc
```

This query returns only machines included in connected inventory. Machines that Configuration Manager manages but aren't included in connected inventory don't have inventory representations in Azure and aren't returned by this query.

### List device collections for a connector

Replace `<site-server-resource-id>` with the connector's Azure resource ID.

```kusto
guestconfigurationresources
| where id contains '<site-server-resource-id>'
| where type =~ 'microsoft.guestconfiguration/devicecollections'
| project
    DisplayName = tostring(properties.displayName),
    Description = tostring(properties.description),
    CollectionId = tostring(properties.collectionId),
    DeviceCount = tostring(properties.deviceCount)
```

### List installed applications for a site server

Replace `<site-server-resource-id>` with the Azure resource ID of the Configuration Manager site server.

```kusto
guestconfigurationresources
| where type =~ 'microsoft.guestconfiguration/applications'
| extend vmResourceId = tolower(tostring(split(id, '/providers/Microsoft.GuestConfiguration/')[0]))
| where vmResourceId =~ '<site-server-resource-id>'
| join kind=leftouter (
    resources
    | where type in~ ('microsoft.compute/virtualmachines', 'microsoft.hybridcompute/machines')
    | project vmResourceId = tolower(id), vmName = name, vmType = type
) on vmResourceId
| project
    Machine = vmName,
    ['Machine resource ID'] = vmResourceId,
    ['Machine type'] = vmType,
    Application = tostring(properties.displayName),
    Version = tostring(properties.version),
    Publisher = tostring(properties.publisher),
    ['Install date'] = todatetime(properties.installDate)
| order by Application asc
```

### List services for a site server or managed machine

Configuration Manager-managed machines and the site server are both represented as Azure Arc-enabled servers (`Microsoft.HybridCompute/machines`), so you can use the same inventory query for either resource. Replace `<machine-resource-id>` with the Azure resource ID of the managed machine or site server.

```kusto
guestconfigurationresources
| where type =~ 'microsoft.guestconfiguration/services'
| extend MachineId = tostring(split(id, '/providers/Microsoft.GuestConfiguration/')[0])
| where MachineId =~ '<machine-resource-id>'
| project
    Service = tostring(properties.displayName),
    Status = tostring(properties.status),
    ['Start type'] = tostring(properties.startType),
    ['Service type'] = tostring(properties.serviceType),
    ['User name'] = tostring(properties.userName),
    Path = tostring(properties.pathName)
| order by Service asc
```

### List machines with an application installed

This query lists Configuration Manager-connected machines where Microsoft Edge is present in connected inventory. Replace `Microsoft Edge` with another application display name to find machines with that application installed. To limit the results to machines managed by a specific site server, add a filter for `tags.OnPremisesConnectorResourceId` in the `resources` subquery.

```kusto
guestconfigurationresources
| where type =~ 'microsoft.guestconfiguration/applications'
| where tostring(properties.displayName) =~ 'Microsoft Edge'
| extend MachineId = tostring(split(id, '/providers/Microsoft.GuestConfiguration/')[0])
| extend MachineIdLower = tolower(MachineId)
| join kind=inner (
    resources
    | where type =~ 'microsoft.hybridcompute/machines'
    | where tostring(tags['ConnectorType']) =~ 'ConfigurationManager'
    | extend MachineIdLower = tolower(id)
    | project MachineIdLower
) on MachineIdLower
| extend Machine = tostring(split(MachineId, '/')[8])
| project
    Machine,
    ['Machine resource ID'] = MachineId,
    Application = tostring(properties.displayName),
    Version = tostring(properties.version),
    Publisher = tostring(properties.publisher),
    ['Resource group'] = resourceGroup
| order by Machine asc
```

---

## Troubleshoot

Use the following steps to troubleshoot missing inventory data.

For Azure Connected Machine agent onboarding issues, see [Troubleshoot Azure Connected Machine agent onboarding](troubleshoot-agent-onboard.md).

### Inventory data doesn't appear in Azure

If connected inventory is enabled for Configuration Manager-managed machines but no inventory data appears in the Azure portal or Azure Resource Graph (ARG), confirm that the site server collects inventory data from its managed machines.

1. In the Configuration Manager console, go to **Administration** > **Client Settings** > **Default Client Settings**, and open its properties.
1. In the dialog box, select **Hardware Inventory**. Under **Device Settings**, next to **Hardware inventory classes**, select **Set Classes**.
1. In the **Hardware Inventory Classes** dialog box, expand **Services** (`Win32_Service`), and make sure **State** is selected.
1. Select **OK** in each dialog box to save your changes.
1. Go to **Assets and Compliance** > **Devices**, and select a device.
1. On the ribbon, select **Client Notification** > **Download Computer Policy**.
1. With the device still selected, select **Client Notification** > **Collect Hardware Inventory**.
1. Wait for Configuration Manager to collect the inventory data and synchronize it to Azure. Inventory refreshes daily and can take up to 24 hours to appear. Then check the Azure portal or Azure Resource Graph for the device's inventory.

## Related content

- Learn about the [Azure Connected Machine agent](agent-overview.md).
- Compare [Azure Connected Machine agent deployment options](deployment-options.md).
- Explore more [Azure Resource Graph sample queries for Azure Arc-enabled servers](resource-graph-samples.md).
