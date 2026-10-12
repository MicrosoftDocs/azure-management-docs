---
title: Connect Microsoft 365 Local to Agentic Retrieval
description: Configure SharePoint Server and Exchange Server as live Microsoft 365 Local knowledge sources for Agentic Retrieval in Foundry Local.
author: dgitelman
ms.author: dgitelman
ms.topic: how-to
ms.date: 10/12/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
# Customer intent: As an administrator, I want to connect on-premises SharePoint and Exchange data to Agentic Retrieval so that agents can retrieve data and perform approved actions as the signed-in user.
---

# Connect Microsoft 365 Local to Agentic Retrieval

Microsoft 365 Local connects Agentic Retrieval to on-premises SharePoint Server and Exchange Server. Agents retrieve current files, email, and calendar information directly from the source instead of ingesting that content into a collection. Source-system permissions are enforced for every user.

> [!IMPORTANT]
> Microsoft 365 Local integration is in public preview. Use the Azure CLI to install or update the extension for this procedure. The Azure portal installation experience doesn't expose the Microsoft 365 Local settings.

## Supported capabilities

Microsoft 365 Local provides two independent Model Context Protocol (MCP) connectors:

- **SharePoint MCP** searches and downloads files from SharePoint Server. In the supported `s2s_user` mode, Active Directory Federation Services (AD FS) identifies the signed-in user and the MCP uses a customer-owned high-trust certificate to call SharePoint as that user.
- **Exchange MCP** searches email and calendar information through Exchange Web Services (EWS), using a delegated AD FS token for the signed-in user.

Both connectors are disabled by default. Enable the Microsoft 365 Local umbrella setting and each connector that you need:

```text
m365Local.enabled=true
sharepointMcp.enabled=true
exchangeMcp.enabled=true
```

Microsoft 365 Local requires a `combined` deployment. You can enable either connector or both connectors.

### Supported actions and human approval

In addition to retrieval, agents can propose the following actions.

| Connector | Supported actions |
|---|---|
| Exchange mail | Create or update a draft, send a draft or new message, reply, reply all, forward, and delete a message |
| Exchange calendar | Accept, tentatively accept, or decline an event; create, update, cancel, or delete an event |
| SharePoint | Upload a document, create a folder, update document metadata, copy or move a document, and grant or revoke document access |

Every write action requires human approval in the chat experience. The agent shows a sanitized proposal and pauses. The action runs only after the user approves it. Rejection doesn't change Microsoft 365 data. The chat shows the authoritative approval, execution, and final result states.

Approval applies to the exact proposed operation and executes it at most once. Connector tokens aren't included in the proposal, model context, persisted approval state, logs, traces, or action result.

## Architecture

The end-to-end flow is:

1. The user signs in to the Agentic Retrieval chat experience.
1. The browser uses OAuth 2.0 authorization code flow with Proof Key for Code Exchange (PKCE) to sign in directly to the customer AD FS connector application.
1. The browser sends the connector token with the agent run.
1. Agents Runtime forwards the token only to the configured SharePoint or Exchange MCP destination.
1. SharePoint MCP independently validates the AD FS token, reads the user's Windows security identifier (SID), and creates a certificate-signed high-trust user token.
1. Exchange MCP uses the delegated token to call EWS as the signed-in user.
1. SharePoint and Exchange enforce the user's permissions and return only authorized data.

Agentic Retrieval doesn't store the user's connector token, mailbox password, client secret, or service-account password. Never pass a private key, certificate password, or access token as an extension configuration value. Extension configuration is stored as plaintext metadata.

## Prerequisites

Before you begin, make sure:

- Agentic Retrieval is deployed, or ready to deploy, in `combined` mode.
- The cluster has a policy-enforcing container network interface (CNI). Microsoft 365 Local requires `networkPolicies.enabled=true`.
- AD FS 2019 or later is available and reachable from end-user browsers and cluster workloads.
- The final Agentic Retrieval gateway hostname and TLS certificate are available and trusted.
- Separate AD FS public clients and Web API audiences are available for SharePoint and Exchange.
- SharePoint high-trust registration and a signing certificate are available.
- Exchange EWS is enabled and accepts delegated AD FS tokens.
- A dynamic `ReadWriteMany` (RWX) storage class is available, or two statically bound RWX claims are prepared.
- Browser, pod, DNS, TLS, storage, SharePoint, Exchange, and AD FS network paths are open.
- Private certification authority (CA) roots are available as PEM files when customer endpoints don't use publicly trusted certificates.

