---
ms.service: azure-arc
ms.topic: include
ms.date: 09/29/2025
# Customer intent: "As a network administrator, I want to configure secure outbound connectivity for the Azure Connected Machine agent so that I can ensure proper communication with Azure Arc while adhering to my organization's security policies."
---

Azure Arc-enabled server endpoints are required for all server-based Azure Arc offerings.

### Networking configuration

The Azure Connected Machine agent for Linux and Windows communicates outbound securely to Azure Arc over TCP port 443. By default, the agent uses the default route to the internet to reach Azure services. You can optionally [configure the agent to use a proxy server](../manage-agent.md#update-or-remove-proxy-settings) if your network requires it. Proxy servers don't make the Connected Machine agent more secure because the traffic is already encrypted.

To further secure your network connectivity to Azure Arc, instead of using public networks and proxy servers, you can implement an [Azure Arc private link scope](../private-link-security.md).

> [!NOTE]
> Azure Arc-enabled servers doesn't support using a [Log Analytics gateway](/azure/azure-monitor/agents/gateway) as a proxy for the Connected Machine agent. At the same time, Azure Monitor Agent supports Log Analytics gateways.

If your firewall or proxy server restricts outbound connectivity, make sure that the URLs and service tags listed here aren't blocked.

### Service tags

Be sure to allow access to the following service tags:

* `AzureActiveDirectory`
* `AzureTrafficManager`
* `AzureResourceManager`
* `AzureArcInfrastructure`
* `Storage`
* `WindowsAdminCenter` (if [you use Windows Admin Center to manage Azure Arc-enabled servers](/windows-server/manage/windows-admin-center/azure/manage-arc-hybrid-machines))

For a list of IP addresses for each service tag/region, see the JSON file [Azure IP Ranges and Service Tags - Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519). Microsoft publishes weekly updates that contain each Azure service and the IP ranges it uses. The information in the JSON file is the current point-in-time list of the IP ranges that correspond to each service tag. The IP addresses are subject to change. If IP address ranges are required for your firewall configuration, use the `AzureCloud` service tag to allow access to all Azure services. Don't disable security monitoring or inspection of these URLs. Allow them as you would other internet traffic.

If you filter traffic to the `AzureArcInfrastructure` service tag, you must allow traffic to the full service tag range. The ranges advertised for individual regions, for example, `AzureArcInfrastructure.AustraliaEast`, don't include the IP ranges that are used by global components of the service. The specific IP address resolved for these endpoints might change over time within the documented ranges. For this reason, using a lookup tool to identify the current IP address for a specific endpoint and allowing access to only that IP address isn't sufficient to ensure reliable access.

For more information, see [Virtual network service tags](/azure/virtual-network/service-tags-overview).

> [!IMPORTANT]
> To filter traffic by IP addresses in Azure Government or Azure operated by 21Vianet, be sure to add the IP addresses from the `AzureArcInfrastructure` service tag for the Azure public cloud, in addition to using the `AzureArcInfrastructure` service tag for your cloud. After October 28, 2025, adding the `AzureArcInfrastructure` service tag for Azure public cloud will be required, and the service tags for Azure Government and Azure operated by 21Vianet will no longer be supported.

### URLs

This table lists the URLs that must be available to install and use the Connected Machine agent.

#### [Azure cloud platform](#tab/azure-cloud)

> [!NOTE]
> When you configure the Connected Machine agent to communicate with Azure through a private link, some endpoints must still be accessed through the internet. The **Private link capable** column in the following table shows the endpoints that you can configure with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall and/or proxy server for the agent to function. Network traffic is routed through private endpoints if a private link scope is assigned.

| Agent resource | Description | When required| Private link capable |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> | Public. |
|`packages.microsoft.com`|Used to download the Linux installation package.|Only at installation time.<sup>1</sup> | Public. |
|`login.microsoftonline.com`|Microsoft Entra ID.|Always.| Public. |
|`*.login.microsoft.com`|Microsoft Entra ID.|Always.| Public. |
|`pas.windows.net`|Microsoft Entra ID.|Always.| Public. |
|`management.azure.com`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured. |
|`*.his.arc.azure.com`|Metadata and hybrid identity services.|Always.| Private. |
|`*.guestconfiguration.azure.com`| Extension management and guest configuration services. |Always.| Private. |
|`guestnotificationservice.azure.com`, `*.guestnotificationservice.azure.com`|Notification service for extension and connectivity scenarios.|Always.| Public. |
|`azgn*.servicebus.windows.net` or `*.servicebus.windows.net`|Notification service for extension and connectivity scenarios.|Always.| Public. |
|`*.servicebus.windows.net`|For Windows Admin Center and Secure Shell (SSH) scenarios.|If you use SSH or Windows Admin Center from Azure.|Public.|
|`*.waconazure.com`|For Windows Admin Center connectivity.|If you use Windows Admin Center.|Public.|
|`*.blob.core.windows.net`|Download source for Azure Arc-enabled server extensions.|Always, except when you use private endpoints.| Not used when a private link is configured. |
|`dc.services.visualstudio.com`|Agent telemetry.|Optional. Not used in agent versions 1.24+.| Public. |
| `*.<region>.arcdataservices.com`<sup>2</sup> | For Azure Arc-enabled SQL Server. Sends data processing service, service telemetry, and performance monitoring to Azure. Allows Transport Layer Security (TLS) 1.2 or 1.3 only. | If you use Azure Arc-enabled SQL Server. | Public. |
| `https://<azure-keyvault-name>.vault.azure.net/`, `https://graph.microsoft.com/`<sup>2</sup>| For Microsoft Entra authentication with Azure Arc-enabled SQL Server. | If you use Azure Arc-enabled SQL Server. | Public. |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates (uses HTTP/TCP 80 and HTTPS/TCP 443). | If you use Extended Security Updates enabled by Azure Arc. Always required for automatic updates or temporarily if you download certificates manually. | Public. |
|`dls.microsoft.com`| Used by Azure Arc machines to perform license validation. | Required when you use hotpatching, Windows Server Azure Benefits, or Windows Server pay-as-you-go billing on Azure Arc-enabled machines. | Public. |

<sup>1</sup> Access to this URL is also needed when updates are performed automatically.

<sup>2</sup> For details about what information is collected and sent, review [Data collection and reporting for SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/data-collection).

For extension versions up to and including February 13, 2024, use `san-af-<region>-prod.azurewebsites.net`. Beginning March 12, 2024, both Azure Arc data processing and Azure Arc data telemetry use `*.<region>.arcdataservices.com`.

> [!NOTE]
> To translate the `*.servicebus.windows.net` wildcard into specific endpoints, use the command `\GET https://guestnotificationservice.azure.com/urls/allowlist?api-version=2020-01-01&location=<region>`. Within this command, the region must be specified for the `<region>` placeholder. These endpoints might change periodically.

[!INCLUDE [arc-region-note](../../includes/arc-region-note.md)]

#### [Azure Government](#tab/azure-government)

> [!NOTE]
> When you configure the Connected Machine agent to communicate with Azure through a private link, some endpoints must still be accessed through the internet. The **Endpoint used with private link** column in the following table shows the endpoints that you can configure with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall and/or proxy server for the agent to function.

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
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates ( uses HTTP/TCP 80 and HTTPS/TCP 443). | If you use Extended Security Updates enabled by Azure Arc. Always required for automatic updates or temporarily if you download certificates manually. | Public. |

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

To ensure the security of data in transit to Azure, we strongly encourage you to configure machines to use TLS 1.2 and 1.3. Older versions of TLS/Secure Sockets Layer (SSL) were found to be vulnerable. Although they still currently work to allow backward compatibility, they *aren't recommended*.

Starting from version 1.56 of the Connected Machine agent (Windows only), the following cipher suites must be configured for at least one of the recommended TLS versions:

* TLS 1.3 (suites in server-preferred order):

  * TLS_AES_256_GCM_SHA384 (0x1302)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  * TLS_AES_128_GCM_SHA256 (0x1301)   ECDH secp256r1 (eq. 3072 bits RSA)   FS

* TLS 1.2 (suites in server-preferred order):

  * TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384 (0xc030)   ECDH secp521r1 (eq. 15360 bits RSA)   FS
  * TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256 (0xc02f)   ECDH secp256r1 (eq. 3072 bits RSA)   FS

For more information, see [Windows TLS configuration issues](../troubleshoot-networking.md#windows-tls-configuration-issues).

The SQL Server enabled by Azure Arc endpoints located at `*.\<region\>.arcdataservices.com` support only TLS 1.2 and 1.3. Only Windows Server 2012 R2 and later have support for TLS 1.2. SQL Server enabled by Azure Arc telemetry endpoint isn't supported for Windows Server 2012 or Windows Server 2012 R2.

|Platform/Language | Support | More information |
| --- | --- | --- |
|Linux | Linux distributions tend to rely on [OpenSSL](https://www.openssl.org) for TLS 1.2 support. | Check the [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) to confirm that your version of OpenSSL is supported.|
| Windows Server 2012 R2 and later | Supported and enabled by default. | Confirm that you're still using the [default settings](/windows-server/security/tls/tls-registry-settings).|
| Windows Server 2012 | Partially supported. *Not recommended.*| Some endpoints still work, but other endpoints require TLS 1.2 or later, which isn't available on Windows Server 2012.|
