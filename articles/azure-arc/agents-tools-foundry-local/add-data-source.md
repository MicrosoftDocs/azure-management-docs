---
title: Add a Data Source for Agents and Tools with Foundry Local
description: "Learn how to add and manage data sources for Agents and Tools with Foundry Local chat solutions, including setup, and ingestion processes."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/27/2026
ms.subservice: edge-rag
#customer intent: As a developer or data scientist, I want to add a data source to Agents and Tools with Foundry Local so that I can enable intelligent search capabilities across my hybrid and multiloud environments.
ms.custom:
  - build-2025
---
# Add a data source for Agents and Tools with Foundry Local

Add and configure a data source for your Agents and Tools with Foundry Local chat solution by using the developer portal. Follow these step-by-step instructions to set up data ingestion and define indexing parameters.

By default, ingested data is added to the `edgeragapp` collection unless you select an existing collection or create a new one. To let end users query data in a collection, you must assign them to the app role mapped to that collection.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin:

- Review the following articles:
  - [Configure the Knowledge Layer](build-chat-solution-overview.md)
  - [Supported data sources](requirements.md#supported-data-sources)
- Decide whether to use the default `edgeragapp` collection or create a new collection for this data source. For more information, see [Collections](collections-overview.md).
- Make sure end users are assigned to the app role for the target collection. For steps, see [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access).
- If you plan to add a SharePoint Server data source and didn't configure SharePoint server-to-server identity parameters during deployment, complete [Set up SharePoint server-to-server authentication](connect-sharepoint-setup.md). You need the following values  to add a SharePoint Server data source: Client ID, Issuer ID, Windows SID, and Realm (optional).
- To access to the developer portal, you must have both the "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles in Microsoft Entra.

## Set up data ingestion

To get started, create a data source by using the local developer portal.

1. Go to the developer portal by using the domain name provided during deployment and app registration. For example: `https://arcrag.contoso.com`.
1. Sign in with developer credentials that have both "EdgeRAGDeveloper" and "EdgeRAGEndUser" roles assigned. If you have the right access configured, you're automatically redirected to the developer portal.
1. Select **Get Started**.
1. Go to the **Data** tab.
1. Select **Add a data source**.
1. On the **Source data** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Name | Name for the data ingestion |
    | Data source | Network share or SharePoint Server |
    | **Network share** | |
    | Network File Share | Path to your network file server (NFS) share |
    | User ID | NFS user ID |
    | Group ID | NFS group ID |
    | **SharePoint Server** | |
    | SharePoint URL | Your SharePoint web application URL (for example, `http://sharepoint.contoso.com`) |
    | Folder Path | Server-relative path to the document library (for example, `/sites/docs/Shared Documents`) |

   :::image type="content" source="media/add-data-source/source-data.png" alt-text="Screenshot of the source data tab where you define the ingestion type and data source.":::

1. If you selected **SharePoint Server** as the data source and didn't configure server-to-server identity parameters during deployment, expand **Authentication** and enter:

    | Field | Value |
    |---|---|
    | Client ID | GUID from your app principal registration |
    | Issuer ID | GUID from your trusted token issuer registration |
    | Windows SID | SID for the service account in `S-1-5-21-...` format |
    | Realm | Leave blank for auto-discovery, or enter the realm GUID |

1. Select **Next**.
1. On the **Vector index** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Collection | Select the target collection for ingested data. Choose an existing collection, or use the default `edgeragapp` collection. |
    | Schedule updates | Frequency at which your data is synced for updates |
    | Chunk size | Select the appropriate chunk size |
    | Chunk overlap | Select the appropriate chunk overlap |

    :::image type="content" source="media/add-data-source/configure-index.png" alt-text="Screenshot of the vector index tab where you configure chunk size and overlap.":::

1. Select **Next**.
1. On the **Review and finish** tab, review your configurations.
1. When you're satisfied, select **Create**.

You can also perform data ingestion programmatically using the Ingestion API. When using the API, specify the target collection with the `collectionName` parameter in the request body.

## Next step

> [!div class="nextstepaction"]
> [Set up the data](set-up-data-query.md)

## Related content

- [Configure the Knowledge Layer](build-chat-solution-overview.md)
