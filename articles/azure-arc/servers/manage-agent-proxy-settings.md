---
title:  Manage and maintain Azure Connected Machine agent proxy settings
description: This article describes proxy setting management tasks for the Azure Connected Machine agent.
ms.date: 03/24/2026
ms.topic: how-to
# Customer intent: As a system administrator, I want to manage the proxy settings of the Azure Connected Machine agent, so that I can ensure optimal connectivity of my Azure-connected servers.
---

# Manage and maintain Azure Connected Machine agent proxy settings

To configure the agent to communicate to the service through a proxy server or to remove this configuration after deployment, use one of the methods described here. The agent communicates outbound using the HTTP protocol under this scenario.

Proxy settings can be configured using the `azcmagent config` command or system environment variables. If a proxy server is specified in both the agent configuration and system environment variables, the agent configuration takes precedence and becomes the effective setting. Use `azcmagent show` to view the effective proxy configuration for the agent.

> [!NOTE]
> Azure Arc-enabled servers doesn't support using [Log Analytics gateway](/azure/azure-monitor/agents/gateway) as a proxy for the Connected Machine agent.

### Agent-specific proxy configuration

Agent-specific proxy configuration is the preferred way of configuring proxy server settings. This method is available starting with version 1.13 of the Azure Connected Machine agent. Using agent-specific proxy configuration helps prevent the proxy settings for the Azure Connected Machine agent from interfering with other applications on your system.

> [!NOTE]
> Some extensions deployed to Azure Arc-enabled servers don't inherit the agent-specific proxy configuration. For guidance on configuring proxy settings for extensions, see the documentation for each extension you deploy.

To configure the agent to communicate through a proxy server, run the following command:

```bash
azcmagent config set proxy.url "http://ProxyServerFQDN:port"
```

You can use an IP address or simple hostname in place of the FQDN if your network requires it. If your proxy server runs on port 80, you may omit ":80" at the end.

To check if a proxy server URL is configured in the agent settings, run the following command:

```bash
azcmagent config get proxy.url
```

To stop the agent from communicating through a proxy server, run the following command:

```bash
azcmagent config clear proxy.url
```

You don't need to restart any services when reconfiguring the proxy settings with the `azcmagent config` command.

### Proxy bypass for private endpoints

Starting with agent version 1.15, you can also specify services which should **not** use the specified proxy server. This can help with split-network designs and private endpoint scenarios where you want Microsoft Entra ID and Azure Resource Manager traffic to go through your proxy server to public endpoints, but you want Azure Arc traffic to skip the proxy and communicate with a private IP address on your network.

The proxy bypass feature doesn't require you to enter specific URLs to bypass. Instead, you provide the name of any services that shouldn't use the proxy server. The location parameter refers to the Azure region of the Arc-enabled server.

Proxy bypass value when set to `ArcData` only bypasses the traffic of the Azure extension for SQL Server and not the Arc agent.

| Proxy bypass value | Affected endpoints |
| --------------------- | ------------------ |
| `AAD` | `login.windows.net`</br>`login.microsoftonline.com`</br> `pas.windows.net` |
| `ARM` | `management.azure.com` |
| `AMA` | `global.handler.control.monitor.azure.com`</br>`<virtual-machine-region-name>.handler.control.monitor.azure.com`</br> `<log-analytics-workspace-id>.ods.opinsights.azure.com`</br>`management.azure.com`</br>`<virtual-machine-region-name>.monitoring.azure.com`</br>`<data-collection-endpoint>.<virtual-machine-region-name>.ingest.monitor.azure.com` |
| `Arc` | `his.arc.azure.com`</br>`guestconfiguration.azure.com` |
| `ArcData` <sup>1</sup> | `*.<region>.arcdataservices.com`|

<sup>1</sup> The proxy bypass value `ArcData` is available starting with Azure Connected Machine agent version 1.36 and Azure Extension for SQL Server version 1.1.2504.99. Earlier versions include the SQL Server enabled by Azure Arc endpoints in the "Arc" proxy bypass value.

To send Microsoft Entra ID and Azure Resource Manager traffic through a proxy server, but skip the proxy for Azure Arc traffic, run the following command:

```bash
azcmagent config set proxy.url "http://ProxyServerFQDN:port"
azcmagent config set proxy.bypass "Arc"
```

To provide a list of services, separate the service names by commas:

```bash
azcmagent config set proxy.bypass "ARM,Arc"
```

To clear the proxy bypass, run the following command:

```bash
azcmagent config clear proxy.bypass
```

You can view the effective proxy server and proxy bypass configuration by running `azcmagent show`.

### Windows environment variables

On Windows, the Azure Connected Machine agent first checks the `proxy.url` agent configuration property (starting with agent version 1.13), and then the system-wide `HTTPS_PROXY` environment variable, to determine which proxy server to use. If both are empty, no proxy server is used, even if the default Windows system-wide proxy setting is configured.

We recommend using the agent-specific proxy configuration instead of the system environment variable.

To set the proxy server environment variable, run the following commands:

```powershell
# If a proxy server is needed, execute these commands with the proxy URL and port.
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", "http://ProxyServerFQDN:port", "Machine")
$env:HTTPS_PROXY = [System.Environment]::GetEnvironmentVariable("HTTPS_PROXY", "Machine")
# For the changes to take effect, the agent services need to be restarted after the proxy environment variable is set.
Restart-Service -Name himds, ExtensionService, GCArcService
```

To configure the agent to stop communicating through a proxy server, run the following commands:

```powershell
[Environment]::SetEnvironmentVariable("HTTPS_PROXY", $null, "Machine")
$env:HTTPS_PROXY = [System.Environment]::GetEnvironmentVariable("HTTPS_PROXY", "Machine")
# For the changes to take effect, the agent services need to be restarted after the proxy environment variable removed.
Restart-Service -Name himds, ExtensionService, GCArcService
```

### Linux environment variables

On Linux, the Azure Connected Machine agent first checks the `proxy.url` agent configuration property (starting with agent version 1.13), and then the `HTTPS_PROXY` environment variable set for the himds, GC_Ext, and GCArcService daemons. An included script configures systemd's default proxy settings so that the Azure Connected Machine agent and all other services on the machine use a specified proxy server.

To configure the agent to communicate through a proxy server, run the following command:

```bash
sudo /opt/azcmagent/bin/azcmagent_proxy add "http://ProxyServerFQDN:port"
```

To remove the environment variable, run the following command:

```bash
sudo /opt/azcmagent/bin/azcmagent_proxy remove
```

### Migrate from environment variables to agent-specific proxy configuration

If you're already using environment variables to configure the proxy server for the Azure Connected Machine agent, and you want to migrate to the agent-specific proxy configuration based on local agent settings, follow these steps:

1. [Upgrade the Azure Connected Machine agent](manage-agent.md) to the latest version.

1. Configure the agent with your proxy server information by running `azcmagent config set proxy.url "http://ProxyServerFQDN:port"`.

1. Remove the unused environment variables by following the steps for [Windows](#windows-environment-variables) or [Linux](#linux-environment-variables).