For the base cluster, identity, model, and networking requirements, see [Deployment prerequisites checklist for Agentic Retrieval](complete-prerequisites.md).

### Required network flows

| Source | Destination | Port | Purpose |
|---|---|---|---|
| End-user browser | Agentic Retrieval gateway | TCP 443 | Chat and API access |
| End-user browser | SharePoint and Exchange AD FS authorities | TCP 443 | Connector authorization and token calls |
| Agentic Retrieval pods | AD FS JSON Web Key Set (JWKS) endpoint | TCP 443 | SharePoint token validation |
| SharePoint MCP pod | SharePoint Windows-claims zone | TCP 443 | SharePoint retrieval and actions |
| Exchange MCP pod | Exchange EWS | TCP 443 | Mail and calendar retrieval and actions |
| Kubernetes nodes | RWX storage | Provider-specific | File and attachment caches. Azure Files SMB uses TCP 445; NFS commonly uses TCP 2049 |
| SharePoint MCP pod | Key Vault | TCP 443 | Certificate delivery when Key Vault is used |

Run the following checks from an administrator workstation on the end-user network. Don't use `-k`; certificate trust and hostname validation must succeed.

```bash
nslookup <agentic-retrieval-host>
nslookup <sharepoint-adfs-host>
nslookup <exchange-adfs-host>

curl -I https://<agentic-retrieval-host>/chat/
curl https://<sharepoint-adfs-host>/adfs/.well-known/openid-configuration
curl https://<exchange-adfs-host>/adfs/.well-known/openid-configuration
```

Also test AD FS JWKS, SharePoint, and EWS from an approved diagnostics pod. A workstation test doesn't validate pod DNS, routes, firewalls, or trust.

## Configure AD FS connector applications

Configure SharePoint and Exchange as independent connector applications, even when they use the same AD FS farm. Each connector has its own public client, audience, scopes, and claims.

### Create public clients

Run these commands on the AD FS server as an AD FS administrator. Use a unique GUID for each client.

```powershell
Add-AdfsNativeClientApplication `
  -Name "Agentic Retrieval SharePoint Connector" `
  -ApplicationGroupIdentifier "AgenticRetrieval-SharePoint" `
  -Identifier "<sharepoint-public-client-id>" `
  -RedirectUri "https://agentic.contoso.com/chat/connector-callback.html"

Add-AdfsNativeClientApplication `
  -Name "Agentic Retrieval Exchange Connector" `
  -ApplicationGroupIdentifier "AgenticRetrieval-Exchange" `
  -Identifier "<exchange-public-client-id>" `
  -RedirectUri "https://agentic.contoso.com/chat/connector-callback.html"
```

These applications are public clients. Don't create or configure a client secret.

### Grant delegated scopes

Create or reuse one AD FS Web API identifier for SharePoint and one for Exchange. Grant each public client the delegated scopes that it requests.

```powershell
Grant-AdfsApplicationPermission `
  -ClientRoleIdentifier "<sharepoint-public-client-id>" `
  -ServerRoleIdentifier "<sharepoint-web-api-identifier>" `
  -ScopeNames "openid","profile","user_impersonation"

Grant-AdfsApplicationPermission `
  -ClientRoleIdentifier "<exchange-public-client-id>" `
  -ServerRoleIdentifier "<exchange-web-api-identifier>" `
  -ScopeNames "openid","profile","EWS.AccessAsUser.All"
```

If refresh tokens are approved, add `offline_access` to the applicable grant. Token persistence and refresh-token lifetime are customer security decisions.

Configure issuance rules so that:

- The SharePoint token contains the authenticated user's AD `primarySID` as one scalar claim named `primarysid`.
- The Exchange token contains the authenticated user's user principal name (UPN) as one scalar `upn` claim.
- The token contains the expected issuer, audience, expiry, and scopes.

