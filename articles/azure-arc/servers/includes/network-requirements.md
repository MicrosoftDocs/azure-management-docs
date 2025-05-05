---
ms.service: azure-arc
ms.topic: include
ms.date: 01/13/2025
---

Azure Arc-enabled server endpoints are required for all server based Arc offerings. 

### Networking configuration

The Azure Connected Machine agent for Linux and Windows communicates outbound securely to Azure Arc over TCP port 443. By default, the agent uses the default route to the internet to reach Azure services. You can optionally [configure the agent to use a proxy server](../manage-agent.md#update-or-remove-proxy-settings) if your network requires it. Proxy servers don't make the Connected Machine agent more secure because the traffic is already encrypted.

To further secure your network connectivity to Azure Arc, instead of using public networks and proxy servers, you can implement an [Azure Arc Private Link Scope](../private-link-security.md) .

> [!NOTE]
> Azure Arc-enabled servers does not support using a [Log Analytics gateway](/azure/azure-monitor/agents/gateway) as a proxy for the Connected Machine agent. At the same time, Azure Monitor Agent supports Log Analytics gateway.

If outbound connectivity is restricted by your firewall or proxy server, make sure the URLs and Service Tags listed below are not blocked.

### Service tags

Be sure to allow access to the following Service Tags:

* AzureActiveDirectory
* AzureTrafficManager
* AzureResourceManager
* AzureArcInfrastructure
* Storage
* WindowsAdminCenter (if [using Windows Admin Center to manage Arc-enabled servers](/windows-server/manage/windows-admin-center/azure/manage-arc-hybrid-machines))

For a list of IP addresses for each service tag/region, see the JSON file [Azure IP Ranges and Service Tags – Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519). Microsoft publishes weekly updates containing each Azure Service and the IP ranges it uses. This information in the JSON file is the current point-in-time list of the IP ranges that correspond to each service tag. The IP addresses are subject to change. If IP address ranges are required for your firewall configuration, then the **AzureCloud** Service Tag should be used to allow access to all Azure services. Do not disable security monitoring or inspection of these URLs, allow them as you would other Internet traffic.

If you filter traffic to the AzureArcInfrastructure service tag, you must allow traffic to the full service tag range. The ranges advertised for individual regions, for example AzureArcInfrastructure.AustraliaEast, do not include the IP ranges used by global components of the service. The specific IP address resolved for these endpoints may change over time within the documented ranges, so just using a lookup tool to identify the current IP address for a given endpoint and allowing access to that will not be sufficient to ensure reliable access.

For more information, see [Virtual network service tags](/azure/virtual-network/service-tags-overview).

### URLs

The table below lists the URLs that must be available in order to install and use the Connected Machine agent.

#### [Azure Cloud](#tab/azure-cloud)

> [!NOTE]
> When configuring the Azure connected machine agent to communicate with Azure through a private link, some endpoints must still be accessed through the internet. The **Private link capable** column in the following table shows which endpoints can be configured with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall and/or proxy server for the agent to function. Network traffic is routed through private endpoint if a private link scope is assigned.

| Agent resource | Description | When required| Private link capable |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package|At installation time, only <sup>1</sup> | Public |
|`packages.microsoft.com`|Used to download the Linux installation package|At installation time, only <sup>1</sup> | Public |
|`login.microsoftonline.com`|Microsoft Entra ID|Always| Public |
|`*.login.microsoft.com`|Microsoft Entra ID|Always| Public |
|`pas.windows.net`|Microsoft Entra ID|Always| Public |
|`management.azure.com`|Azure Resource Manager - to create or delete the Arc server resource|When connecting or disconnecting a server, only| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured |
|`*.his.arc.azure.com`|Metadata and hybrid identity services|Always| Private |
|`*.guestconfiguration.azure.com`| Extension management and guest configuration services |Always| Private |
|`guestnotificationservice.azure.com`, `*.guestnotificationservice.azure.com`|Notification service for extension and connectivity scenarios|Always| Public |
|`azgn*.servicebus.windows.net`|Notification service for extension and connectivity scenarios|Always| Public |
|`*.servicebus.windows.net`|For Windows Admin Center and SSH scenarios|If using SSH or Windows Admin Center from Azure|Public|
|`*.waconazure.com`|For Windows Admin Center connectivity|If using Windows Admin Center|Public|
|`*.blob.core.windows.net`|Download source for Azure Arc-enabled servers extensions|Always, except when using private endpoints| Not used when private link is configured |
|`dc.services.visualstudio.com`|Agent telemetry|Optional, not used in agent versions 1.24+| Public |
| `*.<region>.arcdataservices.com` <sup>2</sup> | For Arc SQL Server. Sends data processing service, service telemetry, and performance monitoring to Azure. Allows TLS 1.2 or 1.3 only. | Always | Public |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for ESUs (note: uses HTTP/TCP 80 and HTTPS/TCP 443) | If using ESUs enabled by Azure Arc. Required always for automatic updates, or temporarily if downloading certificates manually. | Public |
|`dls.microsoft.com`| Used by Arc machines to perform license validation | Required when using Hotpatching, Windows Server Azure Benefits or Windows Server PayGo on Arc enabled machines | Public |


<sup>1</sup> Access to this URL also needed when performing updates automatically.

<sup>2</sup> For details about what information is collected and sent, review [Data collection and reporting for SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/data-collection).


For extension versions up to and including February 13, 2024, use `san-af-<region>-prod.azurewebsites.net`. Beginning March 12, 2024 both Azure Arc data processing, and Azure Arc data telemetry use `*.<region>.arcdataservices.com`. 

> [!NOTE]
> To translate the `*.servicebus.windows.net` wildcard into specific endpoints, use the command `\GET https://guestnotificationservice.azure.com/urls/allowlist?api-version=2020-01-01&location=<region>`. Within this command, the region must be specified for the `<region>` placeholder. These endpoints may change periodically.

[!INCLUDE [arc-region-note](../../includes/arc-region-note.md)]

#### [Azure Government](#tab/azure-government)

> [!NOTE]
> When configuring the Azure connected machine agent to communicate with Azure through a private link, some endpoints must still be accessed through the internet. The **Endpoint used with private link** column in the following table shows which endpoints can be configured with a private endpoint. If the column shows *Public* for an endpoint, you must still allow access to that endpoint through your organization's firewall and/or proxy server for the agent to function.

| Agent resource | Description | When required| Endpoint used with private link |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package|At installation time, only <sup>1</sup> | Public |
|`packages.microsoft.com`|Used to download the Linux installation package|At installation time, only <sup>1</sup> | Public |
|`login.microsoftonline.us`|Microsoft Entra ID|Always| Public |
|`pasff.usgovcloudapi.net`|Microsoft Entra ID|Always| Public |
|`management.usgovcloudapi.net`|Azure Resource Manager - to create or delete the Arc server resource|When connecting or disconnecting a server, only| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured |
|`*.his.arc.azure.us`|Metadata and hybrid identity services|Always| Private |
|`*.guestconfiguration.azure.us`| Extension management and guest configuration services |Always| Private |
|`*.blob.core.usgovcloudapi.net`|Download source for Azure Arc-enabled servers extensions|Always, except when using private endpoints| Not used when private link is configured |
|`dc.applicationinsights.us`|Agent telemetry|Optional, not used in agent versions 1.24+| Public |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for ESUs (note: uses HTTP/TCP 80 and HTTPS/TCP 443) | If using ESUs enabled by Azure Arc. Required always for automatic updates, or temporarily if downloading certificates manually. | Public |

<sup>1</sup> Access to this URL also needed when performing updates automatically.

#### [Microsoft Azure operated by 21Vianet](#tab/azure-china)

| Agent resource | Description | When required|
|---------|---------|--------|
|`download.microsoft.com`|Used to download the Windows installation package|At installation time, only <sup>1</sup> |
|`packages.microsoft.com`|Used to download the Linux installation package|At installation time, only <sup>1</sup> |
|`login.chinacloudapi.cn`|Microsoft Entra ID|Always|
|`login.partner.chinacloudapi.cn`|Microsoft Entra ID|Always|
|`pas.chinacloudapi.cn`|Microsoft Entra ID|Always|
|`management.chinacloudapi.cn`|Azure Resource Manager - to create or delete the Arc server resource|When connecting or disconnecting a server, only|
|`*.his.arc.azure.cn`|Metadata and hybrid identity services|Always|
|`*.guestconfiguration.azure.cn`| Extension management and guest configuration services |Always|
|`guestnotificationservice.azure.cn`, `*.guestnotificationservice.azure.cn`|Notification service for extension and connectivity scenarios|Always|
|`azgn*.servicebus.chinacloudapi.cn`|Notification service for extension and connectivity scenarios|Always|
|`*.servicebus.chinacloudapi.cn`|For Windows Admin Center and SSH scenarios|If using SSH or Windows Admin Center from Azure|
|`*.blob.core.chinacloudapi.cn`|Download source for Azure Arc-enabled servers extensions|Always, except when using private endpoints|
|`dc.applicationinsights.azure.cn`|Agent telemetry|Optional, not used in agent versions 1.24+|

<sup>1</sup> Access to this URL also needed when performing updates automatically.

---

### Transport Layer Security 1.2 protocol

To ensure the security of data in transit to Azure, we strongly encourage you to configure machine to use Transport Layer Security (TLS) 1.2. Older versions of TLS/Secure Sockets Layer (SSL) have been found to be vulnerable and while they still currently work to allow backwards compatibility, they are **not recommended**. 

The SQL Server enabled by Azure Arc endpoints located at *.\<region\>.arcdataservices.com only support TLS 1.2 and 1.3. Only Windows Server 2012 R2 and higher have support for TLS 1.2. SQL Server enabled by Azure Arc telemetry endpoint is not supported for Windows Server 2012 or Windows Server 2012 R2.

|Platform/Language | Support | More Information |
| --- | --- | --- |
|Linux | Linux distributions tend to rely on [OpenSSL](https://www.openssl.org) for TLS 1.2 support. | Check the [OpenSSL Changelog](https://www.openssl.org/news/changelog.html) to confirm your version of OpenSSL is supported.|
| Windows Server 2012 R2 and higher | Supported, and enabled by default. | To confirm that you are still using the [default settings](/windows-server/security/tls/tls-registry-settings).|
| Windows Server 2012 | Partially supported. **Not recommended.**|  Some endpoints as noted above will work, but some endpoints require TLS 1.2 or higher which is not available on Windows Server 2012.|
