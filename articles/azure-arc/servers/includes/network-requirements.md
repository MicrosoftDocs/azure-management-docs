---
ms.service: azure-arc
ms.topic: include
ms.date: 09/23/2026
# Customer intent: "As a network administrator, I want to configure secure outbound connectivity for the Azure Connected Machine agent so that I can ensure proper communication with Azure Arc while adhering to my organization's security policies."
---

All server-based Azure Arc offerings require Azure Arc-enabled server endpoints.

### Networking configuration

The Azure Connected Machine agent for Linux and Windows securely communicates outbound to Azure Arc over TCP port 443. By default, the agent uses the default route to the internet to reach Azure services. You can optionally [configure the agent to use a proxy server](../manage-agent-proxy-settings.md) if your network requires it. Proxy servers don't make the Connected Machine agent more secure because the traffic is already encrypted.

To further secure your network connectivity to Azure Arc, instead of using public networks and proxy servers, you can implement an [Azure Arc private link scope](../private-link-security.md).

> [!NOTE]
> Azure Arc-enabled servers don't support using a [Log Analytics gateway](/azure/azure-monitor/agents/gateway) as a proxy for the Connected Machine agent. At the same time, Azure Monitor Agent supports Log Analytics gateways.

If your firewall or proxy server restricts outbound connectivity, ensure that you don't block the URLs and service tags listed in the documentation.

### Service tags

Be sure to allow access to the following service tags:

* `AzureActiveDirectory`
* `AzureTrafficManager`
* `AzureResourceManager`
* `AzureArcInfrastructure`
* `Storage`
* `AzureFrontDoor.Frontend` (required as of April 2026)
* `WindowsAdminCenter` (if you [use Windows Admin Center to manage Azure Arc-enabled servers](/windows-server/manage/windows-admin-center/azure/manage-arc-hybrid-machines))

For a list of IP addresses for each service tag and region, see the JSON file [Azure IP Ranges and Service Tags - Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519). Microsoft publishes weekly updates that contain each Azure service and the IP ranges it uses. The JSON file provides the current point-in-time list of the IP ranges that correspond to each service tag. The IP addresses are subject to change. If your firewall configuration requires IP address ranges, use the `AzureCloud` service tag to allow access to all Azure services. Don't disable security monitoring or inspection of these URLs. Allow them as you would other internet traffic.

If you filter traffic to the `AzureArcInfrastructure` service tag, you must allow traffic to the full service tag range. The ranges advertised for individual regions, for example, `AzureArcInfrastructure.AustraliaEast`, don't include the IP ranges that are used by global components of the service. The specific IP address resolved for these endpoints might change over time within the documented ranges. For this reason, using a lookup tool to identify the current IP address for a specific endpoint and allowing access to only that IP address isn't sufficient to ensure reliable access.

For more information, see [Virtual network service tags](/azure/virtual-network/service-tags-overview).

> [!IMPORTANT]
> To filter traffic by IP addresses in Azure Government or Azure operated by 21Vianet, be sure to add the IP addresses from the `AzureArcInfrastructure` service tag for the Azure public cloud, in addition to using the `AzureArcInfrastructure` service tag for your cloud. After October 28, 2025, adding the `AzureArcInfrastructure` service tag for Azure public cloud will be required. The service tags for Azure Government and Azure operated by 21Vianet will no longer be supported.

### URLs

This table lists the URLs that must be available to install and use the Connected Machine agent.

#### [Azure cloud platform](#tab/azure-cloud)

> [!NOTE]
> When you configure the Connected Machine agent to communicate with Azure through a private link, you must still access some endpoints through the internet. The **Private link capable** column in the following table shows the endpoints that you can configure with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall or proxy server for the agent to function. Network traffic routes through private endpoints if you assign a private link scope.

