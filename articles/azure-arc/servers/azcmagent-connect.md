---
title: azcmagent connect CLI reference
description: Syntax for the azcmagent connect command line tool
ms.topic: reference
ms.date: 07/29/2025
# Customer intent: "As a system administrator, I want to connect my server to Azure Arc using the command line, so that I can manage my resources consistently across cloud and on-premises environments."
---

# azcmagent connect

Connects the server to Azure Arc by creating a metadata representation of the server in Azure and associating the Azure connected machine agent with it. The command requires information about the tenant, subscription, and resource group where you want to represent the server in Azure and valid credentials with permissions to create Azure Arc-enabled server resources in that location.

## Usage

```
azcmagent connect [authentication] --subscription-id [subscription] --resource-group [resourcegroup] --location [region] [flags]
```

## Examples

Connect a server using the default login method (interactive browser or device code).

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus"
```

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "eastus" --use-device-code
```

Connect a server using a service principal.

```
azcmagent connect --subscription-id "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee" --resource-group "HybridServers" --location "australiaeast" --service-principal-id "ID" --service-principal-secret "SECRET" --tenant-id "TENANT"
```

Connect a server using a private endpoint and device code login method.

```
azcmagent connect --subscription-id "Production" --resource-group "HybridServers" --location "koreacentral" --use-device-code --private-link-scope "/subscriptions/.../Microsoft.HybridCompute/privateLinkScopes/ScopeName"
```

## Authentication options

There are four ways to provide authentication credentials to the Azure connected machine agent. Choose one authentication option and replace the `[authentication]` section in the usage syntax with the recommended flags.

### Interactive browser login (Windows-only)

This option is the default on Windows operating systems with a desktop experience. It login page opens in your default web browser. This option might be required if your organization configured conditional access policies that require you to log in from trusted machines.

No flag is required to use the interactive browser login.

### Device code login

This option generates a code that you can use to log in on a web browser on another device. This is the default option on Windows Server core editions and all Linux distributions. When you execute the connect command, you have 5 minutes to open the specified login URL on an internet-connected device and complete the login flow.

To authenticate with a device code, use the `--use-device-code` flag. If the account you're logging in with and the subscription where you're registering the server aren't in the same tenant, you must also provide the tenant ID for the subscription with `--tenant-id [tenant]`.

### Service principal with secret

Service principals allow you to authenticate non-interactively and are often used for at-scale deployments where the same script is run across multiple servers. Microsoft recommends providing service principal information via a configuration file (see `--config`) to avoid exposing the secret in any console logs. The service principal should also be dedicated for Arc onboarding and have as few permissions as possible, to limit the impact of a stolen credential.

To authenticate with a service principal using a secret, provide the service principal's application ID, secret, and tenant ID: `--service-principal-id [appid] --service-principal-secret [secret] --tenant-id [tenantid]`

### Service principal with certificate

Certificate-based authentication is a more secure way to authenticate using service principals. The agent accepts both PCKS #12 (.PFX) files and ASCII-encoded files (such as .PEM) that contain both the private and public keys. The certificate must be available on the local disk and the user running the `azcmagent` command needs read access to the file. Password-protected PFX files and encrypted PEM files are not supported.

To authenticate with a service principal using a certificate, provide the service principal's application ID, tenant ID, and path to the certificate file: `--service-principal-id [appId] --service-principal-cert [pathToPEMorPFXfile] --tenant-id [tenantid]`

For more information, see [create a service principal for RBAC with certificate-based authentication](/cli/azure/azure-cli-sp-tutorial-3).

### Access token

Access tokens can also be used for non-interactive authentication, but are short-lived and typically used by automation solutions onboarding several servers over a short period of time. You can get an access token with [Get-AzAccessToken](/powershell/module/az.accounts/get-azaccesstoken) or any other Microsoft Entra client.

To authenticate with an access token, use the `--access-token [token]` flag. If the account you're logging in with and the subscription where you're registering the server aren't in the same tenant, you must also provide the tenant ID for the subscription with `--tenant-id [tenant]`.

## Flags

`--access-token`

