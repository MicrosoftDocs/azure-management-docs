---
title: SharePoint Server-to-Server Authentication Overview for Agents and Tools with Foundry Local
description: "Learn about SharePoint Server on-premises connectivity with High-Trust Server-to-Server (S2S) authentication for Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 05/18/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want to understand how SharePoint server-to-server authentication works with Agents and Tools with Foundry Local so that I can securely connect on-premises SharePoint data.
---

# SharePoint server-to-server authentication overview for Agents and Tools with Foundry Local

Agents and Tools with Foundry Local connects to SharePoint Server on-premises by using High-Trust Server-to-Server (S2S) authentication. Instead of storing a username and password, the extension signs a JSON Web Token (JWT) with a certificate's private key, and SharePoint validates it by using the registered public key.

This authentication flow works fully disconnected. It doesn't require Active Directory Federation Services (AD FS), Azure Access Control Service (ACS), or internet connectivity.

This article applies to Agents and Tools with Foundry Local on Azure Kubernetes Service (AKS) or Azure Local (Arc-enabled Kubernetes) connecting to SharePoint Server Subscription Edition (on-premises).

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## What you set up vs. what the extension handles

The following table shows which one-time setup tasks you complete and which certificate and token tasks the extension completes for you automatically.

| Your responsibility (one-time setup) | Extension handles automatically |
|---|---|
| Create a certificate (Personal Information Exchange (PFX) + public certificate file (.cer)) | Syncs the certificate from Key Vault into the cluster via Container Storage Interface (CSI) driver |
| Upload the PFX to Azure Key Vault | Signs JWT tokens with the certificate private key at runtime |
| Register the certificate trust on SharePoint Server | Caches tokens (12 hours), handles expiry, retries on transient errors |
| Register the app principal and grant permissions on SharePoint | Mounts the certificate into ingestion pods securely |
| Create a managed identity and federate it with the cluster | Discovers the SharePoint realm automatically (if not provided) |
| Create a user profile and obtain the Windows Security Identifier (SID) | |
| Provide configuration values during installation or in the user interface (UI) | |

## Architecture

The SharePoint server-to-server ingestion architecture for Agents and Tools with Foundry Local uses certificate-based trust to authenticate requests without stored credentials or an external token service. Azure Key Vault stores the PFX certificate and password, the CSI Secrets Store Driver syncs them into Kubernetes, the ingestion pod signs JWTs locally, and SharePoint Server validates the token by using the trusted public key before returning documents.

:::image type="content" source="media/connect-sharepoint-overview/sharepoint-architecture.png" alt-text="Architecture diagram showing certificate-based SharePoint authentication flow through Azure Key Vault, Kubernetes, and SharePoint Server." lightbox="media/connect-sharepoint-overview/sharepoint-architecture.png" border="false":::

### Key design points

This server-to-server ingestion architecture uses certificate-based trust to provide secure, resilient SharePoint authentication in connected and disconnected environments without storing user credentials.

- **No stored passwords** -  authentication uses a certificate-signed JWT.
- **No AD FS or ACS required** -  the token is created locally by the pod.
- **Disconnected/air-gap compatible** -  once the certificate is synced to the cluster, no Azure connectivity is needed for authentication.
- **Per-datasource isolation** -  different SharePoint sites can use different identities.
- **Token caching** -  JWTs are valid for 12 hours and auto-refreshed.

## Prerequisites checklist

Complete every item before installing Agents and Tools with Foundry Local with SharePoint enabled.

| # | Prerequisite | Who |
|---|---|---|
| **Azure / Cluster** | | |
| 1 | AKS or Arc-enabled Kubernetes cluster operational | Platform Admin |
| 2 | Azure Key Vault instance available (or ability to create one) | Azure Admin |
| 3 | CSI Secrets Store Driver installed on the cluster | Platform Admin |
| 4 | Workload Identity / OpenID Connect (OIDC) Issuer enabled on the cluster | Platform Admin |
| 5 | User-assigned managed identity with Key Vault secret read access | Azure Admin |
| 6 | Federated identity credential linking the managed identity to `edgerag-sp-sa` ServiceAccount | Azure Admin |
| **Certificate** | | |
| 7 | Self-signed certificate created (RSA 2048, SHA256) | IT Admin |
| 8 | PFX (private key) exported and uploaded to Azure Key Vault | IT Admin |
| 9 | CER (public key) exported for SharePoint registration | IT Admin |
| **SharePoint Server** | | |
| 10 | SharePoint Server Subscription Edition installed and operational | SharePoint Admin |
| 11 | Certificate registered as Trusted Root Authority | SharePoint Admin |
| 12 | Certificate registered as Trusted Security Token Issuer | SharePoint Admin |
| 13 | `AllowOAuthOverHttp` enabled (if SharePoint uses HTTP, not HTTPS) | SharePoint Admin |
| 14 | App Management Service Application running (with database) | SharePoint Admin |
| 15 | App Management Service Application Proxy created | SharePoint Admin |
| 16 | Subscription Settings Service Application running | SharePoint Admin |
| 17 | User Profile Service Application running | SharePoint Admin |
| 18 | User Profile Service Application Proxy created | SharePoint Admin |
| 19 | App domain and site subscription configured | SharePoint Admin |
| 20 | App principal registered on target SharePoint site | SharePoint Admin |
| 21 | App principal granted FullControl at SiteCollection scope | SharePoint Admin |
| 22 | User profile created for the service account | SharePoint Admin |
| 23 | Windows SID obtained for the service account | IT Admin |
| **Network** | | |
| 24 | Pods can reach SharePoint server on port 80 (HTTP) or 443 (HTTPS) | Network Admin |
| 25 | Pods can resolve SharePoint server hostname via Domain Name System (DNS) | Network Admin |
| 26 | Pods can reach Azure Key Vault on port 443 (for initial certificate sync) | Network Admin |

