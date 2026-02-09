---
title: Support matrix for Azure Arc-enabled System Center Virtual Machine Manager
description: Learn about the support matrix for Arc-enabled System Center Virtual Machine Manager.
ms.topic: how-to
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.date: 02/09/2026
keywords: "VMM, Arc, Azure"

# Customer intent: As a VI admin, I want to understand the support matrix for System Center Virtual Machine Manager.
ms.custom:
  - build-2025
---

# Support matrix for Azure Arc-enabled System Center Virtual Machine Manager

This article documents the prerequisites and support requirements for using [Azure Arc-enabled System Center Virtual Machine Manager (SCVMM)](overview.md) to manage your SCVMM managed on-premises VMs through Azure Arc.

To use Arc-enabled SCVMM, you must deploy an Azure Arc Resource Bridge in your SCVMM managed environment. The Resource Bridge provides an ongoing connection between your SCVMM management server and Azure. Once you connect your SCVMM management server to Azure, components on the Resource Bridge discover your SCVMM management server inventory. You can [enable them in Azure](enable-scvmm-inventory-resources.md) and start performing virtual hardware and guest OS operations on them using Azure Arc.

## System Center Virtual Machine Manager requirements

The following requirements must be met to use Arc-enabled SCVMM.

### Supported SCVMM versions

Azure Arc-enabled SCVMM works with VMM 2025, 2022, and 2019 versions. It supports SCVMM management servers with a maximum of 15,000 VMs.

### Azure Arc resource bridge prerequisites

