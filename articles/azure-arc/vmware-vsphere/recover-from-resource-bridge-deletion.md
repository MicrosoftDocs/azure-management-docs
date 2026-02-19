---
title: Perform disaster recovery operations
description: Learn how to perform recovery operations for the Azure Arc resource bridge VM in Azure Arc-enabled VMware vSphere disaster scenarios.
ms.topic: how-to
ms.date: 02/10/2026
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
# Customer intent: As an IT administrator managing Azure Arc-enabled VMware environments, I want to perform disaster recovery operations for the resource bridge VM, so that I can restore connectivity and functionality after accidental deletion or other failures.
---

# Recover Arc resource bridge connection

In this article, you learn how to restore the Azure Arc resource bridge connection to a working state in case of an accidental deletion of the resource bridge VM or if the resource bridge is irrecoverable.

## Recover the Arc resource bridge

The connection between on-premises infrastructure and Azure can be lost. Any operations performed through Azure Arc can fail in the following disaster scenarios:

- Accidental deletion of the VM.
- VM connection failure (for example, due to changes in the network configuration of the infrastructure)
- VM upgrade failures that you can't fix and need redeployment.

- Expiration of the VM certs due to lack of upgrades.

In such disaster scenarios, you can restore operations by deploying a new resource bridge with the same properties as the current resource bridge. This disaster recovery procedure requires deletion of the existing Arc resource bridge VM in vCenter and the resource bridge Azure resource in the Azure portal. Then you can modify the onboarding script to use for disaster recovery and attempt the recovery. The recovery creates a new resource bridge Azure resource with the same ARM ID, vCenter resource, and custom location.

To deploy the new resource bridge, follow these steps:

1. In the Azure portal, find and copy the following properties from the resources related to your resource bridge: 

    - Arc resource bridge: Azure region, subscription ID, ARM resource ID, resource group name, Arc resource bridge name
    
    - vCenter Azure resource: resource ID, subscription ID, resource group name, name of vCenter resource in Azure
    
    - Custom location: resource ID, subscription ID, resource group name, custom location name

1. In the Azure portal, search for your resource bridge VM and delete the resource bridge VM. This step is necessary as part of the disaster recovery process because you deploy a new resource bridge with the same properties to replace this one. All the other related components, like custom location, vCenter resource, or any other Azure resources, should remain in the Azure portal. You reconnect the new resource bridge with these already existing resources. 

1. Go to your vCenter console and delete the Azure Arc resource bridge VM from the vCenter if it exists. The VM is re-created in a later step as part of the disaster recovery.

1. Download the [onboarding script](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#download-the-onboarding-script) from the Azure portal. You need to provide a different vCenter and custom location name to generate the onboarding script; otherwise, you receive an error if you try to reuse the same names at this step. In the next step, you edit the onboarding script to reuse the same names.

1. Make changes to the downloaded onboarding script to use for the disaster recovery. The script creates a new resource bridge with the same ARM ID, custom location, and vCenter resource. Open the onboarding script in an editor and update the script with the properties that you copied in Step 1. This step replaces the deleted resource bridge with a new resource bridge with the same properties.

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
    
1. Save the edits you made to the onboarding script. 

1. **This step is only if you are using Arc-enabled AVS. Do not follow this step if you are using Arc-enabled VMware.** Run the following command: `az rest --method delete --url  "https://management.azure.com/subscriptions/ <subId>/resourcegroups/<rgName>/providers/Microsoft.AVS/privateClouds/<pcName>/addons/arc?api-version=2022-05-01"`  
 

1. [Run the onboarding script](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#run-the-script) again with the `-force` parameter. The script prompts you to enter the resource bridge configuration settings. [Provide the inputs](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#inputs-for-the-script) as prompted. You can reuse the same IPs and other configurations from the old resource bridge that already meet your network, firewall, and proxy requirements. Otherwise, if you use new IPs, you might have to ensure these IPs meet the networking requirements. You can also specify a new network, storage, or resource pool to use with the new Arc resource bridge.

    ``` powershell-interactive
   ./resource-bridge-onboarding-script.ps1 -force
    ```
    
1. Once the script successfully finishes, the new resource bridge is deployed and reconnected to all necessary resources like the custom location and Arc extension. The previously disconnected Arc-enabled resources are manageable in Azure again.

## Next steps

[Troubleshoot Azure Arc resource bridge issues](../resource-bridge/troubleshoot-resource-bridge.md)

If the recovery steps mentioned above are unsuccessful in restoring Arc resource bridge to its original state, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.
- [Open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md).