| Agent resource | Description | When required| Private link capable |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> | Public. |
|`packages.microsoft.com`|Used to download the Linux installation package.|Only at installation time.<sup>1</sup> | Public. |
|`login.microsoftonline.com`|Global Microsoft Entra token endpoint used during onboarding and as a fallback when a regional endpoint can't be reached.|Always.| Public. |
|`*.login.microsoft.com`|Regional Microsoft Entra token endpoints used during normal agent operation.|Always.| Public. |
|`pas.windows.net`|Microsoft Entra ID.|Always.| Public. |
|`management.azure.com`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured. |
|`*.his.arc.azure.com`|Metadata and hybrid identity services.|Always.| Private. |
|`*.guestconfiguration.azure.com`| Extension management and guest configuration services. |Always.| Private. |
|`guestnotificationservice.azure.com`, `*.guestnotificationservice.azure.com`|Notification service for extension and connectivity scenarios.|Always.| Public. |
|`azgn*.servicebus.windows.net` or `*.servicebus.windows.net`|Notification service for extension and connectivity scenarios.|Always.| Public. |
|`*.servicebus.windows.net`|For Windows Admin Center and Secure Shell (SSH) scenarios.|If you use SSH or Windows Admin Center from Azure.|Public.|
|`*.waconazure.com`|For Windows Admin Center connectivity.|If you use Windows Admin Center.|Public.|
|`dc.services.visualstudio.com`|Agent telemetry.|Optional. Not used in agent versions 1.24+.| Public. |
| `*.<region>.arcdataservices.com`<sup>2</sup> | For Azure Arc-enabled SQL Server. Sends data processing service, service telemetry, and performance monitoring to Azure. Allows Transport Layer Security (TLS) 1.2 or 1.3 only. | If you use Azure Arc-enabled SQL Server. | Public. |
| `https://<azure-keyvault-name>.vault.azure.net/`, `https://graph.microsoft.com/`<sup>2</sup>| For Microsoft Entra authentication with Azure Arc-enabled SQL Server. | If you use Azure Arc-enabled SQL Server. | Public. |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates (uses HTTP/TCP 80 and HTTPS/TCP 443). | If you use Extended Security Updates enabled by Azure Arc. Always required for automatic updates or temporarily if you download certificates manually. | Public. |
|`dls.microsoft.com`| Used by Azure Arc machines to perform license validation. | Required when you use [hotpatching](/azure/update-manager/manage-hot-patching-arc-machines), Windows Server Azure Benefits, or Windows Server pay-as-you-go billing on Azure Arc-enabled machines. | Public. |

> [!IMPORTANT]
> The Connected Machine agent normally acquires tokens from the regional Microsoft Entra endpoint returned by the Hybrid Identity Service, such as `eastus2.login.microsoft.com`. The global endpoint is a resiliency fallback and doesn't replace the requirement to allow the regional endpoint. If your firewall or proxy supports wildcard fully qualified domain name (FQDN) rules, allow `*.login.microsoft.com`. Otherwise, allow `<region>.login.microsoft.com` for every Azure region where your Arc-enabled servers are registered.

<sup>1</sup> Access to this URL is also needed when updates are performed automatically.

<sup>2</sup> For details about what information is collected and sent, review [Data collection and reporting for SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/data-collection).

For extension versions up to and including February 13, 2024, use `san-af-<region>-prod.azurewebsites.net`. Beginning March 12, 2024, both Azure Arc data processing and Azure Arc data telemetry use `*.<region>.arcdataservices.com`.

> [!NOTE]
> To translate the `*.servicebus.windows.net` wildcard into specific endpoints, use the command `\GET https://guestnotificationservice.azure.com/urls/allowlist?api-version=2020-01-01&location=<region>`. Within this command, you must specify the region for the `<region>` placeholder. These endpoints might change periodically.

[!INCLUDE [arc-region-note](../../includes/arc-region-note.md)]

#### [Azure Government](#tab/azure-government)

> [!NOTE]
> When you configure the Connected Machine agent to communicate with Azure through a private link, you must still access some endpoints through the internet. The **Endpoint used with private link** column in the following table shows the endpoints that you can configure with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall or proxy server for the agent to function.

| Agent resource | Description | When required| Endpoint used with private link |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> | Public. |
|`packages.microsoft.com`|Used to download the Linux installation package.|Only at installation time.<sup>1</sup> | Public. |
|`login.microsoftonline.us`|Microsoft Entra ID.|Always.| Public. |
|`pasff.usgovcloudapi.net`|Microsoft Entra ID.|Always.| Public. |
|`management.usgovcloudapi.net`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured. |
|`*.his.arc.azure.us`|Metadata and hybrid identity services.|Always.| Private. |
|`*.guestconfiguration.azure.us`| Extension management and guest configuration services. |Always.| Private. |
|`*.blob.core.usgovcloudapi.net`|Download source for Azure Arc-enabled servers extensions.|Always, except when you use private endpoints.| Not used when a private link is configured. |
|`dc.applicationinsights.us`|Agent telemetry.|Optional. Not used in agent versions 1.24+.| Public. |
| `*.<region>.arcdataservices.azure.us`<sup>2</sup> | For Azure Arc-enabled SQL Server. Sends data processing service, service telemetry, and performance monitoring to Azure. Allows TLS 1.2 or 1.3 only. | If you use Azure Arc-enabled SQL Server. | Public. |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates (uses HTTP/TCP 80 and HTTPS/TCP 443). | If you use Extended Security Updates enabled by Azure Arc. Always required for automatic updates or temporarily if you download certificates manually. | Public. |

<sup>1</sup> Access to this URL is also needed when updates are performed automatically.

<sup>2</sup> For details about what information is collected and sent, review [Data collection and reporting for SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/data-collection).


