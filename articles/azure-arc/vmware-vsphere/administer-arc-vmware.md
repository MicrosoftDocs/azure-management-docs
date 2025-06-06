---
title: Perform ongoing administration for Arc-enabled VMware vSphere
description: Learn how to perform administrator operations related to Azure Arc-enabled VMware vSphere
ms.topic: how-to 
ms.date: 10/24/2024
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
ms.custom:
  - devx-track-azurecli
  - build-2025
---

# Perform ongoing administration for Arc-enabled VMware vSphere

In this article, you learn how to perform various administrative operations related to Azure Arc-enabled VMware vSphere:

- Upgrading the Azure Arc resource bridge
- Updating the credentials
- Collecting logs from the Arc resource bridge

Each of these operations requires either SSH key to the resource bridge VM or the kubeconfig that provides access to the Kubernetes cluster on the resource bridge VM.

## Upgrade the Arc resource bridge manually

Azure Arc-enabled VMware vSphere requires the Arc resource bridge to connect your vSphere environment with Azure. Periodically, new images of Arc resource bridge are released to include security and feature updates. The Arc resource bridge can be manually upgraded from the vCenter server. You must meet all upgrade [prerequisites](../resource-bridge/upgrade.md#prerequisites) before attempting to upgrade. The vCenter server must have the kubeconfig and appliance configuration files stored locally. If the vSphere account credentials changed after the initial deployment of the resource bridge, [update the new account credentials](administer-arc-vmware.md#updating-the-vsphere-account-credentials-using-a-new-password-or-a-new-vsphere-account-after-onboarding) before attempting manual upgrade.

The manual upgrade generally takes between 30-90 minutes, depending on the network speed. The upgrade command takes your Arc resource bridge to the immediate next version, which might not be the latest available version. Multiple upgrades could be needed to reach a [supported version](../resource-bridge/upgrade.md#supported-versions). You can check your resource bridge version by checking the Azure resource of your Arc resource bridge.

To manually upgrade your Arc resource bridge, make sure you've installed the latest `az arcappliance` CLI extension by running the extension upgrade command from the vCenter server:

```azurecli
az extension add --upgrade --name arcappliance 
```

To manually upgrade your resource bridge, use the following command:

```azurecli
az arcappliance upgrade vmware --config-file <file path to ARBname-appliance.yaml> 
```

## Updating the vSphere account credentials (using a new password or a new vSphere account after onboarding)

Azure Arc-enabled VMware vSphere uses the vSphere account credentials you provided during the onboarding to communicate with your vCenter server. These credentials are only persisted locally on the Arc resource bridge VM.

As part of your security practices, you might need to rotate credentials for your vCenter accounts. As credentials are rotated, you must also update the credentials provided to Azure Arc to ensure the functioning of Azure Arc-enabled VMware services. You can also use the same steps in case you need to use a different vSphere account after onboarding. You must ensure the new account also has all the [required vSphere permissions](support-matrix-for-arc-enabled-vmware-vsphere.md#required-vsphere-account-privileges).

There are two different sets of credentials stored on the Arc resource bridge. You can use the same account credentials for both.

- **Account for Arc resource bridge**. This account is used for deploying the Arc resource bridge VM and will be used for upgrade.
- **Account for VMware cluster extension**. This account is used to discover inventory and perform all VM operations through Azure Arc-enabled VMware vSphere.

To update the credentials of the account for Arc resource bridge, run the following Azure CLI commands. Run the commands from a workstation that can access cluster configuration IP address of the Arc resource bridge locally:

```azurecli
az account set -s <subscription id>
az arcappliance get-credentials -n <name of the appliance> -g <resource group name> 
az arcappliance update-infracredentials vmware --kubeconfig kubeconfig
```
For more information on the commands, see [`az arcappliance get-credentials`](/cli/azure/arcappliance#az-arcappliance-get-credentials) and [`az arcappliance update-infracredentials vmware`](/cli/azure/arcappliance/update-infracredentials#az-arcappliance-update-infracredentials-vmware).


To update the credentials used by the VMware cluster extension on the resource bridge. This command can be run from anywhere with `connectedvmware` CLI extension installed.

```azurecli
az connectedvmware vcenter connect --custom-location <name of the custom location> --location <Azure region>  --name <name of the vCenter resource in Azure>       --resource-group <resource group for the vCenter resource>  --username   <username for the vSphere account>  --password  <password to the vSphere account>
```

## Collecting logs from the Arc resource bridge

For any issues encountered with the Azure Arc resource bridge, you can collect logs for further investigation. To collect the logs, use the Azure CLI [`Az arcappliance log`](/cli/azure/arcappliance/logs#az-arcappliance-logs-vmware) command.

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
