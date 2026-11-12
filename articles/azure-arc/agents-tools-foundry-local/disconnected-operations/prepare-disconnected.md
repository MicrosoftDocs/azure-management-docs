---
title: Download and import the expansion pack
description: "Learn how to download and import the Agents and Tools with Foundry Local extension expansion pack to prepare for a disconnected deployment on Azure Local."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to download and import the Agents and Tools with Foundry Local expansion pack so that the extension is available to install on an Azure Local cluster that uses disconnected operations.
---

# Download and import the expansion pack

This article explains how to download and import the [Agents and Tools with Foundry Local](../overview.md) expansion pack to prepare a disconnected deployment on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview). You download the expansion pack in a connected environment, transfer it to your disconnected environment, and then import it on the Azure Local Disconnected Operations machine.

After you complete these steps, the Agents and Tools with Foundry Local extension type is available for installation. To install the extension, see [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md).

[!INCLUDE [disconnected-operations-preview-notice](../includes/disconnected-operations-preview-notice.md)]

## Prerequisites

Before you begin, verify these prerequisites:

* Azure Local Disconnected Operations is installed on-premises.
* Access to the Azure Local Disconnected Operations machine, including the Azure Local Disconnected Operations PowerShell module (`Azure.Local.ExpansionPack.psm1`).
* A connected environment to download the expansion pack, and a way to transfer the pack to your disconnected environment.

## Download and import the Agents and Tools with Foundry Local expansion pack

Download the Agents and Tools with Foundry Local expansion pack in a connected environment, transfer it to your disconnected environment, and then install it on the Azure Local Disconnected Operations machine.

1. Download the latest [Agents and Tools with Foundry Local expansion pack](https://aka.ms/azurelocal-pxp-microsoft-foundrylocal-agenticretrieval-k8sextension).
1. Transfer the expansion pack to the disconnected Azure Local environment.
1. Run the following commands on the Azure Local Disconnected Operations machine to install the expansion pack.

    Replace `<PATH_TO_EXPANSION_PACK>` with the local path to the expansion pack zip file and `<PATH_TO_ALDO_MODULES>` with the path to the Azure Local Disconnected Operations modules before you run the command.

    ```powershell
    Import-Module "<PATH_TO_ALDO_MODULES>\Azure.Local.ExpansionPack.psm1"

    $expansionPackId = Start-AldoExpansionPackUpload `
        -ExpansionPackPath "<PATH_TO_EXPANSION_PACK>"

    $result = Start-AldoExpansionPackInstallation `
        -ExpansionPackId $expansionPackId `
        -Wait
    ```

When installation finishes:

* Container images are imported into the `edgeartifacts` registry.
* Model artifacts are published to the registry.
* The Agents and Tools with Foundry Local Azure Arc extension becomes available for installation.

### Verify installation

Run the following commands on the Azure Local Disconnected Operations machine, and confirm your expansion pack is in the `Installed` state.

```powershell
Get-AldoExpansionPackDetails -ExpansionPackId $expansionPackId
Get-AldoExpansionPackInstallationStatus -ExpansionPackId $expansionPackId
```

## Next step

> [!div class="nextstepaction"]
> [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md)

## Related content

- [Disconnected operations for Agents and Tools with Foundry Local overview](overview-disconnected.md)
- [Deployment overview for Agents and Tools with Foundry Local](../deploy-overview.md)
- [What you need for Agents and Tools with Foundry Local](../requirements.md)