#### [Azure operated by 21Vianet](#tab/azure-china)

| Agent resource | Description | When required|
|---------|---------|--------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> |
|`packages.microsoft.com`|Used to download the Linux installation package.|Only at installation time.<sup>1</sup> |
|`login.chinacloudapi.cn`|Microsoft Entra ID.|Always.|
|`login.partner.chinacloudapi.cn`|Microsoft Entra ID.|Always.|
|`pas.chinacloudapi.cn`|Microsoft Entra ID.|Always.|
|`management.chinacloudapi.cn`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.|
|`*.his.arc.azure.cn`|Metadata and hybrid identity services.|Always.|
|`*.guestconfiguration.azure.cn`| Extension management and guest configuration services. |Always.|
|`guestnotificationservice.azure.cn`, `*.guestnotificationservice.azure.cn`|Notification service for extension and connectivity scenarios.|Always.|
|`azgn*.servicebus.chinacloudapi.cn`|Notification service for extension and connectivity scenarios.|Always.|
|`*.servicebus.chinacloudapi.cn`|For Windows Admin Center and SSH scenarios.|If you use SSH or Windows Admin Center from Azure.|
|`*.blob.core.chinacloudapi.cn`|Download source for Azure Arc-enabled servers extensions.|Always, except when you use private endpoints.|
|`dc.applicationinsights.azure.cn`|Agent telemetry.|Optional. Not used in agent versions 1.24+.|

<sup>1</sup> Access to this URL is also needed when updates are preformed automatically.

---

### Cryptographic protocols

To ensure the security of data in transit to Azure, configure your machines to use TLS 1.2 and 1.3. Older versions of TLS and Secure Sockets Layer (SSL) are vulnerable. Although these older versions still work to allow backward compatibility, don't use them.

Starting from version 1.56 of the Connected Machine agent (Windows only), you must configure the following cipher suites for at least one of the recommended TLS versions:

* TLS 1.3 (suites in server-preferred order):

  * TLS_AES_256_GCM_SHA384 (0x1302)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  * TLS_AES_128_GCM_SHA256 (0x1301)   ECDH secp256r1 (eq. 3072 bits RSA)   FS

* TLS 1.2 (suites in server-preferred order):

  * TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  * TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH secp256r1 (eq. 3072 bits RSA)   FS

For more information, see [Windows TLS configuration issues](../troubleshoot-networking.md#windows-tls-configuration-issues).

The SQL Server enabled by Azure Arc endpoints located at `*.\<region\>.arcdataservices.com` support only TLS 1.2 and 1.3. Only Windows Server 2012 R2 and later support TLS 1.2. The SQL Server enabled by Azure Arc telemetry endpoint doesn't support Windows Server 2012 or Windows Server 2012 R2.

|Platform/Language | Support | More information |
| --- | --- | --- |
|Linux | Linux distributions tend to rely on [OpenSSL](https://www.openssl.org) for TLS 1.2 support. | Check the [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) to confirm that your version of OpenSSL is supported.|
| Windows Server 2012 R2 and later | Supported and enabled by default. | Confirm that you're still using the [default settings](/windows-server/security/tls/tls-registry-settings).|
| Windows Server 2012 | Partially supported. *Not recommended.*| Some endpoints still work, but other endpoints require TLS 1.2 or later, which isn't available on Windows Server 2012.|

### Bandwidth requirements

The Azure Connected Machine agent is designed to have light bandwidth requirements for most scenarios. The exact bandwidth requirements depend on your configuration.

In normal operation, the agent makes the following regular requests. All connections are outbound.

- For notifications, one persistent WebSocket connection, requiring minimal bandwidth.
- One heartbeat request every five minutes (less than 64 KB).
- For Machine Configuration, one status check every 15 minutes (less than 100 KB).
- For VM extensions, one status check every five minutes (less than 100 KB).
- Diagnostic telemetry messages. A maximum of one message every 30 minutes, with a maximum size of 64 KB.

When **Machine Configurations** are assigned to the server, you must initially download each configuration (up to 1 MB per assignment), then send a status update (up to 200 KB, but for most configurations much smaller) every 15 minutes.

When you install or upgrade **VM extensions or VM applications** on an Azure Arc-enabled server, you must download the extension package from Microsoft's CDN endpoint.
Extension packages can be up to 2 GB, but most are significantly smaller. This download occurs when you install or update an extension on a server, either manually or through extension auto-upgrade.
Once installed, send a status update (up to 100 KB, but for most extensions much smaller) every 15 minutes.

Installed extensions have their own bandwidth requirements, which might also depend on how you configure them. For more information, see the documentation for each extension.

For Azure Monitor, see [Azure Monitor cost and usage](/azure/azure-monitor/fundamentals/cost-usage).
