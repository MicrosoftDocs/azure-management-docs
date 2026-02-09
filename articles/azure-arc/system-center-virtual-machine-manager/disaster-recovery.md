---
title: Recover from accidental deletion of resource bridge VM
description: Learn how to perform recovery operations for the Azure Arc resource bridge VM in Azure Arc-enabled System Center Virtual Machine Manager disaster scenarios.
ms.topic: how-to
ms.date: 02/09/2026
ms.services: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: v-gajeronika
ms.reviewer: v-gajeronika
author: Jeronika-MS
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to recover the Azure Arc resource bridge VM after accidental deletion, so that I can restore connectivity between on-premises infrastructure and Azure for seamless operations.
---

# Recover from accidental deletion of resource bridge virtual machine

This article shows you how to recover the Azure Arc resource bridge connection in disaster scenarios such as accidental deletion. In these scenarios, the connection between on-premises infrastructure and Azure is lost and any operations performed through Arc fail.

## Recover the Arc resource bridge in case of virtual machine deletion

To recover from Arc resource bridge VM deletion, deploy a new resource bridge with the same resource ID as the current resource bridge by using the following steps.

## Prerequisites

1. Run the disaster recovery script from the same folder where the config (.yaml) files are present. The machine used to run the script to deploy Arc resource bridge has the config files. 

1. The machine used to run the script has bidirectional connectivity to the Arc resource bridge VM on port 6443 (Kubernetes API server) and 22 (SSH), and outbound connectivity to the Arc resource bridge VM on port 443 (HTTPS).


### Recover Arc resource bridge from a Windows machine

1.	Copy the Azure region and resource IDs of the Arc resource bridge, custom location, and SCVMM management server Azure resources.

1.	Download [this script](https://aka.ms/arcvmmwindowsdrscript) and update the following section in the script by using the same information as the original resources in Azure. 

    ```powershell
    $location = <Azure region of the original Arc resource bridge>
    $applianceSubscriptionId = <subscription-id>
    $applianceResourceGroupName = <resource-group-name>
    $applianceName = <resource-bridge-name>

    $customLocationSubscriptionId = <subscription-id>
    $customLocationResourceGroupName = <resource-group-name>
    $customLocationName = <custom-location-name>

    $vmmserverSubscriptionId = <subscription-id>
    $vmmserverResourceGroupName = <resource-group-name>
    $vmmserverName= <SCVMM-name-in-azure>
    ```
 
1.	Run the updated script from the same location where the config YAML files are stored after the initial onboarding. This location is most likely the same folder from where you ran the initial onboarding script unless you moved the config files later to a different location. [Provide the inputs](quickstart-connect-system-center-virtual-machine-manager-to-arc.md#script-runtime) as prompted. 

1.	After the script runs successfully, the old Resource Bridge is recovered, and the connection is re-established to the existing Azure-enabled SCVMM resources.

## Next steps

[Troubleshoot Azure Arc resource bridge issues](../resource-bridge/troubleshoot-resource-bridge.md)

If the recovery steps in this article don't restore Arc resource bridge to its original state, try one of the following support channels:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Connect with [@AzureSupport](https://x.com/azuresupport), the official Microsoft Azure account for improving customer experience. Azure Support connects the Azure community to answers, support, and experts.
- [Open an Azure support request](../../azure-portal/supportability/how-to-create-azure-support-request.md).
