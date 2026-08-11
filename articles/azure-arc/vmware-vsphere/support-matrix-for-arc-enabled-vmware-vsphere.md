---
title: Plan for deployment
description: Learn about the support matrix for Arc-enabled VMware vSphere including vCenter Server versions supported, network requirements, and more.
ms.topic: how-to
ms.date: 03/10/2026
ms.service: azure-arc
ms.subservice: vmware-vsphere-azure-arc
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
# Customer intent: As a VI admin, I want to understand the support matrix for Arc-enabled VMware vSphere.
ms.custom:
  - build-2025
---

# Support matrix for Azure Arc-enabled VMware vSphere

This article documents the prerequisites and support requirements for using [Azure Arc-enabled VMware vSphere](overview.md) to manage your VMware vSphere VMs through Azure Arc.

To use Azure Arc-enabled VMware vSphere, deploy an Azure Arc resource bridge in your VMware vSphere environment during onboarding. The resource bridge is an appliance VM that provides an ongoing connection between your VMware vCenter Server and Azure. After you connect VMware vCenter Server to Azure, components running on the resource bridge discover your vCenter inventory and synchronize it with Azure. You can then enable discovered machines for Azure Arc guest management and perform virtual hardware and guest operating system management operations through Azure.


## VMware vSphere requirements

Meet the following requirements to use Azure Arc-enabled VMware vSphere.

### Supported vCenter Server versions

Azure Arc-enabled VMware vSphere works with vCenter Server version 8.

> [!NOTE]
> Azure Arc-enabled VMware vSphere currently supports vCenters with a maximum of 9,500 VMs. If your vCenter has more than 9,500 VMs, don't use Azure Arc-enabled VMware vSphere with it at this point.

### Required vSphere account privileges

Use a vSphere account that can:

- Read all inventory.
- Deploy and update VMs to all the resource pools (or clusters), networks, and VM templates that you want to use with Azure Arc.

>[!Important]
> As part of the Azure Arc-enabled VMware onboarding script, you're prompted to provide a vSphere account to deploy the Azure Arc resource bridge VM on the ESXi host. The process stores this account locally within the Azure Arc resource bridge VM and encrypts it as a Kubernetes secret at rest. The vSphere account allows Azure Arc-enabled VMware to interact with VMware vSphere. If your organization practices routine credential rotation, you must [update the credentials in Azure Arc-enabled VMware](/azure/azure-arc/vmware-vsphere/administer-arc-vmware?tabs=account-for-arc-resource-bridge#update-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding) to maintain the connection between Azure Arc-enabled VMware and VMware vSphere.

## Resource bridge resource requirements

For Arc-enabled VMware vSphere, resource bridge has the following minimum virtual hardware requirements:

