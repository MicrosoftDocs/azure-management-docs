---
title: CLI reference for `azcmagent disconnect`
description: Syntax for the `azcmagent disconnect` command line tool
ms.topic: reference
ms.date: 11/10/2026
ai-usage: ai-assisted
# Customer intent: As a system administrator, I want to disconnect an Azure Arc-enabled server using the command line, so that I can manage the server's configuration and remove it from the cloud effectively without affecting local resources.
---

# `azcmagent disconnect`

Deletes the Azure Arc-enabled server resource in the cloud and resets the configuration of the local agent. For detailed information on removing extensions and disconnecting and uninstalling the agent, see [Uninstall the Azure Connected Machine agent from Arc-enabled servers](uninstall-agent.md).

> [!CAUTION]
> When disconnecting the agent from Azure Local VMs enabled by Azure Arc, use only the `azcmagent disconnect --force-local-only` command. Using the command without the `--force-local-only` flag can cause your Azure Local VM to be deleted both from Azure and on-premises.

## Usage

```
azcmagent disconnect [authentication] [flags]
```

## Examples

Disconnect a server using the default login method (interactive browser or device code):

```
azcmagent disconnect
```

Disconnect a server using a service principal:

```
azcmagent disconnect --service-principal-id "ID" --service-principal-secret "SECRET"
```

Disconnect a server using Azure CLI credentials (requires Azure Connected Machine agent version 1.59 or later):

```
azcmagent disconnect --use-azcli
```

Disconnect a server if the corresponding resource in Azure has already been deleted.

```
azcmagent disconnect --force-local-only
```

## Authentication options

<!-- Preserve published links to the former authentication method sections. -->
<a id="interactive-browser-login-windows-only"></a>
<a id="device-code-login"></a>
<a id="service-principal-with-secret"></a>
<a id="service-principal-with-certificate"></a>
<a id="access-token"></a>
<a id="azure-cli-credentials"></a>

Choose a sign-in method and its flags from [Authenticate the azcmagent CLI](azcmagent-authentication.md). Use an account or service principal with the **[Azure Connected Machine Resource Administrator](security-identity-authorization.md#rbac-roles)** role on the existing server resource, or equivalent permission to delete it.

## Flags

This command supports [Common flags](azcmagent.md#common-flags), [Authentication flags](azcmagent-authentication.md#authentication-flags), and the flag listed in this section.

`-f`, `--force-local-only`

Disconnects the server without deleting the resource in Azure. Primarily used if the Azure resource was deleted and the local agent configuration needs to be cleaned up.