Specifies the Microsoft Entra access token used to create the Azure Arc-enabled server resource in Azure. For more information, see [authentication options](#authentication-options).

`--automanage-profile`

Resource ID of an Azure Automanage best practices profile that will be applied to the server once it's connected to Azure.

Sample value: /providers/Microsoft.Automanage/bestPractices/AzureBestPracticesProduction

`--cloud`

Specifies the Azure cloud instance. Must be used with the `--location` flag. If the machine is already connected to Azure Arc, the default value is the cloud to which the agent is already connected. Otherwise, the default value is "AzureCloud".

Supported values:

* AzureCloud (public regions)
* AzureUSGovernment (Azure US Government regions)
* AzureChinaCloud (Microsoft Azure operated by 21Vianet regions)

`--correlation-id`

Identifies the mechanism being used to connect the server to Azure Arc. For example, scripts generated in the Azure portal include a GUID that helps Microsoft track usage of that experience. This flag is optional and only used for telemetry purposes to improve your experience.

`--ignore-network-check`

Instructs the agent to continue onboarding even if the network check for required endpoints fails. You should only use this option if you're sure that the network check results are incorrect. In most cases, a failed network check indicates that the Azure Connected Machine agent won't function correctly on the server.

`-l`, `--location`

The Azure region to check connectivity with. If the machine is already connected to Azure Arc, the current region is selected as the default.

Sample value: westeurope

`--private-link-scope`

Specifies the resource ID of the Azure Arc private link scope to associate with the server. This flag is required if you're using private endpoints to connect the server to Azure.

`--gateway-id`
Specifies the resource ID of the Azure Arc gateway resource to use for the server connection. This flag is required if you're using [Azure Arc gateway (preview)](arc-gateway.md).

`-g`, `--resource-group`

Name of the Azure resource group where you want to create the Azure Arc-enabled server resource.

Sample value: HybridServers

`-n`, `--resource-name`

Name for the Azure Arc-enabled server resource. By default, the resource name is:

* The AWS instance ID, if the server is on AWS
* The hostname for all other machines

You can override the default name with a name of your own choosing to avoid naming conflicts. Once chosen, the name of the Azure resource can't be changed without disconnecting and re-connecting the agent.

If you want to force AWS servers to use the hostname instead of the instance ID, pass in `$(hostname)` to have the shell evaluate the current hostname and pass that in as the new resource name.

Sample value: FileServer01

`-i`, `--service-principal-id`

Specifies the application ID of the service principal used to create the Azure Arc-enabled server resource in Azure. Must be used with the  `--tenant-id` and either the `--service-principal-secret` or `--service-principal-cert` flags. For more information, see [authentication options](#authentication-options).

`--service-principal-cert`

Specifies the path to a service principal certificate file. Must be used with the `--service-principal-id` and `--tenant-id` flags. The certificate must include a private key and can be in a PKCS #12 (.PFX) or ASCII-encoded text (.PEM, .CRT) format. Password-protected PFX files are not supported. For more information, see [authentication options](#authentication-options).

`-p`, `--service-principal-secret`

Specifies the service principal secret. Must be used with the `--service-principal-id` and `--tenant-id` flags. To avoid exposing the secret in console logs, Microsoft recommended providing the service principal secret in a configuration file. For more information, see [authentication options](#authentication-options).

`-s`, `--subscription-id`

The subscription name or ID where you want to create the Azure Arc-enabled server resource.

Sample values: Production, aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee

`--tags`

Comma-delimited list of tags to apply to the Azure Arc-enabled server resource. Each tag should be specified in the format: TagName=TagValue. If the tag name or value contains a space, use single quotes around the name or value.

Sample value: Datacenter=NY3,Application=SharePoint,Owner='Shared Infrastructure Services'

`-t`, `--tenant-id`

The tenant ID for the subscription where you want to create the Azure Arc-enabled server resource. This flag is required when authenticating with a service principal. For all other authentication methods, the home tenant of the account used to authenticate with Azure is used for the resource as well. If the tenants for the account and subscription are different (guest accounts, Lighthouse), you must specify the tenant ID to clarify the tenant where the subscription is located.

`--use-device-code`

Generate a Microsoft Entra device login code that can be entered in a web browser on another computer to authenticate the agent with Azure. For more information, see [authentication options](#authentication-options).

`--user-tenant-id`

The tenant ID for the account used to connect the server to Azure. This field is required when the tenant of the onboarding account isn't the same as the desired tenant for the Azure Arc-enabled server resource.

[!INCLUDE [common-flags](includes/azcmagent-common-flags.md)]