- 8 GB of memory
- 4 vCPUs
- An external virtual switch that can provide access to the internet directly or through a proxy.
- If internet access for the resource bridge is through a proxy or firewall, ensure to allow-list the following [Arc gateway (preview)](#arc-gateway-for-arc-resource-bridge-on-arc-enabled-vmware-vsphere-preview), Arc resource bridge, Arc agent and VMware vSphere required endpoints.

### Resource bridge networking requirements

When you enable Azure Arc gateway for Arc-enabled VMware vSphere (preview), supported endpoint traffic routes to a single Azure Arc gateway endpoint instead of going to each individual required networking endpoint, which simplifies firewall configuration and improves onboarding reliability. If you don't want to use [Arc gateway (preview)](#arc-gateway-for-arc-resource-bridge-on-arc-enabled-vmware-vsphere-preview) when deploying the Arc resource bridge for Arc-enabled VMware vSphere, review and enable the full list of [Arc resource bridge networking requirements](/azure/azure-arc/resource-bridge/network-requirements). 

### Designated IP ranges for Arc resource bridge

The IP ranges you assign to your VMware vSphere environment and to the Arc resource bridge configuration (IP address prefix, control plane IP, appliance VM IPs, DNS servers, proxy servers, and vSphere ESXi hosts) must not overlap with the IP ranges reserved by the Arc resource bridge. To review the reserved IP ranges, see [Designated IP ranges for Arc resource bridge](/azure/azure-arc/resource-bridge/network-requirements#designated-ip-ranges-for-arc-resource-bridge).

## VMware vSphere network requirements 

In addition, VMware vSphere requires the following:

[!INCLUDE [netork-requirements](includes/network-requirements.md)]

For a complete list of network requirements for Azure Arc features and Azure Arc-enabled services, see [Azure Arc network requirements (Consolidated)](../network-requirements-consolidated.md).

## Azure role and permission requirements

The minimum Azure roles required for operations related to Arc-enabled VMware vSphere are as follows:

| **Operation** | **Minimum role required** | **Scope** |
| --- | --- | --- |
| Onboarding your vCenter Server to Arc | Azure Arc VMware Private Clouds Onboarding | On the subscription or resource group into which you want to onboard |
| Administering Arc-enabled VMware vSphere | Azure Arc VMware Administrator | On the subscription or resource group where vCenter server resource is created |
| VM Provisioning | Azure Arc VMware Private Cloud User | On the subscription or resource group that contains the resource pool/cluster/host, datastore, and virtual network resources, or on the resources themselves |
| VM Provisioning | Azure Arc VMware VM Contributor | On the subscription or resource group where you want to provision VMs |
| VM Operations | Azure Arc VMware VM Contributor | On the subscription or resource group that contains the VM, or on the VM itself |

If you have roles with higher permissions on the same scope, such as Owner or Contributor, you can also perform the operations listed earlier.

## Guest management (Arc agent) requirements

By using Arc-enabled VMware vSphere, you can install the Azure Arc connected machine agent on your VMs at scale and use Azure management services on the VMs. This capability has additional requirements.

To enable guest management (install the Azure Arc connected machine agent), ensure the following:

- The VM is powered on.
- The VM has VMware tools installed and running.
- The resource bridge has access to the host on which the VM is running.
- The VM is running a [supported operating system](#supported-operating-systems).
- The VM has internet connectivity directly or through proxy. If the connection is through a proxy, ensure [these URLs](#networking-requirements) are allow-listed.

Additionally, make sure that the requirements described in the following section are met to enable guest management.

### Supported operating systems

Make sure you're using a version of the Windows or Linux [operating systems that are officially supported for the Azure Connected Machine agent](../servers/prerequisites.md#supported-operating-systems). The agent supports only x86-64 (64-bit) architectures. It doesn't support x86 (32-bit) or ARM-based architectures, including x86-64 emulation on arm64.

### Software requirements

Windows operating systems:

- .NET Framework 4.6 or later. [Download the .NET Framework](/dotnet/framework/install/guide-for-developers).
- Windows PowerShell 5.1. [Download Windows Management Framework 5.1.](https://www.microsoft.com/download/details.aspx?id=54616)

Linux operating systems:

- systemd
- wget (to download the installation script)

### Networking requirements

The Azure Arc agents need the following firewall URL exceptions:

| **URL** | **Description** |
| --- | --- |
| `aka.ms` | Used to resolve the download script during installation |
| `packages.microsoft.com` | Used to download the Linux installation package |
| `download.microsoft.com` | Used to download the Windows installation package |
| `login.windows.net` | Microsoft Entra ID |
| `login.microsoftonline.com` | Microsoft Entra ID |
| `pas.windows.net` | Microsoft Entra ID |
| `management.azure.com` | Azure Resource Manager - to create or delete the Arc server resource |
| `*.his.arc.azure.com` | Metadata and hybrid identity services |
| `*.guestconfiguration.azure.com` | Extension management and guest configuration services |
| `guestnotificationservice.azure.com`, `*.guestnotificationservice.azure.com` | Notification service for extension and connectivity scenarios |
| `azgn*.servicebus.windows.net` | Notification service for extension and connectivity scenarios |
| `*.servicebus.windows.net` | For Windows Admin Center and SSH scenarios |
| `*.blob.core.windows.net` | Download source for Azure Arc-enabled servers extensions |
| `dc.services.visualstudio.com` | Agent telemetry |


## Arc gateway for Arc resource bridge on Arc-enabled VMware vSphere (preview)

You need an existing Azure Arc gateway resource before running the Arc-enabled VMware vSphere onboarding script. [Create a new Azure Arc gateway resource](../servers/arc-gateway.md#create-an-azure-arc-gateway-resource) or reuse an existing one from another Azure Arc product. During onboarding, the script prompts you to provide your Azure Arc gateway resource ID. If you have an existing Azure Arc gateway being used in a different Azure Arc product, you can use the same gateway with Arc resource bridge. 

> [!Important]
> You can enable Azure Arc gateway only during a **new deployment** of Arc resource bridge. Azure Arc gateway isn't currently supported on an existing Azure Arc resource bridge and would require [a recovery scenario](recover-from-resource-bridge-deletion.md).

During Arc-enabled VMware vSphere deployment, the Arc gateway router that runs inside the Arc resource bridge establishes a connection to an existing Arc gateway resource. Supported outbound traffic is then routed through the Arc gateway endpoint instead of going directly to each individual Microsoft URL.

Arc gateway reduces, but doesn't eliminate, the required networking endpoints you must allow. You still need to allow the required endpoints that can't be routed through the gateway, such as bootstrap endpoints. The connection between the on-premises Arc proxy and the Arc gateway is tunnel-based. Proxies that perform TLS/SSL inspection can't inspect the contents of the tunnel and aren't supported. 

### Arc gateway (preview) prerequisites

Before you enable Arc gateway for Arc-enabled VMware vSphere, ensure the following prerequisites are met:

- You're deploying an Arc resource bridge version 1.8.0 or later. 
- You register the **Microsoft.HybridCompute** resource provider in your Azure subscription.
- You have the **Azure Arc Gateway Manager** role, which is required to create the Arc gateway resource and manage its association.
- You have an Arc gateway resource in the **same Azure region** and **same Azure tenant** as the Arc resource bridge. The gateway resource doesn't need to be in the same subscription or resource group. If it resides in a different subscription or resource group, you must have access to all involved subscriptions and resource groups.
- All Arc gateway (preview) firewall and proxy endpoints are allowlisted.

### Supported proxy configurations for Arc gateway (preview)

Arc gateway for Arc resource bridge supports these proxy configurations:

- HTTP proxy (with or without `no_proxy`)

Use `no_proxy` to set addresses or domains that bypass the proxy.

The following configurations aren't supported:

- HTTPS proxy (any proxy configuration that requires a certificate)
- Transparent proxy
- Enterprise proxy or firewall that performs SSL/TLS inspection


### Arc gateway (preview) firewall/proxy endpoint allowlist

Ensure that your Azure Arc gateway URL is allowed as listed in the following table:

| Service | Port | URL | Direction | Notes |
|----------|------|-----|-----------|-------|
| Arc gateway | 443 | Your unique Arc gateway URL | Outbound from Arc resource bridge appliance VM IPs | Reduces the number of Microsoft URLs you must allow. |

### Arc resource bridge with Arc gateway (preview) firewall and proxy endpoints allowlist

The following table lists the URLs that **still require direct access** (not routed through Arc gateway) and must be individually allowed. The allowlist enables communication from [the management machine and Arc resource bridge IP addresses](quick-start-connect-vcenter-to-arc-using-script.md#inputs-for-the-script) to the required endpoints.


| **Service** | **Port** | **URL** | **Direction** | **Notes** |
| --- | --- | --- | --- | --- |
| SFS API endpoint | 443 | `msk8s.api.cdp.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Download product catalog, product bits, and OS images from SFS. |
| Resource bridge (appliance) image download | 443 | `msk8s.sb.tlu.dl.delivery.mp.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Download the Arc resource bridge OS images. |
| Microsoft Container Registry | 443 | `mcr.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Discover container images for Arc resource bridge. |
| Microsoft Container Registry | 443 | `*.data.mcr.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Download container images for Arc resource bridge. |
| Resource bridge (appliance) container image download | 443 | `*.blob.core.windows.net`, `ecpacr.azurecr.io` | Appliance VM IPs need outbound connection. | Required to pull container images. |
| Log collection for Arc resource bridge | 443 | `linuxgeneva-microsoft.azurecr.io` | Appliance VM IPs need outbound connection. | Push logs for appliance managed components. |
| Microsoft open source packages manager | 443 | `packages.microsoft.com` | Appliance VM IPs need outbound connection. | Download Linux installation package. |
| Windows NTP Server | 123 | `time.windows.com` | Management machine and appliance VM IPs (if Hyper-V default is Windows NTP) need outbound connection on UDP. | OS time sync in appliance VM and management machine. |
| Managed Identity | 443 | `*.his.arc.azure.com` | Appliance VM IPs need outbound connection. | Required to pull system-assigned Managed Identity certificates. |
| Azure Resource Manager | 443 | `management.azure.com` | Management machine and appliance VM IPs need outbound connection. | Manage resources in Azure. |
| Microsoft Entra ID | 443 | `login.microsoftonline.com` | Management machine and appliance VM IPs need outbound connection. | Required to update ARM tokens. |
| Microsoft Entra ID | 443 | `*.login.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Required to update ARM tokens. |
| Microsoft Entra ID | 443 | `login.windows.net` | Management machine and appliance VM IPs need outbound connection. | Required to update ARM tokens. |
| Microsoft Graph | 443 | `graph.microsoft.com` | Management machine and appliance VM IPs need outbound connection. | Required for Azure RBAC. |
| Custom Location | 443 | `sts.windows.net` | Appliance VM IPs need outbound connection. | Required for Custom Location. |
| Azure service bus | 443 | `*.servicebus.windows.net` | Appliance VM IPs need outbound connection. | Enables secure control channel. |
| Azure CLI | 443 | `*.blob.core.windows.net` | Management machine needs outbound connection. | Download Azure CLI installer. |
| Arc Extension | 443 | `*.web.core.windows.net` | Management machine needs outbound connection. | Download Arc resource bridge extension. |
| Python package | 443 | `pypi.org`, `*.pypi.org` | Management machine needs outbound connection. | Validate Kubernetes and Python versions. |
| Python package | 443 | `pythonhosted.org`, `*.pythonhosted.org` | Management machine needs outbound connection. | Python packages for Azure CLI installation. |

## Next steps

- [Connect VMware vCenter to Azure Arc using the helper script](quick-start-connect-vcenter-to-arc-using-script.md)

