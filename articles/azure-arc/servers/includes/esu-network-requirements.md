---
ms.service: azure-arc
ms.topic: include
ms.date: 01/13/2025
# Customer intent: "As a system administrator managing Azure Arc-enabled servers, I want to understand the required endpoints for Extended Security Updates so that I can ensure proper connectivity and compliance during installation and updates."
---

If you use Azure Arc-enabled servers only for Extended Security Updates for either or both of the following products:

- Windows Server 2012
- SQL Server 2012

You can enable the following subset of endpoints.

#### [Azure cloud platform](#tab/azure-cloud)

| Agent resource | Description | When required| Endpoint used with private link |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> | Public. |
|`login.windows.net`|Microsoft Entra ID.|Always.| Public. |
|`login.microsoftonline.com`|Microsoft Entra ID.|Always.| Public. |
|`*.login.microsoft.com`|Microsoft Entra ID.|Always.| Public. |
|`management.azure.com`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured. |
|`*.his.arc.azure.com`|Metadata and hybrid identity services.|Always.| Private. |
|`*.guestconfiguration.azure.com`| Extension management and guest configuration services. |Always.| Private. |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates (uses HTTP/TCP 80 and HTTPS/TCP 443). | Always for automatic updates or temporarily if you download certificates manually. | Public. |
|`*.<region>.arcdataservices.com`| Azure Arc data processing service and service telemetry.| SQL Server Extended Security Updates. | Public.|
|`*.blob.core.windows.net` | Download SQL Server Extension package. | SQL Server Extended Security Updates. | Not required if you use Azure Private Link. |

<sup>1</sup> Access to this URL is also needed when you perform updates automatically.

#### [Azure Government](#tab/azure-government)

| Agent resource | Description | When required| Endpoint used with private link |
|---------|---------|--------|---------|
|`download.microsoft.com`|Used to download the Windows installation package.|Only at installation time.<sup>1</sup> | Public. |
|`login.microsoftonline.us`|Microsoft Entra ID.|Always.| Public. |
|`management.usgovcloudapi.net`|Azure Resource Manager is used to create or delete the Azure Arc server resource.|Only when you connect or disconnect a server.| Public, unless a [resource management private link](/azure/azure-resource-manager/management/create-private-link-access-portal) is also configured. |
|`*.his.arc.azure.us`|Metadata and hybrid identity services.|Always.| Private. |
|`*.guestconfiguration.azure.us`| Extension management and guest configuration services. |Always.| Private. |
|`www.microsoft.com/pkiops/certs`| Intermediate certificate updates for Extended Security Updates (uses HTTP/TCP 80 and HTTPS/TCP 443). | Always for automatic updates or temporarily if you download certificates manually. | Public. |
|`*.blob.core.usgovcloudapi.net` | Download SQL Server Extension package. | SQL Server Extended Security Updates. | Not required if you use Azure Private Link. |

<sup>1</sup> Access to this URL is also needed when you perform updates automatically.

#### [Azure operated by 21Vianet](#tab/azure-china)

> [!NOTE]
> Azure Arc-enabled servers used for Extended Security Updates for Windows Server 2012 aren't available in Microsoft Azure operated by 21Vianet regions at this time.