Don't emit duplicate identity claims. Duplicate claims can become arrays and are rejected by downstream validation.

### Configure CORS

The browser calls the AD FS token endpoint directly. Append the exact Chat UI origin to the existing trusted origins:

```powershell
$origin = "https://agentic.contoso.com"
$current = @((Get-AdfsResponseHeaders).CORSTrustedOrigins)
$updated = @($current + $origin) |
  Where-Object { $_ } |
  Sort-Object -Unique

Set-AdfsResponseHeaders `
  -EnableCORS $true `
  -CORSTrustedOrigins $updated
```

The origin contains the scheme, host, and optional nondefault port. It doesn't contain a path or trailing slash. `Set-AdfsResponseHeaders` replaces the list, so always preserve the existing origins.

## Configure SharePoint high-trust authentication

Microsoft 365 Local supports `sharepointMcp.authMode=s2s_user`. Other SharePoint MCP authentication modes aren't supported in this release.

Use the same SharePoint high-trust certificate, trusted issuer, app principal, and Key Vault preparation described in [Set up SharePoint Server-to-Server authentication](connect-sharepoint-setup.md). For Microsoft 365 Local, also complete the following per-user requirements:

1. Point `sharepointMcp.siteUrl` to the SharePoint Windows-claims zone.
1. Make sure App Management, Subscription Settings, and User Profile service applications are running.
1. Grant the app principal the broad access required by your policy. The effective access is the intersection of app-principal and user permissions.
1. Create or synchronize a SharePoint User Profile for every user.
1. Grant content access to the Windows-claims identity, such as `i:0#.w|contoso\alice`.
1. Configure AD FS to emit the user's SID in the `primarysid` claim.

The SharePoint MCP validates the connector token's RS256 signature, exact issuer, exact audience, expiry, and optional scope before it trusts the SID. A missing or invalid token, or a token without the SID claim, is denied. The connector never falls back to a shared service account.

### Deliver the signing certificate

For a connected deployment, use Key Vault and workload identity:

```bash
az keyvault secret set \
  --vault-name <key-vault-name> \
  --name edgerag-sp-s2s-cert \
  --file ./agentic-retrieval-sharepoint-s2s.pfx \
  --encoding base64

az keyvault secret set \
  --vault-name <key-vault-name> \
  --name sp-cert-password \
  --value '<pfx-password>'
```

Grant the workload identity the **Key Vault Secrets User** role on the vault and create a federated credential whose subject is:

```text
system:serviceaccount:arc-rag:sharepoint-mcp-s2s-workload-identity
```

For a disconnected or fully air-gapped deployment, create the Kubernetes Secret before installing the extension:

```bash
kubectl create namespace arc-rag --dry-run=client -o yaml | kubectl apply -f -

kubectl create secret generic sharepoint-mcp-s2s-cert \
  --namespace arc-rag \
  --from-file=cert.pfx=./agentic-retrieval-sharepoint-s2s.pfx \
  --from-literal=cert-password='<pfx-password>' \
  --dry-run=client -o yaml | kubectl apply -f -
```

If SharePoint or AD FS uses a private CA, create a PEM bundle that contains the issuing chain:

```bash
kubectl create secret generic sharepoint-mcp-ca-bundle \
  --namespace arc-rag \
  --from-file=ca-bundle.crt=./contoso-sharepoint-adfs-ca.pem \
  --dry-run=client -o yaml | kubectl apply -f -
```

## Configure Exchange

Collect the following values from the Exchange and AD FS administrators:

| Value | Example | Extension setting |
|---|---|---|
| EWS endpoint | `https://mail.contoso.com/EWS/Exchange.asmx` | `exchangeMcp.ewsUrl` |
| AD FS authority | `https://fs.mail.contoso.com/adfs` | `connectorsRegistry.identityProviders.exchangeAdfs.authority` |
| Public client ID | Connector client GUID | `connectorsRegistry.identityProviders.exchangeAdfs.clientId` |
| Callback | `https://agentic.contoso.com/chat/connector-callback.html` | `connectorsRegistry.identityProviders.exchangeAdfs.redirectUri` |
| Web API audience | `https://mail.contoso.com` or the customer identifier | `connectorsRegistry.connections.exchange.resource` |
| Delegated scopes | `openid profile EWS.AccessAsUser.All` | `connectorsRegistry.connections.exchange.scope` |

