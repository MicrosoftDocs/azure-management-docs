---
title: Azure Arc resource bridge system requirements
description: Learn about system requirements for Azure Arc resource bridge.
ms.topic: concept-article
ms.date: 09/09/2026
# Customer intent: "As an IT administrator, I want to understand the system requirements for deploying Azure Arc resource bridge, so that I can ensure proper configuration and prevent errors during deployment."
---

# Azure Arc resource bridge system requirements

This article describes the system requirements for deploying Azure Arc resource bridge.

Use Arc resource bridge with other partner products, such as [Azure Local](/azure/azure-local/manage/azure-arc-vm-management-overview), [Arc-enabled VMware vSphere](../vmware-vsphere/index.yml), and [Arc-enabled System Center Virtual Machine Manager (SCVMM)](../system-center-virtual-machine-manager/index.yml). These products might have additional requirements.

## Required Azure permissions

- To onboard Arc resource bridge, you need the [Contributor](/azure/role-based-access-control/built-in-roles) role for the resource group.
- To read, modify, and delete Arc resource bridge, you need the [Contributor](/azure/role-based-access-control/built-in-roles) role for the resource group.

## Management tool requirements

You need [Azure CLI](/cli/azure/install-azure-cli) to deploy the Azure Arc resource bridge on supported private cloud environments.

If you deploy Arc resource bridge on VMware, you need to install 64-bit Azure CLI on the management machine to run the deployment commands.

If you deploy on Azure Local, install 32-bit Azure CLI on the management machine.

You need to install the Arc appliance CLI extension, `arcappliance`, by running this command: `az extension add --name arcappliance`

## Minimum resource requirements

Arc resource bridge has the following minimum resource requirements:

- 200 GB disk space
- 4 vCPUs
- 8 GB memory
- supported storage configuration - hybrid storage (flash and HDD) or all-flash storage (SSDs or NVMe)

These minimum requirements enable most scenarios for products that use Arc resource bridge. Review the product's documentation for specific resource requirements. Failure to provide sufficient resources might cause errors during deployment or upgrade.

## IP address prefix (subnet) requirements

The IP address prefix (subnet) where you deploy Arc resource bridge requires a minimum prefix of /29. The IP address prefix must have enough available IP addresses for the gateway IP, control plane IP, appliance VM IP, and reserved appliance VM IP. Arc resource bridge only uses the IP addresses assigned to the IP pool range (Start IP, End IP) and the Control Plane IP. We recommend that the End IP immediately follow the Start IP. For example, Start IP = 192.168.0.2, End IP = 192.168.0.3. Work with your network engineer to ensure that there's an available subnet with the required available IP addresses and IP address prefix for Arc resource bridge.

The IP address prefix is the subnet's IP address range for the virtual network and subnet mask (IP Mask) in CIDR notation, for example `192.168.7.1/29`. You provide the IP address prefix (in CIDR notation) during the creation of the configuration files for Arc resource bridge.

Consult your network engineer to obtain the IP address prefix in CIDR notation. You can use an IP Subnet CIDR calculator to obtain this value.

## Static IP configuration

If you deploy Arc resource bridge to a production environment, you must use static configuration when deploying Arc resource bridge. Use static IP configuration to assign three static IPs (that are in the same subnet) to the Arc resource bridge control plane, appliance VM, and reserved appliance VM.

DHCP is only supported in a test environment for testing purposes only for VM management on Azure Local. Don't use it in a production environment. DHCP isn't supported on any other Arc-enabled private cloud, including Arc-enabled VMware, Arc for AVS, or Arc-enabled SCVMM.

If you use DHCP, you must reserve the IP addresses used by the control plane and appliance VM. In addition, these IPs must be outside of the assignable DHCP range of IPs. For example, treat the control plane IP as a reserved/static IP that no other machine on the network uses or receives from DHCP. If the control plane IP or appliance VM IP changes, this change affects the resource bridge availability and functionality.

## Management machine requirements

The machine you use to run the commands that deploy and maintain the Arc resource bridge is called the *management machine*.

Management machine requirements:

