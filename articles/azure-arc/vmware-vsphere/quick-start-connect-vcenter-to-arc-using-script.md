---
title: Connect VMware vCenter Server to Azure Arc by using the helper script
description: In this quickstart, you learn how to use the helper script to connect your VMware vCenter Server instance to Azure Arc.
ms.topic: quickstart 
ms.custom:
  - references_regions
  - build-2025
ms.date: 04/08/2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
author: Jeronika-MS
# Customer intent: As a VI admin, I want to connect my vCenter Server instance to Azure to enable self-service through Azure Arc.
---

# Quickstart: Connect VMware vCenter Server to Azure Arc by using the helper script

To start using the Azure Arc-enabled VMware vSphere features, you need to connect your VMware vCenter Server instance to Azure Arc. This quickstart shows you how to connect your VMware vCenter Server instance to Azure Arc by using a helper script.

First, the script deploys a virtual appliance called [Azure Arc resource bridge](../resource-bridge/overview.md) in your vCenter environment. Then, it installs a VMware cluster extension to provide a continuous connection between vCenter Server and Azure Arc.

> [!IMPORTANT]
> This article describes a way to connect a generic vCenter Server to Azure Arc. If you're trying to enable Arc for Azure VMware Solution (AVS) private cloud, follow this guide instead - [Deploy Arc-enabled VMware vSphere for Azure VMware Solution private cloud](/azure/azure-vmware/deploy-arc-for-azure-vmware-solution). With the Arc for AVS onboarding process you need to provide fewer inputs and Arc capabilities are better integrated into the AVS private cloud portal experience. 

## Prerequisites

### Azure

- An Azure subscription.

- A resource group in the subscription where you have the *Owner*, *Contributor*, or *Azure Arc VMware Private Clouds Onboarding* role for onboarding.

### Azure Arc Resource Bridge

