---
title:  Perform ongoing maintenance and administration for Azure Arc-enabled System Center Virtual Machine Manager
description: Learn how to perform administrator operations related to Azure Arc-enabled System Center Virtual Machine Manager.
ms.topic: how-to 
ms.date: 04/08/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.custom:
  - devx-track-azurecli
  - build-2025
ms.author: jsuri
author: jyothisuri
# Customer intent: "As a system administrator managing Azure Arc-enabled SCVMM, I want to perform maintenance and update account credentials for the Arc resource bridge, so that I can ensure secure and reliable connectivity between on-premises servers and Azure services."
---

# Perform ongoing maintenance and administration for Azure Arc-enabled System Center Virtual Machine Manager

In this article, you learn how to perform various maintenance and administrative operations related to Azure Arc-enabled System Center Virtual Machine Manager (SCVMM) such as:

- [Maintain the Azure Arc resource bridge manually by following the best practices](#best-practices-to-maintain-the-azure-arc-enabled-scvmm-resources).
- [Update the SCVMM account credentials](#update-the-scvmm-account-credentials-using-a-new-password-or-a-new-scvmm-account-after-onboarding).
- [Collect logs from the Azure Arc resource bridge](#collect-logs-from-the-arc-resource-bridge).

## Best practices to maintain the Azure Arc-enabled SCVMM resources 

The Azure Arc resource bridge establishes line of sight between the on-premises SCVMM management server and Azure. The components of the resource bridge make it possible to bring the goodness of Azure to your on-premises SCVMM managed virtual machines.

The following are a few best practices that you can follow as it deems fit for your organization: 

- **Securely maintain the `.yaml` and `kubeconfig` files**: After the successful deployment of the Azure Arc resource bridge, three configuration files are created: `\<resource-bridge-name\>-resource.yaml`, `\<resource-bridge-name\>-appliance.yaml` and `\<resource-bridge-name\>-infra.yaml`. The resource bridge VM hosts a management Kubernetes cluster. By default, a `kubeconfig` file which will be used to maintain the resource bridge VM, is generated in the current CLI directory during the resource bridge deployment process. It's mandatory to securely store and maintain these files since they're required for the management and upgrade of the resource bridge. 

- **Resource bridge lock**: Considering the critical nature of the resource bridge, you, as an administrator, can lock the Azure resource of the resource bridge to protect it from accidental deletion and hence preventing loss of connectivity to SCVMM server from Azure. To place a resource lock on your resource bridge, navigate to its Azure resource and select **Locks** under the **Settings** blade. You can add a **Delete** lock from here which prevents self-service users from accidentally deleting the resource bridge. 

     >[!IMPORTANT]
     >Both Read-only and Delete type resource locks disable the upgrade of the resource bridge and it is necessary to remove the locks before triggering an upgrade. 

- **Resource bridge health alert**: The Azure Arc resource bridge needs to be maintained online and healthy with a *Running* status to ensure the continuous functioning of the Azure Arc enabled SCVMM offering. The resource bridge can periodically get into an *offline* status due to rotation of credentials as part of your regular security practices or any change in the network. To be notified of any unintended downtime of the resource bridge, you can set up health alerts on the Azure resource of your resource bridge by following these steps: 

     1. Sign in to the [Azure portal](https://portal.azure.com), search and navigate to **Service Health**.
     1. In the left pane, select **Resource Health** > **Resource Health**.
     1. In the **Subscription** dropdown, choose the subscription in which your resource bridge is located. In the **Resource type** dropdown, select **Azure Arc Resource Bridge**. After the list of resource bridge is populated, choose the resource bridge for which you want to set up the alert and select **Add resource health alert**. If you want to set up alerts for all the resource bridges in your subscription, you can select **Add resource health alert** without choosing any resource bridges. This adds health alerts for any resource bridges you may deploy in the future.<br>
           :::image type="content" source="media/administer-arc-scvmm/service-health.png" alt-text="Screenshot of Service Health.":::
     1. Configure the conditions in the alert rule depending on if you want to receive continuous notifications on the health status or if you want to receive notifications only when the resource bridge becomes unhealthy. To receive notifications only when the resource bridge becomes unhealthy, set the following conditions in the **Condition** tab:  
           - **Event status**: Active 
           - **Current resource status**: Unavailable 
           - **Previous resource status**: Available  
     1. In the **Actions** tab, configure the action group with the type of notification and the recipient of the alert. 
     1. Complete creating the alert rule by filling in the details of the alert rule location, identifiers, and optional tags.
       
        Alternatively, you can create health alert from the Azure resource of your resource bridge.<br>
        :::image type="content" source="media/administer-arc-scvmm/resource-health.png" alt-text="Screenshot of resource health."::: 

## Update the SCVMM account credentials (using a new password or a new SCVMM account after onboarding)

Azure Arc-enabled SCVMM uses the SCVMM account credentials you provided during the onboarding to communicate with your SCVMM management server. These credentials are stored locally on the Arc resource bridge VM.

As part of your security practices, you might need to rotate credentials for your SCVMM accounts. As credentials are rotated, you must also update the credentials provided to Azure Arc to ensure the functioning of Azure Arc-enabled SCVMM. You can also use the same steps in case you need to use a different SCVMM account after onboarding. You must ensure the new account also has all the [required SCVMM permissions](quickstart-connect-system-center-virtual-machine-manager-to-arc.md#prerequisites).

There are two different sets of credentials stored on the Arc resource bridge. You can use the same account credentials for both.

### [Account for Arc resource bridge](#tab/account-for-arc-resource-bridge)

This account is used for deploying the Arc resource bridge VM and will be used for upgrade.

**Update the credentials of the account for Arc resource bridge**

To update the credentials of the account for Arc resource bridge, run the following Azure CLI commands. Run the commands from a workstation that can access cluster configuration IP address of the Arc resource bridge locally:

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance update-infracredentials scvmm --kubeconfig kubeconfig
```
For more information on the commands, see [`az arcappliance get-credentials`](/cli/azure/arcappliance#az-arcappliance-get-credentials) and [`az arcappliance update-infracredentials scvmm`](/cli/azure/arcappliance/update-infracredentials#az-arcappliance-update-infracredentials-scvmm).

### [Account for SCVMM cluster extension](#tab/account-for-scvmm-cluster-extension)

This account is used to discover inventory and perform all the VM operations through Azure Arc-enabled SCVMM.

**Update the credentials used by the SCVMM cluster extension**

To update the credentials used by the SCVMM cluster extension on the resource bridge, run the following command. This command can be run from anywhere with the `connectedscvmm` CLI extension installed.

```azurecli
az scvmm vmmserver connect --custom-location <name of the custom location> --location <Azure region>  --name <name of the SCVMM resource in Azure>       --resource-group <resource group for the SCVMM resource>  --username   <username for the SCVMM account>  --password  <password to the SCVMM account>
```
---

## Collect logs from the Arc resource bridge

For any issues encountered with the Azure Arc resource bridge, you can collect logs for further investigation. To collect the logs, use the Azure CLI [`Az arcappliance log`](/cli/azure/arcappliance/logs#az-arcappliance-logs-scvmm) command.

To save the logs to a destination folder, run the following commands. These commands need connectivity to cluster configuration IP address.

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance logs scvmm --kubeconfig kubeconfig --out-dir <path to specified output directory>
```

If the Kubernetes cluster on the Azure Arc resource bridge isn't in functional state, you can use the following commands. These commands require connectivity to IP address of the Azure Arc resource bridge VM via SSH.

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance logs scvmm --out-dir <path to specified output directory> --ip XXX.XXX.XXX.XXX
```

## Next steps

- [Troubleshoot common issues related to resource bridge](../resource-bridge/troubleshoot-resource-bridge.md).
- [Understand disaster recovery operations for resource bridge](./disaster-recovery.md).
- [Azure Arc resource bridge maintenance operations](../resource-bridge/maintenance.md).
- [Protect your Azure resource with a lock](/azure/azure-resource-manager/management/lock-resources?tabs=json).