## Portal fields reference

When you install Agents and Tools with Foundry Local via the Azure portal, the **Data Source Connection / Authentication** section includes the following SharePoint fields.

### Install-time fields (CSI / certificate delivery)

These fields configure how the certificate is delivered from Azure Key Vault to the cluster. Changing these fields requires a Helm upgrade and pod restart after installation.

| Portal field | Helm key | Type | Default | Description |
|---|---|---|---|---|
| **Enable SharePoint Ingestion** | `sharepoint.sharepointIngestionEnabled` | Toggle | `false` | Master switch. Enables all SharePoint resources (CSI SecretProviderClass, ServiceAccount, certificate mounts). |
| **Key Vault Name** | `sharepoint.s2s.keyvaultName` | Text | _(empty)_ | Name of your Azure Key Vault containing the PFX certificate and password. |
| **KV Cert Secret Name** | `sharepoint.s2s.kvCertSecretName` | Text | `edgerag-sp-s2s-cert` | Name of the secret in Key Vault that holds the PFX certificate (base64-encoded). |
| **KV Cert Password Secret Name** | `sharepoint.s2s.kvCertPasswordSecretName` | Text | `sp-cert-password` | Name of the secret in Key Vault that holds the PFX password. |
| **Workload Identity Client ID** | `sharepoint.s2s.workloadIdentityClientId` | Globally unique identifier (GUID) | _(empty)_ | Client ID of the user-assigned managed identity that has `get` access to the Key Vault secrets. |
| **Key Vault Tenant ID** | `sharepoint.s2s.kvTenantId` | Globally unique identifier (GUID) | _(auto from session)_ | Microsoft Entra tenant ID where the managed identity and Key Vault reside. Usually auto-populated. Falls back to `auth.tenantId` if not specified. |

### Post-install fields (server-to-server identity -  optional at install time)

Set these fields during installation or later per-datasource in the user interface (UI). The process stores them in a ConfigMap. You can update them without reinstalling by using `az k8s-extension update`.

| Portal field | Helm key | Type | Default | Description |
|---|---|---|---|---|
| Client ID | `sharepoint.s2s.clientId` | GUID | _(empty)_ | The app Client ID you choose when registering the app principal in SharePoint. |
| Issuer ID | `sharepoint.s2s.issuerId` | GUID | _(empty)_ | The Issuer ID you choose when registering the trusted security token issuer. |
| Default SID | `sharepoint.s2s.defaultSid` | String | _(empty)_ | Windows SID of the service account (for example, `S-1-5-21-...-1001`). |
| Realm | `sharepoint.s2s.realm` | GUID | _(empty, auto-discovered)_ | SharePoint authentication realm. If left blank, the extension discovers it automatically from the SharePoint server. |

If you manage multiple SharePoint sites with different identities, leave the server-to-server identity fields empty at install time and configure them per-datasource in the UI.

## Frequently asked questions

**Does the extension need internet access to authenticate to SharePoint?**
No. The pod signs the JWT locally by using the certificate private key. The extension doesn't contact any external token service. The only Azure connectivity needed is the initial certificate sync from Key Vault (via the CSI driver).

**What SharePoint versions are supported?**
SharePoint Server Subscription Edition (SE). Earlier versions might work but aren't tested.

**Can I use the same certificate for multiple SharePoint sites?**
Yes. Register the trust on each SharePoint server, and register the app principal on each site collection. The certificate is cluster-wide.

**What if I don't know my SharePoint realm?**
Leave the `realm` field blank. The extension auto-discovers it by making an unauthenticated request to your SharePoint server and parsing the `WWW-Authenticate` response header.

**Can I change the server-to-server identity (Client ID, Issuer ID, SID) without reinstalling?**
Yes. The extension stores these values in a ConfigMap. You can update them by using `az k8s-extension update`. The install-time values (Key Vault name, managed identity) require a Helm upgrade.

**What permissions does the app principal need?**
`FullControl` at the `SiteCollection` scope. This permission level allows the extension to enumerate and download documents. Read-only access isn't sufficient due to SharePoint API requirements for document library enumeration.

**What is the Windows SID and why is it needed?**
The Windows SID (Security Identifier, format `S-1-5-21-...`) identifies the service account in the JWT `nameid` claim. SharePoint uses it to resolve the user's identity and apply permissions. Using `DOMAIN\username` format causes 401 errors.

**How does the certificate get into the pod?**
The CSI Secrets Store Driver syncs the PFX and password from Azure Key Vault into a Kubernetes secret. The pod mounts this secret as a volume at `/certs/cert.pfx` and receives the password as the `SHAREPOINT_CERT_PASSWORD` environment variable. The pod never communicates with Key Vault directly.

**What happens if Key Vault becomes unreachable after initial setup?**
The Kubernetes secret persists in the cluster. Existing and new pods continue to work by using the cached secret. The certificate doesn't rotate until Key Vault is reachable again, but authentication continues uninterrupted.

## Next step

> [!div class="nextstepaction"]
> [Set up server-to-server authentication](connect-sharepoint-setup.md)