Confirm that EWS is enabled, the Web API grant permits the delegated scope, and the token contains the user's scalar `upn` claim. The EWS TLS certificate must be trusted by the Exchange MCP container.

> [!NOTE]
> Exchange connector authentication uses AD FS in this public preview. Microsoft Entra ID authentication for hybrid Exchange deployments isn't supported in this release.

## Prepare the file caches

SharePoint and Exchange use separate disposable caches to pass downloaded binary files and attachments to Agents Runtime.

| Connector | Default claim | Default size | MCP mount | Runtime mount |
|---|---|---|---|---|
| SharePoint | `sp-file-cache-v2` | 50 GiB | `/var/cache/sp`, read/write | `/var/cache/sp`, read-only |
| Exchange | `ex-file-cache-v2` | 20 GiB | `/var/cache/exchange`, read/write | `/var/cache/exchange`, read-only |

Both claims must support `ReadWriteMany`. Don't use Azure Disk or another `ReadWriteOnce` storage class. The claims must remain separate because their capacity and cleanup lifecycle are independent.

For connected deployments, the default `azurefile-csi` storage class dynamically creates an Azure Files SMB share for each claim:

```bash
kubectl get csidriver file.csi.azure.com
kubectl get storageclass azurefile-csi -o yaml
```

Make sure cluster nodes can reach Azure Files on TCP 445. If public storage access is disabled, configure a private endpoint, private DNS, and node-subnet routing.

For disconnected deployments, use a customer-provided dynamic RWX class or create two static RWX claims before extension installation. When you provide existing claims, set:

```text
sharepointMcp.cache.createClaim=false
exchangeMcp.cache.createClaim=false
```

Cache data is temporary. Don't use these claims for source documents, ingestion state, backups, or application data.

## Install on a connected cluster

Set all placeholders before you run this example. The command enables both connectors, dynamic Azure Files caches, Key Vault certificate delivery, and private-CA trust. Remove a connector's settings if you don't enable that connector.

