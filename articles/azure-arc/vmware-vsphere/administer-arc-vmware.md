---
title: Perform ongoing maintenance and administration for Azure Arc-enabled VMware vSphere
description: Learn how to perform ongoing maintenance and administrator operations related to Azure Arc-enabled VMware vSphere
ms.topic: how-to
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - devx-track-azurecli
  - build-2025
# Customer intent: "As a cloud administrator, I want to perform ongoing management of the Arc resource bridge for VMware vSphere, so that I can ensure connectivity and functionality between my vSphere environment and Azure services."
---

# Perform ongoing maintenance and administration for Azure Arc-enabled VMware vSphere

In this article, you learn how to perform various ongoing maintenance and administrative operations related to Azure Arc-enabled VMware vSphere, such as:

- [Maintain the Azure Arc resource bridge manually by following the best practices](#best-practices-to-maintain-the-azure-arc-enabled-vmware-vsphere-resources).
- [Update the vSphere account credentials](#update-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding).
- [Collect logs from the Arc resource bridge](#collect-logs-from-the-arc-resource-bridge).

Each operation requires either an SSH key to the resource bridge VM or the kubeconfig that provides access to the Kubernetes cluster on the resource bridge VM.

## Best practices to maintain the Azure Arc-enabled VMware vSphere resources 

The Azure Arc resource bridge establishes line of sight between the on-premises vCenter server and Azure. The components of the resource bridge make it possible to bring the goodness of Azure to your on-premises vCenter managed virtual machines. 

Follow these best practices as they fit your organization: 

- **Securely maintain the `.yaml` and `kubeconfig` files**: After you successfully deploy the Azure Arc resource bridge, the deployment process creates three configuration files: `\<resource-bridge-name\>-resource.yaml`, `\<resource-bridge-name\>-appliance.yaml`, and `\<resource-bridge-name\>-infra.yaml`. The resource bridge VM hosts a management Kubernetes cluster. By default, the deployment process generates a `kubeconfig` file in the current CLI directory. Use this file to maintain the resource bridge VM. Securely store and maintain these files because you need them to manage and upgrade the resource bridge. 

- **Resource bridge lock**: Considering the critical nature of the resource bridge, you, as an administrator, can lock the Azure resource of the resource bridge to protect it from accidental deletion and hence preventing loss of connectivity to vCenter server from Azure. To place a resource lock on your resource bridge, navigate to its Azure resource and select **Locks** under the Settings blade. You can add a **Delete** lock from here, which prevents self-service users from accidentally deleting the resource bridge. 

- **Resource bridge health alert**: Maintain the Azure Arc resource bridge online and healthy with a *Running* status to ensure the continuous functioning of the Azure Arc enabled VMware vSphere offering. The resource bridge can periodically go into an *offline* status due to rotation of credentials as part of your regular security practices or any change in the network. To be notified of any unintended downtime of the resource bridge, set up health alerts on the Azure resource of your resource bridge by following these steps: 

     1. Sign in to the [Azure portal](https://portal.azure.com/), search and navigate to **Service Health**
     1. In the left pane, select **Resource Health** > **Resource Health**.
     1. In the **Subscription** dropdown, choose the subscription where your resource bridge is located. In the **Resource type** dropdown, select **Azure Arc Resource Bridge**. After the list of resource bridge is populated, choose the resource bridge for which you want to set up the alert and select **Add resource health alert**. If you want to set up alerts for all the resource bridges in your subscription, select **Add resource health alert** without choosing any resource bridges. This step adds health alerts for any resource bridges you might deploy in the future.<br>
           :::image type="content" source="media/administer-arc-vmware/service-health.png" alt-text="Screenshot of Service Health.":::
     1. Configure the conditions in the alert rule depending on if you want to receive continuous notifications on the health status or if you want to receive notifications only when the resource bridge becomes unhealthy. To receive notifications only when the resource bridge becomes unhealthy, set the following conditions in the **Condition** tab: 
           - **Event status**: Active 
           - **Current resource status**: Unavailable 
           - **Previous resource status**: Available 
     1. In the **Actions** tab, configure the action group with the type of notification and the recipient of the alert. 
     1. Complete creating the alert rule by filling in the details of the alert rule location, identifiers, and optional tags. 
 
        Alternatively, you can create health alert from the Azure resource of your resource bridge. 
        :::image type="content" source="media/administer-arc-vmware/resource-health.png" alt-text="Screenshot of resource health."::: 

- **Avoid unsupported VM operations on the Azure Arc resource bridge VM**:  
     - Backup and restore of the resource bridge isn't supported and affects the connectivity between vCenter and Azure if attempted. 
     - Network migration of the resource bridge isn't supported as it might involve changing the resource bridge VM’s virtual network or subnet, reassigning the VM’s IP address, or network interfaces. 

## Update the vSphere account credentials (use a new password or a new vSphere account after onboarding)

Azure Arc-enabled VMware vSphere uses the vSphere account credentials you provide during onboarding to communicate with your vCenter server. The Arc resource bridge VM persists these credentials only locally.

As part of your security practices, you might need to rotate credentials for your vCenter accounts. As you rotate credentials, update the credentials you provide to Azure Arc to ensure Azure Arc-enabled VMware vSphere continues functioning. You can also use the same steps if you need to use a different vSphere account after onboarding. Make sure the new account has all the [required vSphere permissions](support-matrix-for-arc-enabled-vmware-vsphere.md#required-vsphere-account-privileges).

The Arc resource bridge stores two different sets of credentials. You can use the same account credentials for both.

### [Account for Arc resource bridge](#tab/account-for-arc-resource-bridge)

Use this account to deploy the Arc resource bridge VM. It also serves as the upgrade account.

**Update the credentials for the account for Arc resource bridge**

Run the following Azure CLI commands to update the credentials for the account for Arc resource bridge. Run the commands from a workstation that can access the cluster configuration IP address of the Arc resource bridge locally:

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance update-infracredentials vmware --kubeconfig kubeconfig
```
For more information on the commands, see [`az arcappliance get-credentials`](/cli/azure/arcappliance#az-arcappliance-get-credentials) and [`az arcappliance update-infracredentials scvmm`](/cli/azure/arcappliance/update-infracredentials#az-arcappliance-update-infracredentials-scvmm).

### [Account for VMware cluster extension](#tab/account-for-vmware-cluster-extension)

Use this account to discover inventory and perform all the VM operations through Azure Arc-enabled VMware vSphere.

**Update the credentials used by the VMware cluster extension**

To update the credentials used by the VMware cluster extension on the resource bridge. This command can be run from anywhere with the `connectedvmware` CLI extension installed.

```azurecli
az connectedvmware vcenter connect --custom-location <name of the custom location> --location <Azure region>  --name <name of the vCenter resource in Azure>       --resource-group <resource group for the vCenter resource>  --username   <username for the vSphere account>  --password  <password to the vSphere account> 
```
---

## Collect logs from the Arc resource bridge

If you encounter any issues with the Azure Arc resource bridge, collect logs for further investigation. To collect the logs, use the Azure CLI [`Az arcappliance log`](/cli/azure/arcappliance/logs#az-arcappliance-logs-vmware) command.

To save the logs to a destination folder, run the following commands. These commands need connectivity to cluster configuration IP address.

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance logs vmware --kubeconfig kubeconfig --out-dir <path to specified output directory>
```

If the Kubernetes cluster on the resource bridge isn't in functional state, you can use the following commands. These commands require connectivity to IP address of the Azure Arc resource bridge VM via SSH

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance logs vmware --out-dir <path to specified output directory> --ip XXX.XXX.XXX.XXX
```

## Next steps

- [Troubleshoot common issues related to resource bridge](../resource-bridge/troubleshoot-resource-bridge.md)
- [Understand disaster recovery operations for resource bridge](recover-from-resource-bridge-deletion.md)
