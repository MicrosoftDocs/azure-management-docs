---
title: Perform disaster recovery operations
description: Learn how to perform recovery operations for the Azure Arc resource bridge VM in Azure Arc-enabled VMware vSphere disaster scenarios.
ms.topic: how-to 
ms.custom:
ms.date: 10/24/2024
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
author: PriskeyJeronika-MS
ms.author: v-gjeronika
manager: jsuri
---

# Recover Arc resource bridge connection

In this article, you learn how to restore the Azure Arc resource bridge connection to a working state in case of an accidental deletion of the resource bridge VM or if the resource bridge is irrecoverable.

## Recover the Arc resource bridge

The connection between on-premises infrastructure and Azure can be lost and any operations performed through Azure Arc can fail in case of the following disaster scenarios:

- Accidental deletion of the VM
- VM connection failure (for example, due to changes in the network configuration of the infrastructure)
- VM upgrade failures which can’t be fixed and need redeployment

- Expiration of the VM certs due to lack of upgrades

In such disaster scenarios, you can restore operations by deploying a new resource bridge with the same resource ID as the current resource bridge following the steps below:

1. To prepare for deploying a new resource bridge to replace the current resource bridge, find and keep a copy of the following properties from the current resource bridge that will be re-used for the new resource bridge: 

1. Azure region, Arc resource bridge ARM resource ID, custom location resource ID, resource ID of the vCenter Azure resource. 

1. Go to Azure Portal, search for your resource bridge VM and delete the resource bridge ARM resource from Azure Portal. This is a necessary step as part of the disaster recovery process because we will deploy a new resource bridge with the same properties to replace this one. All the other related components, like custom location, vCenter resource, in Azure Portal and any other Azure resources should remain in Azure Portal. 

1. Go to your vCenter console and delete the Azure Arc resource bridge VM from the vCenter if it exists. The VM will be re-created in a later step as part of the disaster recovery.

1. Download the [onboarding script](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#download-the-onboarding-script) from the Azure portal. You will need to make changes to this script as this is the day 0 deployment script and changes must be made to support the disaster recovery scenario.

1. Open the onboarding script in an editor and update the following section in the script with the properties from the current Arc resource bridge that you copied in Step 1: Azure region, Arc resource bridge ARM resource ID, custom location resource ID, and resource ID of the vCenter Azure resource.

       ```powershell
    $location = <Azure region of the resources>
    $applianceSubscriptionId = <subscription-id>
    $applianceResourceGroupName = <resource-group-name>
    $applianceName = <resource-bridge-name>
    
    $customLocationSubscriptionId = <subscription-id>
    $customLocationResourceGroupName = <resource-group-name>
    $customLocationName = <custom-location-name>
    
    $vCenterSubscriptionId = <subscription-id>
    $vCenterResourceGroupName = <resource-group-name>
    $vCenterName = <vcenter-name-in-azure>
    ```

4. [Run the onboarding script](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#run-the-script) again with the `--force` parameter.

    ``` powershell-interactive
    ./resource-bridge-onboarding-script.ps1 --force
    ```

5. [Provide the inputs](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#inputs-for-the-script) as prompted.

6. Once the script successfully finishes, the resource bridge should be recovered, and the previously disconnected Arc-enabled resources are manageable in Azure again.

## Next steps

[Troubleshoot Azure Arc resource bridge issues](../resource-bridge/troubleshoot-resource-bridge.md)

If the recovery steps mentioned above are unsuccessful in restoring Arc resource bridge to its original state, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.
- [Open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md).
