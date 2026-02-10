---
title: Migrate Existing Target Resources to General Availability
description: Learn how to migrate existing target resources in Azure Arc workload orchestration to the general availability (GA) version.
author: sethmanheim
ms.author: sethm
ms.topic: reference
ms.date: 06/24/2025
---

# Migrate existing target resources to the general availability 

If you used workload orchestration in preview, you need to run a migration script to update the existing target resources to the general availability (GA) version. For more information about the GA release, see [Release Notes for Workload Orchestration](release-notes.md).

## Prerequisites

- Set up your environment for workload orchestration. If you haven't, go to [Prepare your environment for workload orchestration](initial-setup-environment.md) to set up the prerequisites.

- Install Azure CLI `resource-graph` extension. If you don't have it installed, run the following command:

    ```bash
    az extension add --name resource-graph
    ```

## Migration script

In the [ZIP folder](https://github.com/Azure/workload-orchestration/blob/main/workload%20orchestration%20files.zip) you downloaded as part of [Prepare your environment for workload orchestration](initial-setup-environment.md), you can find the PowerShell script `WOGAMigration.ps1` that migrates your existing workload orchestration target resources to the GA version. The script updates the `contextId` property of your existing targets to point to the new API version of workload orchestration.

Run the script in PowerShell with the `-location` parameter set to the Azure region where your workload orchestration environment is deployed. 

```powershell
.\WOGAMigration.ps1 -location <location>
```