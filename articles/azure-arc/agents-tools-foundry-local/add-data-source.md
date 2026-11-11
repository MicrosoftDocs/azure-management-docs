---
title: Add a Data Source for Agents and Tools with Foundry Local
description: "Learn how to add and manage data sources for indexed sources in Agentic Retrieval in Agents and Tools with Foundry Local, including setup and ingestion processes."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/08/2026
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ai-usage: ai-assisted
#customer intent: As a developer or data scientist, I want to add a data source to Agents and Tools with Foundry Local so that I can enable intelligent search capabilities across my hybrid and multiloud environments.
ms.custom:
  - build-2025
---
# Add a data source for Agents and Tools with Foundry Local

Use the developer portal to add and configure a data source as an indexed source in Agentic Retrieval, the retrieval and grounding component of Agents and Tools with Foundry Local. Follow these step-by-step instructions to set up data ingestion and define indexing parameters.

The portal sends data to the existing `edgeragapp` collection. It doesn't create collections or offer a collection picker. To ingest into another existing collection, use the [Ingestion REST API](reference/ingestion-api-reference.md) and set `collectionName`.

> [!NOTE]
> Local ingestion and the developer portal are available only when `indexedSourcesEnabled=true`.

## Prerequisites

Before you begin:

- Review the following articles:
  - [Agentic Retrieval overview](knowledge-layer-overview.md)
  - [Supported data sources](requirements.md#supported-data-sources)
- The default `edgeragapp` collection is created automatically on startup. Verify that it exists with `GET /collections/edgeragapp` (use `POST /collections` only if it was deleted or when creating custom collections for API ingestion). See the [Collections REST API reference](reference/collections-api-reference.md). Ingestion returns `400 Bad Request` with `COLLECTION_NOT_FOUND` if the target collection doesn't exist.
- For end users who rely on app-role access, assign the app role for the target collection. See [Create app roles for collection access](prepare-authentication.md#create-app-roles-for-collection-access). Callers allowed by an Azure role aren't limited by collection app roles.
- Ensure each source file is 30 MB or smaller. The ingestion service skips larger files.
- If you plan to add a SharePoint Server data source, complete [Set up SharePoint server-to-server authentication](connect-sharepoint-setup.md). Each SharePoint data source needs the Client ID, Issuer ID, Windows SID, optional Realm, and the name of a Kubernetes Secret in the `arc-rag` namespace that contains the signing certificate.
- To access the developer portal, you need the `EdgeRAGDeveloper` app role in Microsoft Entra ID. For the Local chat UI, assign **Reader** and **Cognitive Services OpenAI User** at the extension scope. In disconnected deployments, use **Contributor**. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

## Set up data ingestion

To get started, create a data source by using the local developer portal.

1. Go to the developer portal by using the domain name provided during deployment and app registration. For example: `https://arcrag.contoso.com`.
1. Sign in with developer credentials. Sign-in opens **Chat**, not the developer portal.
1. Open `https://<domain>/developer`, using your deployment domain. This address opens the **Data** page.
1. Select **Add a data source**.
1. On the **Source data** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Name | Name for the data ingestion |
    | Ingestion type | `Advanced` (preselected) |
    | Data Source | **Network share** or **SharePoint Server** |
    | **Network share** | |
    | Network File Share (all NFS versions using AUTH_SYS are supported) / Network File Share (Kerberos authentication) | Path to your NFS share in `server:/path` format, such as `nfs.contoso.com:/exports/data` |
    | Authentication Method | Shown when Kerberos is enabled. Select **Kerberos** or **UID/GID**. With **Kerberos**, the worker node manages credentials and the portal hides the User ID and Group ID fields. |
    | User ID | NFS user ID for **UID/GID** authentication |
    | Group ID | NFS group ID for **UID/GID** authentication |
    | **SharePoint Server** | |
    | SharePoint URL | Your SharePoint web application URL (for example, `https://sharepoint.contoso.com`) |
    | SharePoint relative folder path | Server-relative path to the document library (for example, `/sites/docs/Shared Documents`) |

   :::image type="content" source="media/add-data-source/source-data.png" alt-text="Screenshot of the source data tab where you define the ingestion type and data source." lightbox="media/add-data-source/source-data.png":::

1. If you selected **SharePoint Server** as the data source, enter the following values under **SharePoint authentication**. The portal requires them for each SharePoint data source.

    | Field | Value |
    |---|---|
    | Client ID | GUID from your app principal registration |
    | Issuer ID | GUID from your trusted token issuer registration |
    | Windows SID | SID for the service account in `S-1-5-21-...` format |
    | Certificate Secret name | Name of an existing Kubernetes Secret in the `arc-rag` namespace that contains the signing certificate in the `cert.pfx` key. Enter the name only, without a namespace. |
    | Certificate password Secret name (optional) | Optional. Name of the Secret that contains the certificate password. If you leave it empty, the certificate Secret is used. |
    | Certificate password key (optional) | Optional. Key that holds the certificate password. The default is `cert-password`. The key must exist, even when the password is empty. |
    | Realm | Leave blank for auto-discovery, or enter the realm GUID |

    For steps to create the certificate Secret, see [Create the certificate Secret](connect-sharepoint-setup.md#step-3-create-the-certificate-secret).

1. Select **Next**.
1. On the **Vector index** tab, provide the following information:

    | Field | Value |
    |---|---|
    | Schedule updates | Select `One time index (No schedule updates)`, `1 hour`, `4 hours`, or `1 day`. |
    | Chunk size (characters) | Leave the default value. This setting isn't applied in the current release. |
    | Chunk overlap (characters) | Leave the default value. This setting isn't applied in the current release. |

    The service uses Docling HybridChunker with the BGE-M3 tokenizer to split text into chunks of about 512 tokens. A token is a small unit of text, such as a word or part of a word.

1. Select **Next**.
1. On the **Review and finish** tab, review your configurations.
1. When you're satisfied, select **Create**.

You can also perform data ingestion programmatically by using the [Ingestion REST API](reference/ingestion-api-reference.md). Set `collectionName` to an existing collection in the request body. The API checks Azure roles first and uses the `EdgeRAGDeveloper` app role as a fallback. For setup, see [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

## Next step

> [!div class="nextstepaction"]
> [Set up the data query](set-up-data-query.md)

## Related content

- [Agentic Retrieval overview](knowledge-layer-overview.md)