```bash
az k8s-extension create \
  --subscription <subscription-id> \
  --cluster-type connectedClusters \
  --cluster-name <cluster-name> \
  --resource-group <resource-group> \
  --name edgerag \
  --extension-type microsoft.arc.rag \
  --release-train <release-train> \
  --version <extension-version> \
  --auto-upgrade-minor-version false \
  --scope cluster \
  --release-namespace arc-rag \
  --config components=combined \
  --config isAldo=false \
  --config networkPolicies.enabled=true \
  --config m365Local.enabled=true \
  --config sharepointMcp.enabled=true \
  --config sharepointMcp.authMode=s2s_user \
  --config sharepointMcp.siteUrl=https://sp.contoso.com \
  --config sharepointMcp.s2s.clientId=<sharepoint-app-principal-guid> \
  --config sharepointMcp.s2s.issuerId=<sharepoint-trusted-issuer-guid> \
  --config sharepointMcp.s2s.realm=<sharepoint-farm-realm-guid> \
  --config sharepointMcp.s2s.sidClaim=primarysid \
  --config sharepointMcp.s2s.tokenJwksUri=https://fs.sp.contoso.com/adfs/discovery/keys \
  --config 'sharepointMcp.s2s.tokenIssuer=<exact-token-issuer>' \
  --config 'sharepointMcp.s2s.tokenAudience=<sharepoint-web-api-identifier>' \
  --config sharepointMcp.s2s.tokenScope=openid \
  --config sharepointMcp.s2s.certSource=keyvault \
  --config sharepointMcp.s2s.keyvaultName=<key-vault-name> \
  --config sharepointMcp.s2s.workloadIdentityClientId=<key-vault-mi-client-id> \
  --config sharepointMcp.s2s.kvTenantId=<key-vault-tenant-id> \
  --config sharepointMcp.s2s.kvCertObjectName=edgerag-sp-s2s-cert \
  --config sharepointMcp.s2s.kvCertPasswordObjectName=sp-cert-password \
  --config sharepointMcp.tls.caBundle.enabled=true \
  --config sharepointMcp.tls.caBundle.secretName=sharepoint-mcp-ca-bundle \
  --config sharepointMcp.tls.caBundle.key=ca-bundle.crt \
  --config sharepointMcp.cache.enabled=true \
  --config sharepointMcp.cache.createClaim=true \
  --config sharepointMcp.cache.storageClassName=azurefile-csi \
  --config exchangeMcp.enabled=true \
  --config exchangeMcp.ewsUrl=https://mail.contoso.com/EWS/Exchange.asmx \
  --config exchangeMcp.cache.enabled=true \
  --config exchangeMcp.cache.createClaim=true \
  --config exchangeMcp.cache.storageClassName=azurefile-csi \
  --config connectorsRegistry.identityProviders.sharepointAdfs.type=adfs \
  --config connectorsRegistry.identityProviders.sharepointAdfs.authority=https://fs.sp.contoso.com/adfs \
  --config connectorsRegistry.identityProviders.sharepointAdfs.clientId=<sharepoint-public-client-id> \
  --config connectorsRegistry.identityProviders.sharepointAdfs.clientType=public \
  --config connectorsRegistry.identityProviders.sharepointAdfs.redirectUri=https://agentic.contoso.com/chat/connector-callback.html \
  --config connectorsRegistry.connections.sharepoint.idp=sharepointAdfs \
  --config 'connectorsRegistry.connections.sharepoint.resource=<sharepoint-web-api-identifier>' \
  --config 'connectorsRegistry.connections.sharepoint.scope=openid profile user_impersonation' \
  --config connectorsRegistry.connections.sharepoint.mcpDestination=http://localhost:3500/v1.0/invoke/sharepoint-mcp-server/method/edgeai/mcp \
  --config connectorsRegistry.identityProviders.exchangeAdfs.type=adfs \
  --config connectorsRegistry.identityProviders.exchangeAdfs.authority=https://fs.mail.contoso.com/adfs \
  --config connectorsRegistry.identityProviders.exchangeAdfs.clientId=<exchange-public-client-id> \
  --config connectorsRegistry.identityProviders.exchangeAdfs.clientType=public \
  --config connectorsRegistry.identityProviders.exchangeAdfs.redirectUri=https://agentic.contoso.com/chat/connector-callback.html \
  --config connectorsRegistry.connections.exchange.idp=exchangeAdfs \
  --config 'connectorsRegistry.connections.exchange.resource=<exchange-web-api-identifier>' \
  --config 'connectorsRegistry.connections.exchange.scope=openid profile EWS.AccessAsUser.All' \
  --config connectorsRegistry.connections.exchange.mcpDestination=http://exchange-mcp-server.arc-rag.svc.cluster.local:3001/mcp
```

Add the standard authentication, ingress, and model settings from [Deploy the extension for Agentic Retrieval](deploy.md) to the same command.

## Install in a disconnected environment

First, import the Agentic Retrieval expansion pack and confirm that the local control plane recognizes `microsoft.arc.rag`. For instructions, see [Download and import the Agentic Retrieval expansion pack](disconnected-operations/prepare-disconnected.md).

Use the connected command as a base, with these changes:

```text
isAldo=true
auth.aadAuthorityHost=https://login.<disconnected-domain>
networking.domainName=<agentic-retrieval-host>.<disconnected-domain>

sharepointMcp.s2s.certSource=secret
sharepointMcp.s2s.certSecretName=sharepoint-mcp-s2s-cert

sharepointMcp.cache.createClaim=false
sharepointMcp.cache.claimName=sp-file-cache-v2
sharepointMcp.cache.storageClassName=<rw-storage-class>

exchangeMcp.cache.createClaim=false
exchangeMcp.cache.claimName=ex-file-cache-v2
exchangeMcp.cache.storageClassName=<rw-storage-class>
```

Use only in-environment AD FS, SharePoint, Exchange, DNS, storage, registry, identity, and model endpoints. A disconnected deployment doesn't require internet access at runtime, but every local dependency must remain reachable.

## Validate the deployment

### Check workloads and storage

