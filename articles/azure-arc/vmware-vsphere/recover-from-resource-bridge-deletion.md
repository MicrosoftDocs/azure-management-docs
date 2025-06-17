---
title: Perform disaster recovery operations
description: Learn how to perform recovery operations for the Azure Arc resource bridge VM in Azure Arc-enabled VMware vSphere disaster scenarios.
ms.topic: how-to 
ms.date: 10/24/2024
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: jsuri
author: jyothisuri
ms.custom:
  - build-2025
---

# Recover Arc resource bridge connection

In this article, you learn how to restore the Azure Arc resource bridge connection to a working state in case of an accidental deletion of the resource bridge VM or if the resource bridge is irrecoverable.

## Recover the Arc resource bridge

The connection between on-premises infrastructure and Azure can be lost and any operations performed through Arc can fail in case of the following disaster scenarios:
- Accidental deletion of the VM
- VM connection failure (for example, due to changes in the network configuration of the infrastructure)
- VM upgrade failures which can’t be fixed and need redeployment

In such disaster scenarios, you need to deploy a new resource bridge with the same resource ID as the current resource bridge using the following steps.

1. Copy the Azure region and resource IDs of the Arc resource bridge, custom location, and vCenter Azure resources.

2. The onboarding script (or `az arcappliance run` command) creates a few `YAML` files to store some metadata about the appliance.
    - If the `YAML` files are not accessible, manually delete the resource bridge from Azure and also the resource bridge VM from the vCenter (if the VM exists).
    - If the `YAML` files are accessible, run the script from the same folder where these files are present, and the script will handle the deletion of the resource bridge.

3. Download the [onboarding script](../vmware-vsphere/quick-start-connect-vcenter-to-arc-using-script.md#download-the-onboarding-script) from the Azure portal and update the following section in the script, using the same information as the original resources in Azure.

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
