---
title: Configure OneLake Identity for Cloud subvolumes
description: Learn how to configure OneLake Identity for Cloud subvolumes in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 09/27/2025
# Customer intent: "As a cloud administrator, I want to configure OneLake Identity for Cloud subvolumes in Azure Container Storage enabled by Azure Arc."
---

# Configure OneLake Identity for Cloud subvolumes

This article describes an alternate configuration for OneLake lakehouses to be used for a subvolume configuration. This applies to either Cloud Ingest subvolumes or Cloud Mirror subvolumes.

## Configure OneLake for Extension Identity

### Add Extension Identity to OneLake workspace

1. Navigate to your OneLake portal; for example, `https://youraccount.powerbi.com`.
1. Create or navigate to your workspace.
   :::image type="content" source="media/howto-configure-onelake-identity/onelake-workspace.png" alt-text="Screenshot showing workspace ribbon in portal." lightbox="media/howto-configure-onelake-identity/onelake-workspace.png":::
1. Select **Manage Access**.
   :::image type="content" source="media/howto-configure-onelake-identity/onelake-manage-access.png" alt-text="Screenshot showing manage access screen in portal." lightbox="media/howto-configure-onelake-identity/onelake-manage-access.png":::
1. Select **Add people or groups**.
1. Enter your extension name from your Azure Container Storage enabled by Azure Arc installation. The extension is called `microsoft.arc.containerstorage` in the Azure portal. This name must be unique within your tenant.
   :::image type="content" source="media/howto-configure-onelake-identity/add-extension-name.png" alt-text="Screenshot showing add extension name screen." lightbox="media/howto-configure-onelake-identity/add-extension-name.png":::
1. Change the drop-down for permissions from **Viewer** to **Contributor**.
   :::image type="content" source="media/howto-configure-onelake-identity/onelake-set-contributor.png" alt-text="Screenshot showing set contributor screen." lightbox="media/howto-configure-onelake-identity/onelake-set-contributor.png":::
1. Select **Add**.

## Attach subvolume to Edge Volume

When creating a subvolume for [Cloud Ingest](howto-configure-cloud-ingest-subvolumes.md#attach-ingest-subvolume-to-edge-volume) or [Cloud Mirror](howto-configure-cloud-mirror-subvolumes.md#attach-mirror-subvolume-to-the-edge-volume), during the CRD creation process, there are two parameters that need to be set specifically for OneLake:

- `spec.storageaccountendpoint` in **ingestSubvolume.yaml** or `spec.blobAccount.accountendpoint` in **mirrorSubvolume.yaml**: Your storage account endpoint is the prefix of your Power BI web link. For example, if your OneLake page is `https://contoso-motors.powerbi.com/`, then your endpoint is `https://contoso-motors.dfs.fabric.microsoft.com`.
- `spec.containerName` in **ingestSubvolume.yaml** or `spec.blobAccount.containerName` in **mirrorSubvolume.yaml**: Details of your OneLake lakehouse, for example, `<WORKSPACE>/<DATA_LAKE>.Datalake/Files)`.

## Next Step

Continue the steps in either [Attach Ingest subvolume to Edge Volume](howto-configure-cloud-ingest-subvolumes.md#attach-ingest-subvolume-to-edge-volume) or [Attach Mirror subvolume to the Edge Volume](howto-configure-cloud-mirror-subvolumes.md#attach-mirror-subvolume-to-the-edge-volume) to complete the configuration.