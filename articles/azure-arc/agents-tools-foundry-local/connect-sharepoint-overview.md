---
title: SharePoint server-to-server authentication overview
description: "Learn about SharePoint Server on-premises connectivity with High-Trust Server-to-Server (S2S) authentication for Agents and Tools with Foundry Local."
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 10/29/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#customer intent: As a platform administrator, I want to understand how SharePoint server-to-server authentication works with Agents and Tools with Foundry Local so that I can securely connect on-premises SharePoint data.
---

# SharePoint server-to-server authentication overview

The **Agents and Tools with Foundry Local** extension uses High-Trust Server-to-Server (S2S) authentication for indexed ingestion from SharePoint Server on-premises. The extension signs a JSON Web Token (JWT) with a certificate private key, and SharePoint validates it by using the registered public key.

The PFX private key and password are long-lived credentials stored in Kubernetes Secrets. This flow doesn't store the ingestion account's AD password or contact an external token service. It is separate from the live SharePoint MCP flow, which uses delegated user-token authentication.

This article explains the authentication architecture, trust components, and configuration model so that you can plan a secure SharePoint connection for your environment.

This article applies if you run Agents and Tools with Foundry Local on Azure Kubernetes Service (AKS) or Azure Local (Arc-enabled Kubernetes) and connect to SharePoint Server Subscription Edition (on-premises).

## What you set up versus what the extension handles

The following table shows the one-time setup tasks you complete and the certificate and token tasks the extension handles automatically.

| Your responsibility (one-time setup) | Extension handles automatically |
|---|---|
| Create a certificate (Personal Information Exchange (PFX)) and a public certificate file (.cer) | Mounts the certificate Secret that each data source selects into its ingestion pods |
| Store the PFX and its password in a Kubernetes Secret in the application namespace | Reads the Secret selected by each new ingestion source |
| Register certificate trust on SharePoint Server | Signs JWT tokens with the certificate private key at runtime |
| Register the app principal and grant permissions on SharePoint | Caches tokens (12 hours), handles expiration, and retries on transient errors |
| Give the app principal and selected ingestion account read access to the target content | Discovers the SharePoint realm automatically (if not provided) |
| Create a user profile and obtain the Windows Security Identifier (SID) | |
| Provide the identity values and certificate Secret name when you add each data source | |

## Architecture

The SharePoint server-to-server ingestion architecture uses certificate-based trust. A customer-managed Kubernetes Secret in the application namespace holds the PFX certificate and password. Each new SharePoint source selects an existing Secret. The ingestion pod signs JWTs locally. SharePoint Server validates the token by using the trusted public key and then returns documents.

The following diagram shows the indexed-ingestion authentication flow.

:::image type="content" source="media/connect-sharepoint-overview/sharepoint-architecture.svg" alt-text="Architecture diagram. A cluster administrator creates a Kubernetes Secret with the certificate and password. The ingestion pod reads the selected Secret, signs a JWT, and sends it to SharePoint Server, which returns documents." lightbox="media/connect-sharepoint-overview/sharepoint-architecture.svg" border="false":::

### Key design points

This server-to-server ingestion architecture uses certificate-based trust in connected and disconnected environments.

- **No stored AD user password**: Authentication uses a certificate-signed JWT. The PFX and its password remain protected Kubernetes Secret data.
- **No AD FS or ACS required**: The pod creates the token locally.
- **Compatible with disconnected environments**: The certificate is stored in the cluster, so no Azure connectivity is needed for authentication.
- **Data-source-level isolation**: Different SharePoint sites can use different identities.
- **Token caching**: JWTs are valid for 12 hours and refresh automatically.

## Prerequisites summary

Before you deploy Agents and Tools with Foundry Local with SharePoint enabled, ensure you have:

- A working AKS or Arc-enabled Kubernetes cluster.
- A working SharePoint Server Subscription Edition environment.
- Completed required certificate, identity, and trust configuration.
- Read access for both the app principal and the selected ingestion account.
- Network connectivity and CA trust from cluster pods to SharePoint over HTTPS.

For full setup steps and step-by-step validation, see [Set up SharePoint server-to-server authentication for Agents and Tools with Foundry Local](connect-sharepoint-setup.md).

## SharePoint configuration fields

SharePoint ingestion is enabled by default in Knowledge and combined deployments. You configure identity and certificate references for each new data source.

Enter these values when you add each SharePoint data source in the developer portal or the [Ingestion REST API](reference/ingestion-api-reference.md). A new SharePoint data source must include its own Client ID, Issuer ID, Windows SID, and certificate Secret name.

| Portal field | Ingestion API field | Required | Description |
|---|---|---|---|
| Client ID | `sharePointClientId` | Yes | The app Client ID you choose when registering the app principal in SharePoint. |
| Issuer ID | `sharePointIssuerId` | Yes | The Issuer ID you choose when registering the trusted security token issuer. |
| Windows SID | `sharePointSid` | Yes | Windows SID of the service account (for example, `S-1-5-21-...-1001`). |
| Certificate Secret name | `sharePointCertSecretName` | Yes | Kubernetes Secret in the application namespace that contains the PFX file in the `cert.pfx` key. |
| Certificate password Secret name | `sharePointCertPasswordSecretName` | No | Secret that contains the PFX password. Defaults to the certificate Secret. |
| Certificate password key | `sharePointCertPasswordSecretKey` | No | Key that holds the PFX password. Defaults to `cert-password`. |
| Realm | `sharePointRealm` | No | SharePoint authentication realm. If left blank, the extension discovers it automatically from the SharePoint server. |

Values that you enter apply only to that data source. This approach lets you use different identities and certificate Secrets for different SharePoint sites.

## Frequently asked questions

**Does the extension need internet access to authenticate to SharePoint?**
No. The pod signs the JWT locally by using the certificate private key. The extension doesn't contact an external token service for this indexed-ingestion flow.

**What SharePoint versions are supported?**
SharePoint Server Subscription Edition (SE). Earlier versions might work, but Microsoft doesn't test them.

**Can I use the same certificate for multiple SharePoint sites?**
Yes. Register the trust on each SharePoint server, and register the app principal on each site collection. Each data source can reference the same certificate Secret or a different one.

**Can I change the server-to-server identity (Client ID, Issuer ID, SID) without reinstalling?**
Yes. Each data source stores its own identity values and certificate Secret name. Provide the new values when you create the replacement source or job.

**What if I don't know my SharePoint realm?**
Leave the `realm` field blank. The extension automatically discovers it by sending an unauthenticated request to your SharePoint server and parsing the `WWW-Authenticate` response header.

**What permissions does the app principal need?**
Grant the app principal read access at the scope that contains the selected library. The ingestion account identified by the Windows SID also needs read access to that content. A SharePoint User Profile makes the SID resolvable but doesn't grant content access.

**What is the Windows SID and why is it needed?**
The Windows SID (Security Identifier, format `S-1-5-21-...`) identifies the service account in the JWT `nameid` claim. SharePoint uses it to resolve user identity and apply permissions. Using `DOMAIN\username` causes HTTP 401 errors.

**How does the certificate get into the pod?**
The ingestion worker reads the Kubernetes Secret selected by the data source. The Secret must contain a nonempty `cert.pfx` key. The selected password data key must exist even when the PFX password is empty.

## Related content

- [Set up server-to-server authentication](connect-sharepoint-setup.md)
- [SharePoint Server-to-Server reference](connect-sharepoint-reference.md)
