---
ms.topic: include
ms.date: 08/25/2026

# Customer intent: "As a network administrator, I want to configure outbound and inbound connectivity settings for the appliance VM and management machine, so that I can ensure smooth communication and functionality for the Arc resource bridge."
---

## Inbound connectivity requirements

To deploy and maintain Arc resource bridge, the management machine, appliance VM IPs, and control plane IPs need to communicate through the following ports. Ensure these ports are open and that traffic doesn't go through a proxy.

> [!IMPORTANT]
> During onboarding, provide two IP addresses for the Arc Resource Bridge appliance VMs - either as a range or as two individual IPs. For successful deployment, operations, and upgrades:
> - Ensure the management machine, appliance VM IPs, and control plane IPs can communicate over the required ports listed in the following table.
> - Don't route traffic through a proxy for these connections.

|**Service**|**Port**|**IP/machine**|**Direction**|**Notes**|
|--|--|--|--|--|
|SSH| 22 | `appliance VM IPs` and `Management machine` | Bidirectional | Management machine connects outbound to the appliance VM IPs. Appliance VM IPs must allow inbound connections.|
|Kubernetes API server| 6443 | `appliance VM IPs` and `Management machine` | Bidirectional | Management machine connects outbound to the appliance VM IPs. Appliance VM IPs must allow inbound connections.|
|SSH| 22 | `control plane IP` and `Management machine` | Bidirectional | Used for deploying and maintaining the appliance VM.|
|Kubernetes API server| 6443 | `control plane IP` and `Management machine` | Bidirectional | Management of the appliance VM.|
|HTTPS | 443 | `private cloud control plane address` and `Management machine` | Management machine needs outbound connection. | Communication with private cloud (ex: VMware vCenter address and vSphere datastore).|
|Kubernetes API server| 6443, 2379, 2380, 10250, 10257, 10259 | `appliance VM IPs` (to each other) | Bidirectional | Required for appliance VM upgrade. Ensure all appliance VM IPs have outbound connectivity to each other over these ports.|
|HTTPS | 443 | `private cloud control plane address` and `appliance VM IPs` | appliance VM IPs need outbound connection. | Communication with private cloud (ex: VMware vCenter address and vSphere datastore).|

### Outbound connectivity requirements

> [!NOTE]
> For Arc-enabled VMware vSphere, this requirement doesn't apply if you use Azure Arc gateway (preview). Azure Arc gateway for Arc-enabled VMware vSphere (preview) reduces the firewall and proxy URL allow list requirements. For more information, see [Arc-enabled VMware vSphere - Support Matrix](/azure/azure-arc/vmware-vsphere/support-matrix-for-arc-enabled-vmware-vsphere).

The following firewall and proxy URLs must be on the allow list in order to enable communication from the management machine, Arc resource bridge VM (initially deployed), Arc resource bridge VM 2 (upgrade creates a new VM using a different VM IP), and Control Plane IP to the required Arc resource bridge URLs.

> [!IMPORTANT]
> When onboarding Arc Resource Bridge, you must provide two IP addresses for the appliance VMs. Specify these IP addresses as either:
> - A range of IPs
> - Two individual IPs (one for each VM)
>
> To ensure successful upgrades, all appliance VM IPs must have outbound access to the required URLs. Ensure these URLs are on the allow list in your network.

### Firewall/Proxy URL allow list

### [Azure Cloud](#tab/azure-cloud)