> [!NOTE]
> If the VMM server runs on a Windows Server 2016 machine, ensure that the [Open SSH package](https://github.com/PowerShell/Win32-OpenSSH/releases) is installed.
> If you deploy an older version of appliance (version lesser than 0.2.25), Arc operation fails with the error *Appliance cluster is not deployed with AAD authentication*. To fix this issue, download the latest version of the onboarding script and deploy the Resource Bridge again.
> Azure Arc Resource Bridge deployment by using private link isn't currently supported.

| **Requirement** | **Details** |
| --- | --- |
| **Azure** | An Azure subscription  <br/><br/> A resource group in the above subscription where you have the *Owner/Contributor* role. |
| **SCVMM** | You need an SCVMM management server running version 2019 or later.<br/><br/> A private cloud or a host group with a minimum free capacity of 32 GB of RAM, 4 vCPUs with 100 GB of free disk space. The supported storage configurations are hybrid storage (flash and HDD) and all-flash storage (SSDs or NVMe). <br/><br/> A VM network with internet access, directly or through proxy. You deploy the appliance VM by using this VM network.<br/><br/> Only static IP allocation is supported; dynamic IP allocation by using DHCP isn't supported. You can perform static IP allocation by using one of the following approaches:<br><br> 1. **VMM IP Pool**: Follow [these steps](/system-center/vmm/network-pool?view=sc-vmm-2022&preserve-view=true) to create a VMM Static IP Pool and ensure that the Static IP Pool has at least three IP addresses. If your SCVMM server is behind a firewall, all the IPs in this IP Pool and the Control Plane IP need to communicate through WinRM ports. The default WinRM ports are 5985 and 5986. <br> <br> 2. **Custom IP range**: Ensure that your VM network has three continuous free IP addresses. If your SCVMM server is behind a firewall, all the IPs in this IP range and the Control Plane IP need to communicate through WinRM ports. The default WinRM ports are 5985 and 5986. If the VM network is configured with a VLAN, the VLAN ID is required as an input. Azure Arc Resource Bridge requires internal and external DNS resolution to the required sites and the on-premises management machine for the Static gateway IP and the IP addresses of your DNS servers are needed. <br/><br/> A library share with write permission for the SCVMM admin account through which Resource Bridge deployment is going to be performed. |
| **SCVMM accounts** | An SCVMM admin account that can perform all administrative actions on all objects that VMM manages. <br/><br/> The user needs to be part of local administrator account in the SCVMM server. If the SCVMM server is installed in a High Availability configuration, the user needs to be a part of the local administrator accounts in all the SCVMM cluster nodes. <br/><br/>Use this account for the ongoing operation of Azure Arc-enabled SCVMM and the deployment of the Arc Resource Bridge VM. |
| **Workstation** | Use the workstation to run the helper script. Ensure you have [64-bit Azure CLI installed](/cli/azure/install-azure-cli) on the workstation.<br/><br/> When you execute the script from a Linux machine, the deployment takes a bit longer and you might experience performance issues. |

### Resource Bridge networking requirements

The following firewall URL exceptions are required for the Azure Arc Resource Bridge VM:

[!INCLUDE [network-requirements](../resource-bridge/includes/network-requirements.md)]

>[!Note] 
> To configure SSL proxy and to view the exclusion list for no proxy, see [Additional network requirements](../resource-bridge/network-requirements.md#azure-arc-resource-bridge-network-requirements).

In addition, SCVMM requires the following exception:

| **Service** | **Port** | **URL** | **Direction** | **Notes**|
| --- | --- | --- | --- | --- |
| SCVMM Management Server | 443 | URL of the SCVMM management server. | Appliance VM IP and control plane endpoint need outbound connection. | Used by the SCVMM server to communicate with the Appliance VM and the control plane. |
| WinRM | WinRM Port numbers (Default: 5985 and 5986). | URL of the WinRM service. | IPs in the IP Pool used by the Appliance VM and control plane need connection with the VMM server. | Used by the SCVMM server to communicate with the Appliance VM. |

[!INCLUDE [network-requirement-principles](../includes/network-requirement-principles.md)]

For a complete list of network requirements for Azure Arc features and Azure Arc-enabled services, see [Azure Arc network requirements (Consolidated)](../network-requirements-consolidated.md).

### Azure role and permission requirements

The minimum Azure roles required for operations related to Arc-enabled SCVMM are: 

| **Operation** | **Minimum role required** | **Scope** |
| --- | --- | --- |
| Onboard your SCVMM Management Server to Arc | Azure Arc SCVMM Private Clouds Onboarding  | On the subscription or resource group into which you want to onboard |
| Administer Arc-enabled SCVMM | Azure Arc SCVMM Administrator | On the subscription or resource group where you create the SCVMM management server resource  |
| VM provisioning | Azure Arc SCVMM Private Cloud User  | On the subscription or resource group that contains the SCVMM cloud, datastore, and virtual network resources, or on the resources themselves |
| VM provisioning | Azure Arc SCVMM VM Contributor | On the subscription or resource group where you want to provision VMs |
| VM operations | Azure Arc SCVMM VM Contributor | On the subscription or resource group that contains the VM, or on the VM itself |

If you have roles with higher permissions on the same scope, such as Owner or Contributor, you can also perform the operations listed earlier. 

### Azure connected machine agent (Guest Management) requirements

Before you install Arc agents at scale for SCVMM VMs, ensure the following conditions are met:

- The Resource Bridge must be in a running state.
- The SCVMM management server must be in a connected state.
- The user account has the permissions listed in the Azure Arc-enabled SCVMM Administrator role.
- All the target machines are:
     - Powered on, and the resource bridge has network connectivity to the host running the VM.
     - Running a [supported operating system](/azure/azure-arc/servers/prerequisites#supported-operating-systems).
     - Able to connect through the firewall to communicate over the Internet, and [these URLs](/azure/azure-arc/servers/network-requirements?tabs=azure-cloud#urls) aren't blocked.

### Supported SCVMM versions

Azure Arc-enabled SCVMM supports direct installation of Arc agents in VMs managed by:

- SCVMM 2025 or later versions of SCVMM server or console
- SCVMM 2022 UR1 or later versions of SCVMM server or console
- SCVMM 2019 UR5 or later versions of SCVMM server or console

For VMs managed by other SCVMM versions, [install Arc agents through the script](install-arc-agents-using-script.md).

>[!Important]
>Maintain the SCVMM management server and the SCVMM console in the same Long-Term Servicing Channel (LTSC) and Update Rollup (UR) version.

### Supported operating systems

Azure Arc-enabled SCVMM supports direct installation of Arc agents in VMs running Windows Server 2025, 2022, 2019, 2016, 2012R2, Windows 10, and Windows 11 operating systems. For other Windows and Linux operating systems, [install Arc agents through the script](install-arc-agents-using-script.md).

### Software requirements

Windows operating systems:

* Use the latest version, [Windows Management Framework 5.1](https://www.microsoft.com/download/details.aspx?id=54616).

Linux operating systems:

* systemd
* wget (to download the installation script)
* openssl
* gnupg (Debian-based systems, only)

### Networking requirements

The Azure Arc agents require the following firewall URL exceptions:

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

### Instance metadata 

The deployment process collects the following metadata for Azure Arc resource bridge: 

- **SCVMM server credentials**
  - Username 
  - Password (The deployment process stores the password as a Kubernetes secret inside the Azure Arc resource bridge VM, which resides in your datacenter and isn't propagated to Azure.) 
- **SCVMM server version**
- **SCVMM server FQDN/IP**
- Other parameters specific to the deployment target of Azure Arc resource bridge VM, such as VMM Cloud Name, VMM Cloud ID, Host Group Name, Host Group ID, VM Network Name, VM Network ID, IP ranges, Gateway, DNS, VLAN ID, Library Server Name. The deployment process collects only the parameters specific to the Azure Arc resource bridge VM and not the entire information of your environment. 

After you successfully deploy Azure Arc resource bridge, the deployment process collects metadata for the resources associated with the VMM server: 
- VMM Cloud 
  - Name  
  - UUID
  - Storage QoS policies
  - Cloud Capacity 
    - CPU 
    - Memory 
    - VM 
- VM Network 
  - Name 
  - UUID 
- VM Template 
  - Name 
  - UUID 
  - Operating System 
  - Memory 
  - CPU Count 
  - VM Generation 
  - Disks 
  - NICs 
- Virtual Machine
  - Name 
  - UUID 
  - Power status 
  - Operating System 
  - Operating System SKU 
  - Domain Name 
  - Computer Name 
  - CPU 
  - Memory 
  - NIC Properties 
  - IP Address 
  - Address Type 
  - Disks 
  - Availability Sets 
  - AD/DNS FQDN
  - Host Operating System 
  - Host type 
- Host 
  - Operating System 
  - Host type 

After you install the Azure Connected Machine agent in the individual VMs, the agent collects the metadata listed [here](/azure/azure-arc/servers/agent-overview#instance-metadata). 

## Next steps

[Connect your System Center Virtual Machine Manager management server to Azure Arc](quickstart-connect-system-center-virtual-machine-manager-to-arc.md).