- Azure Arc resource bridge IP needs access to the URLs listed [here](../vmware-vsphere/support-matrix-for-arc-enabled-vmware-vsphere.md#resource-bridge-networking-requirements).

### vCenter Server

- vCenter Server version 7 or 8.

- A virtual network that can provide internet access, directly or through a proxy. It must also be possible for VMs on this network to communicate with the vCenter server on TCP port (usually 443).

- At least three free static IP addresses on the above network.

- A resource pool or a cluster with a minimum capacity of 8 GB of RAM and 4 vCPUs.

- A datastore with a minimum of 200 GB of free disk space or 400 GB for High Availability deployment, available through the resource pool or cluster.

> [!NOTE]
> Azure Arc-enabled VMware vSphere supports vCenter Server instances with a maximum of 9,500 virtual machines (VMs). If your vCenter Server instance has more than 9,500 VMs, we don't recommend that you use Azure Arc-enabled VMware vSphere with it at this point.

### vSphere account

You need a vSphere account that can:
- Read all inventory. 
- Deploy and update VMs to all the resource pools (or clusters), networks, and VM templates that you want to use with Azure Arc.

> [!IMPORTANT]
> As part of the Azure Arc-enabled VMware onboarding script, you will be prompted to provide a vSphere account to deploy the Azure Arc resource bridge VM on the ESXi host. This account will be stored locally within the Azure Arc resource bridge VM and encrypted as a Kubernetes secret at rest. The vSphere account allows Azure Arc-enabled VMware to interact with VMware vSphere. If your organization practices routine credential rotation, you must [update the credentials in Azure Arc-enabled VMware](administer-arc-vmware.md#update-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding) to maintain the connection between Azure Arc-enabled VMware and VMware vSphere.


### Workstation

You need a Windows or Linux machine that can access both your vCenter Server instance and the internet, directly or through a proxy. The workstation must also have outbound network connectivity to the ESXi host backing the datastore. Datastore connectivity is needed for uploading the Arc resource bridge image to the datastore as part of the onboarding.    

## Prepare vCenter Server

1. Create a resource pool with a reservation of at least 16 GB of RAM and four vCPUs. It should also have access to a datastore with at least 100 GB of free disk space.

2. Ensure that the vSphere accounts have the appropriate permissions.

## Download the onboarding script

1. Go to [Azure portal](https://aka.ms/SCVMM/MgmtServers).
2. Search and select **Azure Arc**.
3. In the **Overview** page, select **Add resources** under **Manage resources across environments**.

     :::image type="content" source="media/quick-start-connect-vcenter-to-arc-using-script/add-vmware-vcenter.png" alt-text="Screenshot that shows how to add VMware vCenter through Azure Arc.":::

4. In the **Host environments** section, in **VMware vSphere** select **Add**.

    :::image type="content" source="media/quick-start-connect-vcenter-to-arc-using-script/platform-add-vmware-vsphere.png" alt-text="Screenshot of how to select System Center V M M platform." lightbox="media/quick-start-connect-vcenter-to-arc-using-script/platform-add-vmware-vsphere.png":::

5. Select **Create a new resource bridge** and select **Next : Basics >**.
6. Provide a name for **Azure Arc resource bridge**. For example: *contoso-nyc-resourcebridge*.
7. Select a subscription and resource group where you want to create the resource bridge.
8. Under **Region**, select an Azure location where you want to store the resource metadata.
9. Provide a name for **Custom location**. This is the name that you'll see when you deploy virtual machines. Name it for the datacenter or the physical location of your datacenter. For example: *contoso-nyc-dc.*
10. Leave the option **Use the same subscription and resource group as your resource bridge** selected.
11. Provide a name for your vCenter Server instance in Azure. For example: **contoso-nyc-vcenter**.
12. You can choose to **Enable Kubernetes Service on VMware [Preview]**. If you choose to do so, ensure you update the namespace of your custom location to "default" in the onboarding script: $customLocationNamespace = ("default".ToLower() -replace '[^a-z0-9-]', ''). For more information about this update, refer the [known issues from AKS on VMware (preview)](/azure/aks/hybrid/aks-vmware-known-issues)
13. Select **Next: Tags >**.
14. Assign Azure tags to your resources in **Value** under **Physical location tags**. You can add additional tags to help you organize your resources to facilitate administrative tasks using custom tags.
15. Select **Next: Download and run script**.
16. If your subscription isn't registered with all the required resource providers, a **Register** button will appear. Select the button before you proceed to the next step.

    :::image type="content" source="media/quick-start-connect-vcenter-to-arc-using-script/register-arc-vmware-providers.png" alt-text="Screenshot that shows the button to register required resource providers during vCenter onboarding to Azure Arc.":::

17. Based on the operating system of your workstation, download the PowerShell or Bash script and copy it to the [workstation](#prerequisites).

## Run the script

Use the following instructions to run the script, depending on which operating system your machine is using.

### Windows

1. Open a PowerShell window as an Administrator and go to the folder where you've downloaded the PowerShell script.

    > [!NOTE]
    > On Windows workstations, the script must be run in PowerShell window and not in PowerShell Integrated Script Editor (ISE) as PowerShell ISE doesn't display the input prompts from Azure CLI commands. If the script is run on PowerShell ISE, it could appear as though the script is stuck while it is waiting for input. 

2. Run the following command to allow the script to run, because it's an unsigned script. (If you close the session before you complete all the steps, run this command again for the new session.)

    ``` powershell-interactive
    Set-ExecutionPolicy -Scope Process -ExecutionPolicy Bypass
    ```

3. Run the script:

     ``` powershell-interactive
     ./resource-bridge-onboarding-script.ps1
     ```

### Linux

1. Open the terminal and go to the folder where you've downloaded the Bash script.

2. Run the script by using the following command:

    ``` sh
    bash resource-bridge-onboarding-script.sh
    ```

## Inputs for the script

A typical onboarding that uses the script takes 30 to 60 minutes. During the process, you're prompted for the following details:

| **Requirement** | **Details** |
| --- | --- |
| **Azure login** | When you're prompted, go to the [device sign-in page](https://www.microsoft.com/devicelogin), enter the authorization code shown in the terminal, and sign in to Azure. |
| **vCenter FQDN/Address** | Enter the fully qualified domain name for the vCenter Server instance (or an IP address). For example: **10.160.0.1** or **nyc-vcenter.contoso.com**. |
| **vCenter Username** | Enter the username for the vSphere account. The required permissions for the account are listed in the [prerequisites](#prerequisites). |
| **vCenter password** | Enter the password for the vSphere account. |
| **Data center selection** | Select the name of the datacenter (as shown in the vSphere client) where the Azure Arc resource bridge VM should be deployed. |
| **Network selection** | Select the name of the virtual network or segment to which the Azure Arc resource bridge VM must be connected. This network should allow the appliance to communicate with vCenter Server and the Azure endpoints (or internet). |
| **Static IP** | Arc Resource Bridge requires static IP address assignment and DHCP isn't supported. </br> 1. **Static IP address prefix**: Network address in CIDR notation. For example: **192.168.0.0/24**. </br> 2. **Static gateway**: Gateway address. For example: **192.168.0.0**. </br> 3. **DNS servers**: IP address(es) of DNS server(s) used by Azure Arc resource bridge VM for DNS resolution. Azure Arc resource bridge VM must be able to resolve external sites, like mcr.microsoft.com and the vCenter server. </br> 4. **Start range IP**: Minimum size of two available IP addresses is required. One IP address is for the Azure Arc resource bridge VM, and the other is reserved for upgrade scenarios. Provide the starting IP address of that range. Ensure the Start range IP has internet access. </br> 5. **End range IP**: Last IP address of the IP range requested in the previous field. Ensure the End range IP has internet access. </br>|
| **Control Plane IP address** | Azure Arc resource bridge runs a Kubernetes cluster, and its control plane always requires a static IP address. Provide an IP address that meets the following requirements:  <br> - The IP address must have internet access. <br> - The IP address must be within the subnet defined by IP address prefix. <br> - If you're using static IP address option for resource bridge VM IP address, the control plane IP address must be outside of the IP address range provided for the VM (Start range IP - End range IP). |
| **Resource pool** | Select the name of the resource pool to which the Azure Arc resource bridge VM will be deployed. |
| **Data store** | Select the name of the datastore to be used for the Azure Arc resource bridge VM. |
| **Folder** | Select the name of the vSphere VM and the template folder where the Azure Arc resource bridge's VM will be deployed. |
| **Appliance proxy settings** | Enter **y** if there's a proxy in your appliance network. Otherwise, enter **n**. </br> You need to populate the following boxes when you have a proxy set up: </br> 1. **Http**: Address of the HTTP proxy server. </br> 2. **Https**: Address of the HTTPS proxy server. </br> 3. **NoProxy**: Addresses to be excluded from the proxy. </br> 4. **CertificateFilePath**: For SSL-based proxies, the path to the certificate to be used.

After the command finishes running, your setup is complete. You can now use the capabilities of Azure Arc-enabled VMware vSphere.

> [!IMPORTANT]
> After the successful installation of Azure Arc Resource Bridge, it's recommended to retain a copy of the resource bridge config.yaml files in a place that facilitates easy retrieval. These files could be needed later to run commands to perform management operations (e.g. [az arcappliance upgrade](/cli/azure/arcappliance/upgrade#az-arcappliance-upgrade-vmware)) on the resource bridge. You can find the three .yaml files (config files) in the same folder where you ran the script. 

## Recovering from failed deployments

If the Azure Arc resource bridge deployment fails, consult the [Azure Arc resource bridge troubleshooting document](../resource-bridge/troubleshoot-resource-bridge.md). While there can be many reasons why the Azure Arc resource bridge deployment fails, one of them is KVA timeout error. For more information about the KVA timeout error and how to troubleshoot it, see [KVA timeout error](../resource-bridge/troubleshoot-resource-bridge.md#kva-timeout-error).

To clean up the installation and retry the deployment, use the following commands.

### Retry command - Windows

Run the command with ```-Force``` to clean up the installation and onboard again.

```powershell-interactive
./resource-bridge-onboarding-script.ps1 -Force
```

### Retry command - Linux

Run the command with ```--force``` to clean up the installation and onboard again.
```bash
bash resource-bridge-onboarding-script.sh --force
```    

## Next steps

- [Browse and enable VMware vCenter resources in Azure](browse-and-enable-vcenter-resources-in-azure.md)