|**Service**|**Port**|**URL**|**Direction**|**Notes**|
|--|--|--|--|--|
|DNS servers | 53 | Your DNS server IPs | Management machine and appliance VM IPs need outbound connection. | Network connectivity to the DNS servers specified during deployment to resolve required service endpoints. |
|SFS API endpoint | 443 | `msk8s.api.cdp.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Download product catalog, product bits, and OS images from SFS. |
|Resource bridge (appliance) image download| 443 | `msk8s.sb.tlu.dl.delivery.mp.microsoft.com`| Management machine & Appliance VM IPs need outbound connection. |  Download the Arc Resource Bridge OS images.|
|Microsoft Container Registry| 443 | `mcr.microsoft.com`| Management machine & Appliance VM IPs need outbound connection. | Discover container images for Arc Resource Bridge.|
|Microsoft Container Registry| 443 | `*.data.mcr.microsoft.com`| Management machine & Appliance VM IPs need outbound connection. | Download container images for Arc Resource Bridge.|
|Windows NTP Server| 123 | `time.windows.com` | Management machine & Appliance VM IPs (if Hyper-V default is Windows NTP) need outbound connection on UDP | OS time sync in appliance VM & Management machine (Windows NTP).|
|Azure Resource Manager| 443 | `management.azure.com`| Management machine & Appliance VM IPs need outbound connection. | Manage resources in Azure. |
|Microsoft Graph | 443 | `graph.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Required for Azure RBAC. |
|Azure Resource Manager | 443 | `login.microsoftonline.com`| Management machine & Appliance VM IPs need outbound connection. | Required to update ARM tokens.|
|Azure Resource Manager | 443 | `*.login.microsoft.com`| Management machine & Appliance VM IPs need outbound connection. | Required to update ARM tokens.|
|Azure Resource Manager | 443 | `login.windows.net`| Management machine & Appliance VM IPs need outbound connection. | Required to update ARM tokens.|
|Resource bridge (appliance) Dataplane service| 443 | `*.dp.prod.appliances.azure.com`| Appliance VMs IP need outbound connection. | Communicate with resource provider in Azure.|
|Resource bridge (appliance) container image download| 443 | `*.blob.core.windows.net, ecpacr.azurecr.io`| Appliance VM IPs need outbound connection. | Required to pull container images. |
|Managed Identity| 443 | `*.his.arc.azure.com`| Appliance VM IPs need outbound connection. | Required to pull system-assigned Managed Identity certificates. |
|Microsoft events data service | 443 |`v20.events.data.microsoft.com`| Appliance VM IPs need outbound connection. | Send diagnostic data from Windows. |
|Log collection for Arc Resource Bridge| 443 | `linuxgeneva-microsoft.azurecr.io`| Appliance VM IPs need outbound connection. | Push logs for Appliance managed components.|
|Microsoft open source packages manager| 443 | `packages.microsoft.com`| Appliance VM IPs need outbound connection. | Download Linux installation package.|
|Custom Location| 443 | `sts.windows.net`| Appliance VM IPs need outbound connection. | Required for Custom Location.|
|Azure Arc| 443 | `guestnotificationservice.azure.com` | Appliance VM IPs need outbound connection. | Required for Azure Arc.|
|Diagnostic data | 443 | `gcs.prod.monitoring.core.windows.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
|Diagnostic data | 443 | `*.prod.microsoftmetrics.com` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
|Diagnostic data | 443 | `*.prod.hot.ingest.monitor.core.windows.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
|Diagnostic data | 443 | `*.prod.warm.ingest.monitor.core.windows.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
|Azure service bus | 443 | `*.servicebus.windows.net`| Appliance VM IPs need outbound connection. Outbound WebSocket (wss://) connections must be allowed. | Enables secure control channel.|
|Azure CLI | 443 | `*.blob.core.windows.net`| Management machine needs outbound connection. | Download Azure CLI Installer. |
|Arc Extension | 443 | `*.web.core.windows.net`| Management machine needs outbound connection. | Download Arc resource bridge extension. |
|Azure Arc Agent| 443 | `*.dp.kubernetesconfiguration.azure.com`| Management machine needs outbound connection. | Dataplane used for Arc agent.|
|Python package| 443 | `pypi.org`, `*.pypi.org`| Management machine needs outbound connection. | Validate Kubernetes and Python versions.|
|Azure CLI| 443 | `pythonhosted.org`, `*.pythonhosted.org`| Management machine needs outbound connection. | Python packages for Azure CLI installation.|

### [Azure Government](#tab/azure-government)

| **Service** | **Port** | **Azure US Government URL** | **Direction** | **Notes** |
| --- | --- | --- | --- | --- |
| DNS servers | 53 | Your DNS server IPs | Management machine & Appliance VM IPs need outbound connection. | Network connectivity to the DNS servers specified during deployment to resolve required service endpoints. |
| SFS API endpoint | 443 | `msk8s.api.cdp.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Download product catalog, product bits, and OS images from SFS. |
| Resource bridge (appliance) image download | 443 | `msk8s.sb.tlu.dl.delivery.mp.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Download the Arc Resource Bridge OS images. |
| Microsoft Container Registry | 443 | `mcr.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Discover container images for Arc Resource Bridge. |
| Microsoft Container Registry | 443 | `*.data.mcr.microsoft.com` | Management machine & Appliance VM IPs need outbound connection. | Download container images for Arc Resource Bridge. |
| Windows NTP Server | 123 | `time.windows.com` | Management machine & Appliance VM IPs (if Hyper-V default is Windows NTP) need outbound connection on UDP | OS time sync in appliance VM & Management machine (Windows NTP). |
| Azure Resource Manager | 443 | `management.usgovcloudapi.net` | Management machine & Appliance VM IPs need outbound connection. | Manage resources in Azure. |
| Microsoft Graph | 443 | `graph.microsoft.us` | Management machine & Appliance VM IPs need outbound connection. | Required for Azure RBAC. |
| Azure Resource Manager | 443 | `login.microsoftonline.us` <br> `<region>.login.microsoftonline.us` | Management machine & Appliance VM IPs need outbound connection. | Required to update ARM tokens. <br> <br> Region example: `usgovvirginia.login.microsoftonline.us` |
| Resource bridge (appliance) Dataplane service | 443 | `*.dp.prod.appliances.azure.us` | Appliance VMs IP need outbound connection. | Communicate with resource provider in Azure. |
| Resource bridge (appliance) container image download | 443 | `*.blob.core.usgovcloudapi.net` | Appliance VM IPs need outbound connection. | Required to pull container images. |
| Managed Identity | 443 | `gbl.his.arc.azure.us`, `usgv.his.arc.azure.us` (not region dependent) | Appliance VM IPs need outbound connection. | Required to pull system-assigned Managed Identity certificates. |
| Microsoft events data service | 443 | `v20.events.data.microsoft.com` | Appliance VM IPs need outbound connection. | Send diagnostic data from Windows. |
| Log collection for Arc Resource Bridge | 443 | `linuxgeneva-microsoft.azurecr.io` | Appliance VM IPs need outbound connection. | Push logs for Appliance managed components. |
| Microsoft open source packages manager | 443 | `packages.microsoft.com` | Appliance VM IPs need outbound connection. | Download Linux installation package. |
| Custom Location | 443 | `sts.windows.net` | Appliance VM IPs need outbound connection. | Required for Custom Location. |
| Azure Arc | 443 | `guestnotificationservice.azure.us` | Appliance VM IPs need outbound connection. | Required for Azure Arc. |
| Diagnostic data | 443 | `gcs.prod.monitoring.core.usgovcloudapi.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
| Diagnostic data | 443 | `*.prod.microsoftmetrics.com` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
| Diagnostic data | 443 | `*.prod.hot.ingest.monitor.core.usgovcloudapi.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
| Diagnostic data | 443 | `*.prod.warm.ingest.monitor.core.windows.net` | Appliance VM IPs need outbound connection. | Periodically sends Microsoft required diagnostic data. |
| Azure service bus | 443 | `*.servicebus.usgovcloudapi.net` | Appliance VM IPs need outbound connection. Outbound WebSocket (wss://) connections must be allowed. | Enables secure control channel. |
| Azure CLI | 443 | `*.blob.core.usgovcloudapi.net` | Management machine needs outbound connection. | Download Azure CLI Installer. |
| Arc Extension | 443 | `*.web.core.usgovcloudapi.net` | Management machine needs outbound connection. | Download Arc resource bridge extension. |
| Azure Arc Agent | 443 | `*.dp.kubernetesconfiguration.azure.us` | Management machine needs outbound connection. | Dataplane used for Arc agent. |
| Python package | 443 | `pypi.org` <br> `*.pypi.org` | Management machine needs outbound connection. | Validate Kubernetes and Python versions. |
| Azure CLI | 443 | `pythonhosted.org` <br> `*.pythonhosted.org` | Management machine needs outbound connection. | Python packages for Azure CLI installation. |

---