```bash
az k8s-extension show \
  --cluster-type connectedClusters \
  --cluster-name <cluster-name> \
  --resource-group <resource-group> \
  --name edgerag \
  --query '{state:provisioningState,version:version}' -o table

kubectl get pods -n arc-rag
kubectl get pvc -n arc-rag sp-file-cache-v2 ex-file-cache-v2
```

The extension state must be `Succeeded`. Enabled MCP and Agents Runtime pods must be `Running` and ready. Both enabled claims must be `Bound` and show `RWX`.

### Create the Microsoft 365 Local agent

The installation registers these knowledge sources:

- `sharepoint-knowledge-agent`
- `exchange-knowledge-agent`

Use the Knowledge Sources API to get their IDs. Then create a knowledge base whose `knowledge_source_ids` contains the enabled sources and whose metadata identifies the connector tools:

```json
{
  "name": "M365 Local Agent",
  "description": "Connects to on-premises SharePoint and Exchange",
  "knowledge_source_ids": [
    "<sharepoint-knowledge-source-id>",
    "<exchange-knowledge-source-id>"
  ],
  "metadata": {
    "connections": "sharepoint,exchange",
    "icon_url": "/chat/agent-icons/m365.svg"
  }
}
```

Send the payload to `POST /knowledge-bases`. Agentic Retrieval creates the knowledge base and paired agent with the same ID. To update an existing agent, send the complete payload to `PATCH /knowledge-bases/{agent-id}`.

> [!IMPORTANT]
> On PATCH, `knowledge_source_ids` replaces the complete association list, and `metadata` replaces the complete metadata object. Include every source ID and metadata key that the agent must retain.

### Validate per-user access

1. Open the Chat UI at the final HTTPS hostname.
1. Connect SharePoint and Exchange separately.
1. Confirm that the browser reaches AD FS without DNS, TLS, redirect, or CORS errors.
1. Ask a SharePoint question that uses a binary Office document and an Exchange question that uses an attachment.
1. Test with two users who have different SharePoint and mailbox permissions. Each user must see only their authorized results.
1. Confirm that a missing or invalid connector token is denied.
1. Propose a supported write action. Confirm that no change occurs before approval, rejection causes no side effect, and an approved action runs once.

## Troubleshoot Microsoft 365 Local

| Symptom | Likely cause | Resolution |
|---|---|---|
| Connector card is absent | Connector registry entry is incomplete | Set the authority, client ID, redirect URI, resource, scope, and MCP destination |
| Browser reports a redirect mismatch | AD FS callback and extension callback differ | Register exactly `/chat/connector-callback.html` with the same scheme, host, and port |
| Browser token request is blocked by CORS | Chat UI origin is absent from AD FS CORS | Append the exact origin with `Set-AdfsResponseHeaders` |
| Browser can't open AD FS | DNS, routing, or TLS failure from the user network | Publish or route AD FS and install the trusted CA on clients |
| SharePoint MCP rejects all tokens | Wrong JWKS, issuer, audience, scope, or missing `primarysid` | Compare the settings with a newly issued connector token and correct AD FS issuance rules |
| Exchange call returns unauthorized | Wrong EWS URL, audience, scope, or missing `upn` | Verify the EWS Web API grant and token claims |
| Pod has `CreateContainerConfigError` | Referenced Secret or key is missing | Create the Secret with the exact configured name and keys |
| Key Vault mount fails | CSI provider, federation, RBAC, firewall, DNS, or tenant mismatch | Verify the CSI driver, service-account subject, **Key Vault Secrets User** role, and vault route |
| Cache claim remains `Pending` | Storage class can't provision RWX storage | Use an RWX class or create static PV and PVC pairs before installation |
| SharePoint returns 401 or 403 | High-trust issuer, app principal, User Profile, claims zone, or user permissions are incomplete | Recheck SharePoint high-trust and per-user configuration |
| TLS verification fails in a pod | Private CA bundle is missing or incomplete | Recreate the CA bundle Secret with the complete PEM issuing chain |

## Related content

- [MCP server in Agentic Retrieval](mcp-server-overview.md)
- [Knowledge bases in Agentic Retrieval](knowledge-bases-guide.md)
- [Knowledge sources in Agentic Retrieval](knowledge-sources-guide.md)
- [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md)
- [Known issues in Agentic Retrieval](known-issues.md)
