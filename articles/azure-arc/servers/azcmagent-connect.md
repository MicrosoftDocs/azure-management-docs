---
title: CLI reference for `azcmagent connect`
description: Syntax for the `azcmagent connect` command line tool
ms.topic: reference
ms.date: 11/10/2026
ai-usage: ai-assisted
# Customer intent: "As a system administrator, I want to connect my server to Azure Arc using the command line, so that I can manage my resources consistently across cloud and on-premises environments."
---

# `azcmagent connect`

Connects the server to Azure Arc by creating a metadata representation of the server in Azure and associating the Azure connected machine agent with it. The command requires information about the tenant, subscription, and resource group where you want to represent the server in Azure and valid credentials with permissions to create Azure Arc-enabled server resources in that location.

To restore a server's connection to an existing Azure Arc resource, see [`azcmagent reconnect`](azcmagent-reconnect.md) (preview, agent version 1.70 or later).

## Usage

```
azcmagent connect [authentication] --subscription-id [subscription] --resource-group [resourcegroup] --location [region] [flags]
```

## Examples

Connect a server using the default login method (interactive browser or device code):

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus"
```

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus" --use-device-code
```

Connect a server using a service principal:

```
azcmagent connect --subscription-id "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" --resource-group "HybridServers" --location "australiaeast" --service-principal-id "ID" --service-principal-secret "SECRET" --tenant-id "TENANT"
```

Connect a server using a private endpoint and device code login method:

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "koreacentral" --use-device-code --private-link-scope "/subscriptions/.../Microsoft.HybridCompute/privateLinkScopes/ScopeName"
```

Connect a server using Azure CLI credentials (requires Azure Connected Machine agent version 1.59 or later):

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus" --use-azcli
```

```
azcmagent connect --subscription-id "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" --resource-group "HybridServers" --location "eastus" --use-azcli --tenant-id "bbbbbbbb-cccc-dddd-eeee-ffffffffffff"
```

## Authentication options

<!-- Preserve published links to the former authentication method sections. -->
<a id="interactive-browser-login-windows-only"></a>
<a id="device-code-login"></a>
<a id="service-principal-with-secret"></a>
<a id="service-principal-with-certificate"></a>
<a id="access-token"></a>
<a id="azure-cli-credentials"></a>

Choose a sign-in method and its flags from [Authenticate the azcmagent CLI](azcmagent-authentication.md). Use an account or service principal with the **[Azure Connected Machine Onboarding](security-identity-authorization.md#rbac-roles)** role, or equivalent permission to create servers in the target resource group.

## Flags

This command supports [Common flags](azcmagent.md#common-flags), [Authentication flags](azcmagent-authentication.md#authentication-flags), and the flags listed in this section.

`--automanage-profile`

Resource ID of an Azure Automanage best practices profile that will be applied to the server once it's connected to Azure.

Sample value: `/providers/Microsoft.Automanage/bestPractices/AzureBestPracticesProduction`

`--cloud`

Specifies the Azure cloud instance. Must be used with the `--location` flag. If the machine is already connected to Azure Arc, the default value is the cloud to which the agent is already connected. Otherwise, the default value is `AzureCloud`.

Supported values:

* `AzureCloud` (public regions)
* `AzureUSGovernment` (Azure US Government regions)
* `AzureChinaCloud` (Microsoft Azure operated by 21Vianet regions)

`--correlation-id`

Identifies the mechanism being used to connect the server to Azure Arc. For example, scripts generated in the Azure portal include a GUID that helps Microsoft track usage of that experience. This flag is optional and only used for telemetry purposes to improve your experience.

`--enable-automatic-upgrade`

Instructs the agent to automatically upgrade itself when a new version is available. If this flag isn't specified when the machine is connected, automatic upgrades are disabled unless another [automatic upgrade method](automatic-agent-upgrade.md) is configured later.

The `--enable-automatic-upgrade` flag is currently in public preview.

`--ignore-network-check`

Instructs the agent to continue onboarding even if the network check for required endpoints fails. You should only use this option if you're sure that the network check results are incorrect. In most cases, a failed network check indicates that the Azure Connected Machine agent won't function correctly on the server.

`-l`, `--location`

The Azure region to check connectivity with. If the machine is already connected to Azure Arc, the current region is selected as the default.

Sample value: `westeurope`

`--private-link-scope`

Specifies the resource ID of the Azure Arc private link scope to associate with the server. This flag is required if you're using private endpoints to connect the server to Azure.

`--gateway-id`
Specifies the resource ID of the Azure Arc gateway resource to use for the server connection. This flag is required if you're using [Azure Arc gateway](arc-gateway.md).

`-g`, `--resource-group`

Name of the Azure resource group where you want to create the Azure Arc-enabled server resource.

Sample value: `HybridServers`

`-n`, `--resource-name`

Name for the Azure Arc-enabled server resource. By default, the resource name is:

* The AWS instance ID, if the server is on AWS
* The hostname for all other machines

You can override the default name with a name of your own choosing to avoid naming conflicts. Once chosen, the name of the Azure resource can't be changed without disconnecting and reconnecting the agent.

If you want to force AWS servers to use the hostname instead of the instance ID, pass in `$(hostname)` to have the shell evaluate the current hostname and pass that in as the new resource name.

Sample value: `FileServer01`

`-s`, `--subscription-id`

The subscription name or ID where you want to create the Azure Arc-enabled server resource.

Sample values: `Production`, `aaaa0a0a-bb1b-cc2c-dd3d-eeeeee4e4e4e`

`--tags`

Comma-delimited list of tags to apply to the Azure Arc-enabled server resource. Each tag should be specified in the format: `TagName=TagValue`. If the tag name or value contains a space, use single quotes around the name or value.

Sample value: `Datacenter=NY3,Application=SharePoint,Owner='Shared Infrastructure Services'`

`-t`, `--tenant-id`

The tenant ID for the subscription where you want to create the Azure Arc-enabled server resource. This flag is required when authenticating with a service principal or whenever `--user-tenant-id` is supplied. Otherwise, the home tenant of the account used to authenticate with Azure is used for the resource as well. If the tenants for the account and subscription are different (such as when using guest accounts or [Azure Lighthouse](/azure/lighthouse)), you must specify the tenant ID to clarify the tenant where the subscription is located.
