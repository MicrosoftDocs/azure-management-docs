---
title: Download and Import the Agentic Retrieval Expansion Pack for Disconnected Operations on Azure Local
description: "Learn how to download and import the Agentic Retrieval extension expansion pack to prepare for a disconnected deployment on Azure Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 07/07/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to download and import the Agentic Retrieval expansion pack so that the extension is available to install on an Azure Local cluster that uses disconnected operations.
---

# Download and import the Agentic Retrieval expansion pack for disconnected operations on Azure Local

This article explains how to download and import the Agentic Retrieval expansion pack, part of [Agents and Tools with Foundry Local](../overview.md), to prepare a disconnected deployment on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview). You download the expansion pack in a connected environment, transfer it to your disconnected environment, and then import it on the Azure Local Disconnected Operations machine.

After you complete these steps, the Agentic Retrieval extension type is available for installation. To install the extension, see [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Prerequisites

Before you begin, verify these prerequisites:

* Azure Local Disconnected Operations is installed on-premises.
* Access to the Azure Local Disconnected Operations machine, including the Azure Local Disconnected Operations PowerShell module (`Azure.Local.ExpansionPack.psm1`).
* A connected environment to download the expansion pack, and a way to transfer the pack to your disconnected environment.

## Download and import the Agentic Retrieval expansion pack

Download the Agentic Retrieval expansion pack in a connected environment, transfer it to your disconnected environment, and then install it on the Azure Local Disconnected Operations machine.

1. Download the latest [Agentic Retrieval expansion pack](https://aka.ms/azurelocal-pxp-microsoft-foundrylocal-agenticretrieval-k8sextension).
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
* The Agentic Retrieval Azure Arc extension becomes available for installation.

### Verify installation

Run the following command on the Azure Local Disconnected Operations machine, and confirm your expansion pack is in the `Installed` state.

```powershell
Get-ApplianceExpansionPackDetails
```

## Next step

> [!div class="nextstepaction"]
> [Install Agents and Tools with Foundry Local for disconnected operations on Azure Local](deploy-disconnected.md)

## Related content

- [Deployment overview for Agentic Retrieval in Foundry Local](../deploy-overview.md)
- [What you need for Agentic Retrieval](../requirements.md)