- [Azure CLI x64](/cli/azure/install-azure-cli-windows?tabs=azure-cli) installed
- Communication to Control Plane IP (SSH TCP port 22, Kubernetes API port 6443)
- Communication to Appliance VM IPs (SSH TCP port 22, Kubernetes API port 6443)
- Communication to the reserved Appliance VM IPs (SSH TCP port 22, Kubernetes API port 6443)
- Communication over port 443 to the private cloud management console, such as a VMware vCenter machine
- Internal and external DNS resolution. The DNS server must resolve internal names, such as the vCenter endpoint for vSphere or cloud agent service endpoint for Azure Local. The DNS server must also resolve external addresses that are [required URLs](network-requirements.md#outbound-connectivity-requirements) for deployment.
- Internet access

## Appliance VM IP address requirements

Arc resource bridge includes an appliance VM that you deploy on-premises. The appliance VM can see the on-premises infrastructure and tag on-premises resources (guest management) for projection into Azure Resource Manager (ARM). You assign the appliance VM an IP address from the `k8snodeippoolstart` parameter in the `createconfig` command. Partner products might refer to this IP address as Start Range IP, RB IP Start, or VM IP 1. The appliance VM IP is the starting IP address for the appliance VM IP pool range. You initially assign this IP to your appliance VM when you first deploy Arc resource bridge. The VM IP pool range requires a minimum of two IP addresses.

Appliance VM IP address requirements:

- Communicate with the management machine (SSH TCP port 22, Kubernetes API port 6443).
- Communicate with the private cloud management endpoint via port 443 (such as VMware vCenter).
- Internet connectivity to [required URLs](network-requirements.md#outbound-connectivity-requirements) enabled in proxy or firewall.
- Static IP assigned and within the IP address prefix.
- Internal and external DNS resolution.
- If you use a proxy, the proxy server must be reachable from this IP and all IPs within the VM IP pool.

## Reserved appliance VM IP requirements

Arc resource bridge reserves an extra IP address for the appliance VM upgrade. You assign the reserved appliance VM IP an IP address through the `k8snodeippoolend` parameter in the `az arcappliance createconfig` command. Partner products might refer to this IP address as End Range IP, RB IP End, or VM IP 2. The reserved appliance VM IP is the ending IP address for the appliance VM IP pool range. When you upgrade your appliance VM for the first time, you assign the reserved appliance VM IP to your appliance VM. The initial appliance VM IP returns to the IP pool for future upgrades. If you specify an IP pool range larger than two IP addresses, you reserve the extra IPs.

Reserved appliance VM IP requirements:

- Communicate with the management machine (SSH TCP port 22, Kubernetes API port 6443).
- Communicate with the private cloud management endpoint via port 443 (such as VMware vCenter).
- Internet connectivity to [required URLs](network-requirements.md#outbound-connectivity-requirements) enabled in proxy or firewall.
- Static IP assigned and within the IP address prefix.
- Internal and external DNS resolution.
- If you use a proxy, the proxy server must be reachable from this IP and all IPs within the VM IP pool.

## Control plane IP requirements

The appliance VM hosts a management Kubernetes cluster with a control plane that requires a single, static IP address. Assign this IP address from the `controlplaneendpoint` parameter in the `createconfig` command or equivalent configuration files creation command.

Control plane IP requirements:

- Communicate with the management machine (SSH TCP port 22, Kubernetes API port 6443).
- Assign a static IP address within the IP address prefix.
- If you use a proxy, the proxy server must be reachable from IPs within the IP address prefix, including the reserved appliance VM IP.

## DNS server

DNS servers must have internal and external endpoint resolution. The appliance VM and control plane need to resolve the management machine and vice versa. All three IPs must reach the required URLs for deployment. Updating the DNS configuration post-deployment isn't supported and requires [performing a recovery operation](maintenance.md#recovery-procedure).

## Gateway

The gateway IP is the IP of the gateway for the network where you deploy Arc resource bridge. Use an IP from within the subnet designated in the IP address prefix.

## Example minimum configuration for static IP deployment

The following example shows valid configuration values that you can use during configuration file creation for Arc resource bridge.

The IP addresses for the gateway, control plane, appliance VM, and DNS server (for internal resolution) are within the IP address prefix. The VM IP Pool Start and End values are sequential. This key detail helps ensure successful deployment of the appliance VM.

| Setting | Value |
|---|---|
| IP address prefix (CIDR format) | 192.168.0.0/29 |
| Gateway IP | 192.168.0.1 |
| VM IP Pool Start (IP format) | 192.168.0.2 |
| VM IP Pool End (IP format) | 192.168.0.3 |
| Control Plane IP | 192.168.0.4 |
| DNS servers (IP list format) | 192.168.0.1, 10.0.0.5, 10.0.0.6 |

## User account and credentials

Arc resource bridge might require a dedicated user account with the necessary roles to view and manage resources in the on-premises private cloud. If so, provide the `username` and `password` parameters when you create the configuration files. The account credentials are stored as a secret within the appliance VM.

> [!WARNING]
> Arc resource bridge can only use a user account that doesn't have multifactor authentication enabled. If the user account is set to periodically change passwords, [you must immediately update the credentials on the resource bridge](maintenance.md#update-credentials-in-the-appliance-vm). You can also set a lockout policy for this user account to protect the on-premises infrastructure if the credentials aren't updated and the resource bridge makes multiple attempts to use expired credentials to access the on-premises control center.

For example, with Arc-enabled VMware, Arc resource bridge needs a dedicated user account for vCenter with the necessary roles. If the [credentials for the user account change](troubleshoot-resource-bridge.md#insufficient-privileges), then the credentials stored in Arc resource bridge must be immediately updated by running `az arcappliance update-infracredentials` from the [management machine](#management-machine-requirements). Otherwise, the appliance makes repeated attempts to use the expired credentials to access vCenter, which can result in a lockout of the account.

## Internal Certificates

Arc resource bridge contains internal certificates that are required to maintain secure communication to Azure and verify internal components. These certificates require that Arc resource bridge remain online and maintain a persistent connection to Azure. If Arc resource bridge is offline for greater than 45 days, there is a risk that the certificate will expire, requiring a redeployment as the certificate is irrecoverable. Arc resource bridge also requires an upgrade once every six months to ensure that internal certificates are refreshed. If Arc resource bridge is unable to upgrade and the certificates expire, then a redeployment is required. Please review the [Maintenance page](maintenance.md) for important information to maintain your Arc resource bridge.

## Configuration files

Arc resource bridge uses an appliance VM that you deploy in your on-premises infrastructure. To maintain the appliance VM, save the configuration files generated during deployment in a secure location and ensure they're available on the management machine.

The types of configuration files vary based on the on-premises infrastructure.

### Appliance configuration files

You create three configuration files when you deploy the Arc resource bridge: `<appliance-name>-resource.yaml`, `<appliance-name>-appliance.yaml`, and `<appliance-name>-infra.yaml`.

By default, the deployment commands generate these files in the current CLI directory. Save these files on the management machine because you need them to maintain the appliance VM. The configuration files reference each other and should be stored in the same location.

The `az arcappliance` CLI commands that rely on the YAML configuration files are `az arcappliance delete` to delete the Arc resource bridge and its Azure backend associations, and `az arcappliance upgrade` to manually upgrade the Arc resource bridge.


### Kubeconfig

The appliance VM hosts a management Kubernetes cluster. The kubeconfig is a low-privilege Kubernetes configuration file that you use to maintain the appliance VM. By default, the deployment generates it in the current CLI directory when the `deploy` command completes. Save the kubeconfig in a secure location on the management machine because you need it to maintain the appliance VM. If you lose the kubeconfig, retrieve it by running the `az arcappliance get-credentials` command.

> [!IMPORTANT]
> After you create the Arc resource bridge VM, you can only update the proxy settings. To update other configuration settings, such as DNS and the resource bridge VM location path, [perform a recovery](maintenance.md#recovery-procedure). The Arc resource bridge VM name is a unique GUID that you can't rename after deployment.

## Next steps

- Understand [network requirements for Azure Arc resource bridge](network-requirements.md).
- Review the [Azure Arc resource bridge overview](overview.md) to learn more about features and benefits.
- Learn about [security configuration and considerations for Azure Arc resource bridge](security-overview.md).
