---
title: Prepare infrastructure for Microsoft 365 Local
description: "Prepare identity, SharePoint and Exchange trust, certificates, networking, and storage for Microsoft 365 Local in Agents and Tools with Foundry Local."
author: Lea1702
ms.author: leabenero
ms.topic: how-to
ms.date: 11/09/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
ms.custom:
  - sfi-ropc-nochange
---

# Prepare infrastructure for Microsoft 365 Local

> [!IMPORTANT]
> Microsoft 365 Local is in limited preview for approved customers. To request access, complete the [preview registration form](https://aka.ms/AgenticRetrievalM365LocalPuP).
>
> Microsoft 365 Local is supported only with Agentic Retrieval, through knowledge sources that you use in a knowledge base. It isn't supported as a general Agent Service integration.

## What is Microsoft 365 Local?

By using Microsoft 365 Local, Agentic Retrieval in the Agents and Tools with Foundry Local extension can retrieve on-premises SharePoint Server documents and Exchange Server email and calendars. *Agentic Retrieval* is the extension component that finds relevant information in your knowledge sources and returns it with citations. The extension runs managed Model Context Protocol (MCP) servers that retrieve current information from those systems at request time. You don't need to ingest that content into a separate index to use these sources. The source systems continue to enforce the signed-in user's permissions.

You use the sources for read-only retrieval: you add them to a knowledge base, and agents use that knowledge base. Retrieval doesn't grant additional access to SharePoint or Exchange content.

Administrators first prepare identity, source-system trust, certificates, networking, and storage as described in this article. Developers then [create SharePoint and Exchange knowledge sources](connect-microsoft-365-local.md), [check readiness](connect-microsoft-365-local.md#check-current-generation-readiness), and [use the sources through a knowledge base](connect-microsoft-365-local.md#use-the-sources-through-a-knowledge-base).

## Before you prepare the infrastructure

Complete these tasks before a developer creates a managed SharePoint or Exchange source through the Knowledge Sources API. API creation doesn't configure AD FS, register SharePoint farm trust, enable EWS OAuth, grant users source permissions, or publish signing credentials.

Before creating a source:

- Verify the [supported extension and server versions](#supported-releases-and-server-versions).
- Complete [preview entitlement and offline installation](#preview-entitlement-and-offline-installation). For a disconnected deployment, also complete the [disconnected environment checklist](#9-disconnected-environment-checklist).
- For SharePoint, complete [farm trust and user preparation](#5-prepare-sharepoint-farm-trust-and-users).
- For Exchange, complete [EWS OAuth trust and mailbox preparation](#6-prepare-exchange-ews-and-users), then [validate EWS OAuth before developer handoff](#validate-ews-oauth-before-developer-handoff).
- Configure and verify [shared cache storage during installation or update](#configure-shared-cache-storage-during-installation-or-update).

For the API requests and the steps that follow this preparation, see [Connect Microsoft 365 Local to Agents and Tools with Foundry Local extension](connect-microsoft-365-local.md). Creating a knowledge source isn't a substitute for these prerequisites.

This guide assumes version **1.0.0 or later** of the Agents and Tools with Foundry Local extension is already installed. If it isn't, start with the [deployment overview](deploy-overview.md) and [installation instructions](deploy.md). Prepare the RWX storage provider before installation, then return here to complete the source prerequisites.

> [!IMPORTANT]
> This procedure is for on-premises SharePoint Server and Exchange Server in the Microsoft 365 Local limited preview for approved customers. It isn't a SharePoint Online, Exchange Online, or Microsoft Graph configuration guide. Before using Microsoft 365 Local, complete [Preview entitlement and offline installation](#preview-entitlement-and-offline-installation) for the target subscription and verify [Supported releases and server versions](#supported-releases-and-server-versions).

All names, GUIDs, and domains in the examples are fictitious. Use customer-approved values and secret handling. Never copy production private keys, passwords, user tokens, or environment-specific identifiers into a documentation example or source repository.

### Preview entitlement and offline installation

Microsoft 365 Local requires a signed subscription entitlement (`entitlement.jws`) for the Azure subscription that hosts the extension. This entitlement is a single subscription-scoped prerequisite for both SharePoint and Exchange sources.

1. Request preview approval for Microsoft 365 Local by completing the preview registration form linked at the start of this article. Provide the Azure subscription ID that hosts the extension (`00000000-0000-0000-0000-000000000000` format). The entitlement is bound only to this subscription ID.

   Don't send cluster credentials, certificates, or private keys.

1. After approval, Microsoft issues a PS256-signed compact JSON Web Signature (JWS) file named `entitlement.jws` for that subscription ID. The file doesn't contain private key material.

1. Install `entitlement.jws` as the Kubernetes Secret `m365-preview-entitlement` in the extension's application namespace (`arc-rag` by default):

   ```bash
   NAMESPACE="arc-rag"
   kubectl create secret generic m365-preview-entitlement \
     --namespace "$NAMESPACE" \
     --from-file=entitlement.jws=./entitlement.jws \
     --dry-run=client -o yaml | kubectl apply -f -
   ```

1. If the extension is already running, restart the deployments that verify the entitlement:

   ```bash
   for deployment in knowledgesources-deployment agentic-retrieval-operator-deployment \
     agents-execution-deployment kb-execution-deployment; do
     kubectl rollout restart "deployment/$deployment" --namespace "$NAMESPACE"
     kubectl rollout status "deployment/$deployment" --namespace "$NAMESPACE" --timeout=5m
   done
   ```

   If Exchange sources already exist, also restart their MCP deployments, which have the label `edge-rag/component=exchange-mcp`.

Before Microsoft 365 Local source creation, provisioning, or runtime tool execution, the extension verifies the `entitlement.jws` signature offline against its built-in public key and confirms that the signed `subscriptionId` matches the subscription in the extension's Azure Resource Manager ID. This offline check works in both connected and disconnected environments. If the Secret is missing, invalid, or for another subscription, the Knowledge Sources API rejects Microsoft 365 Local source creation with `403 Forbidden` and the message `M365 Local is not enabled for this subscription.` Other extension features keep working.

## 1. Assign owners and collect values

| Owner | Task | Frequency | Developer handoff |
|---|---|---|---|
| Platform administrator | Install the managed-source capability and its networking, admission, runtime, model, and storage dependencies | Per installation or supported upgrade | Gateway URL, application audience, namespace, enabled capabilities |
| AD FS administrator | Configure connector application groups, public clients, resource audiences, scopes, claims, callbacks, and CORS | Per connector application/environment; update on URL or identity-policy changes | Authority, client ID, callback, audience, scopes; exact SharePoint token issuer and JWKS URI |
| SharePoint farm administrator | Register signing certificate, trusted issuer, app principal, Search entry point, and permitted operations | Per farm trust/environment; repeat applicable work on rotation | Public configuration identifiers; private PFX/password through the approved secret path |
| Exchange administrator | Enable and validate EWS OAuth trust, mailbox access, and source permissions | Per Exchange organization; user checks per mailbox | HTTPS EWS URL, validated AD FS audience/scope, and optional approved OWA base URL |
| Kubernetes security administrator | Delegate approved SharePoint material-publication permissions | Per installation/publisher | Confirmation that the designated administrator can publish protected resources |
| Kubernetes administrator | Publish immutable SharePoint binding, signing Secret, and optional CA ConfigMap; publish Exchange CA if needed | Per configuration revision | SharePoint binding name and Exchange CA ConfigMap name |
| AD/SharePoint administrators and mailbox owners | Prepare enabled identities, SharePoint User Profiles, content permissions, and mailboxes | Per user | Confirmation of access; never a shared username/password for the connector |
| Network/storage administrators | Validate DNS, TLS, routes, firewall rules, clock synchronization, and RWX storage | Per environment and topology change | Verified connectivity from browsers and workloads |

Users don't need a new AD FS application or SharePoint signing certificate for every sign-in. A managed source isn't a deployment per user.

### Supported releases and server versions

Verify that your environment meets these version requirements before configuring trust or creating sources:

| Component | Supported version or requirement |
|---|---|
| **Agents and Tools with Foundry Local extension** | Version **1.0.0 or later**, which includes Microsoft 365 Local (limited preview for approved customers), the `knowledgesources.foundrylocal.azure.com` custom resource definition (CRD), the `/knowledgesources` API, and the managed SharePoint and Exchange MCP services. See [Release notes](release-notes.md) and [Requirements](requirements.md). |
| **Active Directory Federation Services (AD FS)** | Windows Server 2019 AD FS or later, required for OAuth 2.0 authorization code flow with `S256` PKCE for public clients. |
| **SharePoint Server** | SharePoint Server Subscription Edition on-premises, configured with an HTTPS Windows-claims web application zone, High-Trust S2S (`s2s_user`), and the Search, App Management, Subscription Settings, and User Profile service applications. |
| **Exchange Server** | Exchange Server Subscription Edition (SE) on-premises, with HTTPS Exchange Web Services (EWS) enabled and AD FS OAuth trust (`New-AuthServer -Type ADFS`). See [Enable Modern Auth in Exchange Server on-premises](/exchange/plan-and-deploy/post-installation-tasks/enable-modern-auth-in-exchange-server-on-premises). |

Confirm that the installed extension includes the managed-source CRD:

```bash
kubectl get crd knowledgesources.foundrylocal.azure.com
```

### Installed platform versus source provisioning

Use version **1.0.0 or later** of the extension with managed SharePoint and Exchange source support. Create source workloads through the Knowledge Sources API after installation, not by turning on a per-server installation switch.

| Capability | Installation requirement |
|---|---|
| Managed SharePoint | Installed managed-source services, enforced network policies, admission controls, and an administrator-prepared RWX cache claim |
| Managed Exchange | Installed managed-source services and an administrator-prepared RWX cache claim; a static Exchange MCP isn't required |
| User experience | Browser connector authentication available and primary application authentication configured |
| Parsing and storage | Cache claims set for each provider you use, and parsing dependencies ready for the selected capabilities |

Don't use removed install-time M365 enablement flags. Don't put per-source EWS URLs, AD FS clients, SharePoint farm values, or signing passwords into the extension installation command. Administrators publish the approved configuration resources described in the following sections; developers select them in their source API requests.

Source provisioning and knowledge base configuration are separate steps. A ready source isn't automatically added to any knowledge base. Follow [Use the sources through a knowledge base](connect-microsoft-365-local.md#use-the-sources-through-a-knowledge-base) to add the source to a knowledge base and add that knowledge base to an agent.

Use the supported installation or upgrade procedure for your platform version. Don't bypass admission checks with an ad hoc Helm upgrade.

## 2. Verify infrastructure, DNS, TLS, and storage

Use a healthy supported Kubernetes cluster with a CNI that enforces NetworkPolicy. Ensure sufficient schedulable CPU and memory for each requested source and the installation's scaling bounds. Confirm the Kubernetes version supports the admission policies required by the managed-source installation.

### Network paths

| Source | Destination | Required connectivity |
|---|---|---|
| User browser | Application gateway | Trusted HTTPS on the final application hostname |
| User browser | Each connector's AD FS authority | HTTPS discovery, authorization, and token endpoints; correct CORS |
| SharePoint MCP | SharePoint Search entry and permitted content webs | HTTPS; Windows-claims URLs, resolvable names, trusted issuing CA |
| SharePoint MCP | AD FS JWKS | HTTPS for independent user-token validation |
| Exchange MCP | Exchange EWS | HTTPS with certificate verification and delegated OAuth |
| Kubernetes nodes and consuming workloads | Cache storage | Provider-required ports and routing; Azure Files SMB uses TCP 445, NFS commonly uses TCP 2049 |
| Installation | Required package/image/model sources | Approved connected endpoints or locally imported disconnected artifacts |
| Identity/source systems | DNS and time services | Correct resolution and synchronized clocks for token/certificate validation |

Check the existing M365 source and identity hostnames from a workstation on the real end-user network, not only from a server inside the datacenter:

```bash
nslookup fs.contoso.com
nslookup sharepoint.contoso.com
nslookup mail.contoso.com
curl --fail-with-body https://fs.contoso.com/adfs/.well-known/openid-configuration
curl --fail-with-body https://fs.contoso.com/adfs/discovery/keys
```

Check the source and identity endpoints separately from approved cluster diagnostics. A workstation's DNS and CA trust don't prove that pods can connect.

An unauthenticated EWS request can legitimately return 401 or 403. Verify DNS, connection, TLS, and expected endpoint behavior; don't treat that response as proof of mailbox authorization.

Don't disable TLS verification, use curl -k, disable NetworkPolicy, or accept an unverified root certificate as a workaround. Distribute verified CA roots to user devices through your approved management system.

### Shared file-cache requirements

Each Microsoft 365 Local provider that you use needs its own shared file-cache claim. The extension doesn't create these PersistentVolumeClaims (PVCs), StorageClasses, shares, or storage accounts. The administrator prepares them. Installations that don't use Microsoft 365 Local don't need these claims. The writer and readers can run on different nodes, so each claim must be a `Bound` ReadWriteMany (RWX) filesystem claim. Don't use an Azure Disk/ReadWriteOnce class, `hostPath`, or container-local storage for this shared handoff.

StorageClass is a cluster-wide resource. You can inspect it before the platform is installed:

```bash
kubectl get storageclass
```

PVCs are namespaced resources. Before creating Microsoft 365 Local knowledge sources, check that each cache claim that you configure exists and has a status of `Bound` in the existing Agents and Tools with Foundry Local extension application namespace. The following examples use `arc-rag`; replace it if the platform administrator installed the application in a different namespace.

```bash
NAMESPACE="arc-rag"
kubectl get namespace "$NAMESPACE"
kubectl get pvc --namespace "$NAMESPACE" <sharepoint-cache-claim> <exchange-cache-claim>
```

If the application namespace doesn't exist yet, complete the platform installation first. Don't create a new namespace just for Microsoft 365 Local sources or check PVCs in `default`. Use the same application namespace for protected configuration resources and every managed source.

Use the selected RWX provider's capacity, permissions, expansion, retention, and network guidance. A connected installation can use Azure Files where supported. A disconnected installation needs an available local RWX provider and its driver, or approved preprovisioned volumes. Managed SharePoint doesn't support `hostPath` storage.

| Provider | Extension setting | Default |
|---|---|---|
| SharePoint | `sharepointMcp.cache.claimName` | Empty: no SharePoint cache is configured |
| Exchange | `exchangeMcp.cache.claimName` | Empty: no Exchange cache is configured |

Configure only the providers that you use. When you use both, give each provider a separate claim and a separate backing share or export. Each claim must:

- Exist in the extension's application namespace and not be terminating.
- Be a `Bound` ReadWriteMany filesystem claim that every node running the MCP workloads, Agent Execution, and KB Execution can reach.
- Let the non-root workloads write and read. The workloads use the filesystem group set in `sharepointMcp.managed.fsGroup` (default `65532`).

For example, save the following as `m365-cache-pvcs.yaml`. Replace `<rwx-storage-class>` with an RWX StorageClass on your cluster, and set the sizes your administrator approves. The 50Gi and 20Gi values are sizing references, not minimums. Omit the claim for a provider you don't use.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: m365-sharepoint-cache
  namespace: arc-rag
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  storageClassName: <rwx-storage-class>
  resources:
    requests:
      storage: 50Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: m365-exchange-cache
  namespace: arc-rag
spec:
  accessModes:
    - ReadWriteMany
  volumeMode: Filesystem
  storageClassName: <rwx-storage-class>
  resources:
    requests:
      storage: 20Gi
```

Create the claims and check that they're `Bound`:

```bash
NAMESPACE="arc-rag"
kubectl apply --filename m365-cache-pvcs.yaml
kubectl get pvc --namespace "$NAMESPACE"
```

If the StorageClass uses `WaitForFirstConsumer` binding, complete the binding as part of storage preparation. The installation doesn't create a temporary pod to bind the claim. A `Bound` status alone doesn't prove that the storage works across nodes; test writes and reads on the target storage.

The extension mounts each configured cache into its MCP workloads and, read-only, into Agent Execution and KB Execution. Creating a PVC or knowledge source doesn't change the mounts of an existing installation.

The caches hold temporary downloaded content, not source documents or backups. Don't publish cache locations as a client API or manually edit the platform's source-scoped cache layout. The extension doesn't resize or delete the claims you supply. Don't delete the application namespace to troubleshoot storage, because that also deletes the claims.

### Configure shared cache storage during installation or update

Both Agent Execution and KB Execution must mount the provider's cache before they can use Microsoft 365 Local sources. Configure these mounts through the extension's configuration settings, not through individual knowledge-source requests.

- When you install the extension, add `sharepointMcp.cache.claimName=<claim-name>`, `exchangeMcp.cache.claimName=<claim-name>`, or both to the extension configuration settings.
- If the extension is already installed, prepare the claims and run an extension update before you create Microsoft 365 Local knowledge sources. For example:

  ```azurecli
  az k8s-extension update \
    --cluster-type connectedClusters \
    --cluster-name <cluster> \
    --resource-group <resource-group> \
    --name <extension-name> \
    --configuration-settings sharepointMcp.cache.claimName=m365-sharepoint-cache \
    --configuration-settings exchangeMcp.cache.claimName=m365-exchange-cache
  ```

  Keep your other installation settings unchanged.
- An installation or update stops if a configured claim is missing, terminating, not `Bound`, or not a ReadWriteMany filesystem claim.
- After installation or update, wait for the affected deployments to be ready:

  ```bash
  NAMESPACE="arc-rag"
  kubectl rollout status deployment/agents-execution-deployment --namespace "$NAMESPACE"
  kubectl rollout status deployment/kb-execution-deployment --namespace "$NAMESPACE"
  kubectl rollout status deployment/knowledgesources-deployment --namespace "$NAMESPACE"
  kubectl rollout status deployment/agentic-retrieval-operator-deployment --namespace "$NAMESPACE"
  ```

- Adding more knowledge sources for a provider whose cache is already configured doesn't require another extension update. Adding the other provider for the first time does.
- In later updates, keep the same claim names. An update that would remove or change a configured cache claim stops instead of moving or detaching the cache.

If a provider's cache setting is empty, creating a knowledge source for that provider returns `400 Bad Request` with a message that names `sharepointMcp.cache.claimName` or `exchangeMcp.cache.claimName`. The source isn't created. A missing or unsuitable claim also returns `400 Bad Request`. A claim that's still binding, or a temporary failure to inspect it, returns `503 Service Unavailable`. Fix the storage prerequisite, and then retry.

## 3. Register primary and connector callback URLs

Primary application sign-in and source-connector sign-in are separate.

| Value | Generic example | Where it belongs |
|---|---|---|
| Application origin | https://agentic.contoso.com | DNS, gateway certificate, application CORS, and AD FS trusted CORS origins |
| Primary sign-in redirect | https://agentic.contoso.com/ and https://agentic.contoso.com/chat/ | Primary application registration, following the deployment's authentication procedure |
| Connector callback | https://agentic.contoso.com/chat/connector-callback.html | Each connector's AD FS public client and corresponding source login configuration |
| SharePoint AD FS authority | https://fs.contoso.com/adfs | SharePoint binding login.authority |
| Exchange AD FS authority | https://fs.contoso.com/adfs | Exchange login.authority; a separate forest can use a different authority |
| SharePoint connector audience | urn:contoso:sharepoint | AD FS Web API identifier, binding tokenB.audience, and binding login.resource |
| Exchange connector audience | https://mail.contoso.com | Exchange-approved OAuth resource and Exchange login.resource |

Callbacks must match exactly: scheme, host, port, path, and trailing slash. For a nondefault ingress port or application base path, register the actual URL the deployed client uses. Don't mix a test origin with the production callback.

## 4. Configure AD FS connector applications

Use AD FS 2019 or later for browser-direct OAuth authorization code flow with S256 PKCE. Each connector needs a public/native client; don't configure a browser client secret.

The following example shows creating new application groups and clients. Don't rerun it blindly against existing applications. Use your approved naming and access-control policy and retain the issued client IDs.

Run as an AD FS administrator:

```powershell
$callback = "https://agentic.contoso.com/chat/connector-callback.html"
$sharePointClientId = [guid]::NewGuid().ToString()
$exchangeClientId = [guid]::NewGuid().ToString()
Add-AdfsApplicationGroup `
  -Name "Agents and Tools with Foundry Local extension SharePoint" `
  -ApplicationGroupIdentifier "FoundryLocal-SharePoint"
Add-AdfsApplicationGroup `
  -Name "Agents and Tools with Foundry Local extension Exchange" `
  -ApplicationGroupIdentifier "FoundryLocal-Exchange"
Add-AdfsNativeClientApplication `
  -Name "Agents and Tools with Foundry Local extension SharePoint browser client" `
  -ApplicationGroupIdentifier "FoundryLocal-SharePoint" `
  -Identifier $sharePointClientId `
  -RedirectUri $callback
Add-AdfsNativeClientApplication `
  -Name "Agents and Tools with Foundry Local extension Exchange browser client" `
  -ApplicationGroupIdentifier "FoundryLocal-Exchange" `
  -Identifier $exchangeClientId `
  -RedirectUri $callback
```

In AD FS Management, go to **Application Groups**, and create or select the corresponding Web API resource in each application group. Set its identifier to the approved audience and apply your organization's access-control policy. See [Add-AdfsWebApiApplication](/powershell/module/adfs/add-adfswebapiapplication).

### Claims and scopes

Configure issuance rules on the connector Web API, not merely on the application's primary sign-in registration.

| Connector | Required identity claim | Additional requirements |
|---|---|---|
| SharePoint | The authenticated user's Windows SID as one scalar primarysid claim, or the exact configured SID claim name | Correct iss, aud, expiry, and requested scope; login.resource must equal tokenB.audience |
| Exchange | The authenticated user's UPN as one scalar upn claim | EWS must accept the issuer, audience, delegated scope, and mailbox identity |

For SharePoint, map the authenticated AD user's primary SID to the configured outgoing claim. Don't put a fixed administrator SID in the rule or binding. For Exchange, configure the UPN issuance rule so the resulting JWT contains upn. Claims mappings must reflect the authenticated user.

Don't duplicate registered or identity claims. Verify that the token has scalar identity values rather than arrays. Obtain the exact issuer from your AD FS configuration or a securely inspected token; don't derive it from the authority hostname. An issuer identifier can differ from the HTTPS discovery URL.

Grant only the registered scopes the connector requests. After the required Web API scopes exist, a typical grant is:

```powershell
Grant-AdfsApplicationPermission `
  -ClientRoleIdentifier $sharePointClientId `
  -ServerRoleIdentifier "urn:contoso:sharepoint" `
  -ScopeNames "openid","user_impersonation"
Grant-AdfsApplicationPermission `
  -ClientRoleIdentifier $exchangeClientId `
  -ServerRoleIdentifier "https://mail.contoso.com" `
  -ScopeNames "openid","profile","EWS.AccessAsUser.All"
```

The Exchange scope must be the one accepted by your EWS/AD FS configuration. A grant alone isn't enough: the login configuration must request it, and the resulting token must contain the expected claim. If a SharePoint binding sets requiredScope, include that value in login.scope and grant it.

Set refresh-token policy according to your organization's requirements. Don't assume automatic renewal is available in every AD FS configuration.

### CORS

The browser calls AD FS directly. Append the application origin without removing existing trusted origins:

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

CORS origins contain scheme, host, and optional port, not a path or trailing slash. Apply the setting to every AD FS farm used by the connectors. Preserve other applications' origins.

## 5. Prepare SharePoint farm trust and users

Complete the following farm and user preparation for a [supported SharePoint Server version](#supported-releases-and-server-versions).

### Farm services and Search

The farm administrator must verify:

- The farm runs **SharePoint Server Subscription Edition** on-premises.
- The query URL uses the HTTPS Windows-claims zone and is a valid SharePoint Search entry point.
- The Search service is healthy, crawled content is available, and expected results are security trimmed.
- App Management, Subscription Settings, and User Profile service applications and their proxies are configured for the web application.
- Each participating user has a synchronized or created SharePoint User Profile and the appropriate Windows-claims identity.

Run in the SharePoint Management Shell:

```powershell
Get-SPServiceApplication | Select-Object DisplayName, TypeName
Get-SPServiceApplicationProxy | Select-Object DisplayName, TypeName
Get-SPAuthenticationRealm
```

Create missing service applications through your normal farm administration procedure. This guide doesn't create service databases, change the farm account, or enable OAuth over HTTP.

### Signing certificate

Use an administrator-approved RSA signing certificate with a private key. SharePoint trusts its public certificate. Deliver its PFX and password to the managed MCP through a protected Kubernetes Secret.

The signing certificate is different from the HTTPS certificates on AD FS and SharePoint. Publishing a CA bundle doesn't register the signing certificate with the farm.

For example, export an already-issued certificate on a secured administration host:

```powershell
$certificate = Get-Item "Cert:\LocalMachine\My\<signing-certificate-thumbprint>"
if (-not $certificate.HasPrivateKey) {
  throw "The signing certificate must include its private key."
}
$pfxPassword = Read-Host "PFX export password" -AsSecureString
Export-PfxCertificate `
  -Cert $certificate `
  -FilePath "C:\SecureTransfer\sharepoint-signing.pfx" `
  -Password $pfxPassword
Export-Certificate `
  -Cert $certificate `
  -FilePath "C:\SecureTransfer\sharepoint-signing.cer"
```

Use a protected transfer location and remove temporary exports according to your security policy. Don't use a self-signed test certificate as a production shortcut.

### Register high trust

Run in the SharePoint Management Shell as an authorized farm administrator. Use new identifiers for a new trust; coordinate existing registrations and rotations rather than creating duplicates.

```powershell
$publicCertificate = New-Object `
  System.Security.Cryptography.X509Certificates.X509Certificate2(
    "C:\SecureTransfer\sharepoint-signing.cer"
  )
$issuerId = [guid]::NewGuid().ToString()
$realm = Get-SPAuthenticationRealm
New-SPTrustedRootAuthority `
  -Name "Agents and Tools with Foundry Local extension signing certificate" `
  -Certificate $publicCertificate
New-SPTrustedSecurityTokenIssuer `
  -Name "Agents and Tools with Foundry Local extension high trust" `
  -Certificate $publicCertificate `
  -RegisteredIssuerName "$issuerId@$realm" `
  -IsTrustBroker
```

Record the issuer GUID and farm realm GUID for the binding. The farm realm is the GUID from Get-SPAuthenticationRealm, not a WS-Federation sign-in realm string.

Register the app principal and grant approved access:

```powershell
$clientId = [guid]::NewGuid().ToString()
$site = Get-SPSite "https://sharepoint.contoso.com/sites/knowledge"
$web = $site.RootWeb
$principal = Register-SPAppPrincipal `
  -NameIdentifier "$clientId@$realm" `
  -Site $web `
  -DisplayName "Agents and Tools with Foundry Local extension"
Set-SPAppPrincipalPermission `
  -Site $web `
  -AppPrincipal $principal `
  -Scope SiteCollection `
  -Right Read
```

This grants retrieval access only, which is what Agentic Retrieval needs. Effective access remains bounded by both the app principal and the signed-in user.

Use the same approved client identity when authorizing additional site collections in the intended search scope. Don't generate a different client ID for each site while expecting one binding to use them all.

See [Create high-trust SharePoint Add-ins](/sharepoint/dev/sp-add-ins/create-high-trust-sharepoint-add-ins) and [Package and publish high-trust SharePoint Add-ins](/sharepoint/dev/sp-add-ins/package-and-publish-high-trust-sharepoint-add-ins) for production farm prerequisites and certificate handling.

### Per-user preparation

For each participating user:

- Verify the AD account is enabled and has the expected primary SID and UPN.
- Create or synchronize the SharePoint User Profile.
- Grant access using the correct Windows-claims identity, such as i:0#.w\|contoso\\alice, through the normal site-permission process.
- Verify AD FS issues that user's SID, not a shared configured SID.
- Test both permitted and denied content with two users.

Don't confuse this live per-user flow with indexed SharePoint ingestion, which has its own ingestion identity and certificate-selection contract.

## 6. Prepare Exchange EWS and users

The Exchange administrator must:

- Verify that the organization runs **Exchange Server Subscription Edition (SE)** on-premises, with healthy mailboxes and an HTTPS EWS URL (`https://mail.contoso.com/EWS/Exchange.asmx`).
- Ensure the Exchange servers trust the AD FS HTTPS TLS certificate chain in the machine root store (`Cert:\LocalMachine\Root`).
- Configure Exchange to trust the AD FS OAuth authorization server and its federation metadata (`https://<adfs-host>/FederationMetadata/2007-06/FederationMetadata.xml`).
- Enable OAuth on the applicable EWS virtual directory under the organization's change process.
- Confirm the AD FS resource audience (`https://mail.contoso.com`) and scopes (`openid profile EWS.AccessAsUser.All`) match what EWS accepts.
- Confirm each user has a valid mailbox and the delegated permissions needed for the enabled operations.
- Verify Exchange and the MCP can validate the relevant HTTPS certificate chains.
- Optional: for clickable email and calendar citations, obtain the approved HTTPS Outlook on the web (OWA) base URL, such as `https://mail.contoso.com/owa`. Verify that users can reach it, trust its certificate, and sign in to their mailbox. The developer sets this URL for each Exchange knowledge source in `exchange_parameters.owa_url`. It's separate from the EWS endpoint, and it must not contain credentials, query parameters, or a fragment.

Inspect existing configuration in Exchange Management Shell before changing it:

```powershell
Get-ExchangeServer |
  Select-Object Name, Edition, AdminDisplayVersion
Get-WebServicesVirtualDirectory |
  Select-Object Identity, InternalUrl, ExternalUrl, OAuthAuthentication
Get-AuthServer |
  Select-Object Name, Type, Enabled, AuthMetadataUrl
Get-CASMailbox -Identity "alice@contoso.com" |
  Select-Object PrimarySmtpAddress, EwsEnabled
```

If AD FS isn't registered yet as an Exchange authorization server, register the AD FS federation metadata endpoint. If `OAuthAuthentication` is `False` on an EWS virtual directory, enable it:

```powershell
$adfsMetadataUrl = "https://fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml"

New-AuthServer `
  -Name "Contoso AD FS" `
  -Type ADFS `
  -AuthMetadataUrl $adfsMetadataUrl `
  -Enabled $true

Get-WebServicesVirtualDirectory |
  Set-WebServicesVirtualDirectory -OAuthAuthentication $true

Get-AuthServer "Contoso AD FS" |
  Select-Object Name, Type, Enabled, AuthMetadataUrl
Get-WebServicesVirtualDirectory |
  Select-Object Identity, InternalUrl, ExternalUrl, OAuthAuthentication
```

Verify that `https://fs.contoso.com/FederationMetadata/2007-06/FederationMetadata.xml` is reachable over HTTPS from each Exchange server before running `New-AuthServer`. Don't use `-TrustAnySSLCertificate`, enable Basic authentication as a workaround, or overwrite an existing organization-wide authentication configuration. See [New-AuthServer](/powershell/module/exchangepowershell/new-authserver) and [Set-WebServicesVirtualDirectory](/powershell/module/exchangepowershell/set-webservicesvirtualdirectory).

The full Exchange procedure for AD FS OAuth also covers the default authorization endpoint (`Set-AuthServer -IsDefaultAuthorizationEndpoint`), the organization setting `Set-OrganizationConfig -OAuth2ClientProfileEnabled`, and authentication policies that allow or block modern authentication per user. These settings also change how Outlook and other clients sign in. Plan and apply them by following [Configure Exchange Server to use ADFS OAuth tokens](/exchange/plan-and-deploy/post-installation-tasks/enable-modern-auth-in-exchange-server-on-premises#configure-exchange-server-to-use-adfs-oauth-tokens). If a user's authentication policy blocks modern authentication for EWS (`BlockModernAuthWebServices`), the Exchange connector can't access that user's mailbox.

### Validate EWS OAuth before developer handoff

Before you hand the Exchange settings to the developer, verify both that EWS is reachable and that an authenticated delegated EWS call succeeds:

1. Confirm that DNS, TLS, and the EWS endpoint respond. An unauthenticated request returns `401 Unauthorized`:

   ```bash
   curl --silent --show-error --dump-header - --output /dev/null \
     "https://mail.contoso.com/EWS/Exchange.asmx" \
     | grep -iE "^(HTTP/|WWW-Authenticate:)"
   ```

   Expected output includes `HTTP/1.1 401` (or `HTTP/2 401`) and the `WWW-Authenticate` schemes that EWS offers. This response proves only reachability and TLS. It doesn't prove that OAuth or mailbox access works.

1. Obtain a delegated AD FS access token (`EWS_TOKEN`) for a test mailbox user with audience `https://mail.contoso.com`, scopes `openid profile EWS.AccessAsUser.All`, and a scalar `upn` claim, then send a read-only SOAP `GetFolder` request for `inbox`:

   ```bash
   curl --fail-with-body --silent --show-error \
     --request POST \
     --header "Authorization: Bearer $EWS_TOKEN" \
     --header "Content-Type: text/xml; charset=utf-8" \
     --data '<?xml version="1.0" encoding="utf-8"?>
   <soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
                  xmlns:t="http://schemas.microsoft.com/exchange/services/2006/types"
                  xmlns:m="http://schemas.microsoft.com/exchange/services/2006/messages">
     <soap:Header>
       <t:RequestServerVersion Version="Exchange2013_SP1"/>
     </soap:Header>
     <soap:Body>
       <m:GetFolder>
         <m:FolderShape><t:BaseShape>IdOnly</t:BaseShape></m:FolderShape>
         <m:FolderIds><t:DistinguishedFolderId Id="inbox"/></m:FolderIds>
       </m:GetFolder>
     </soap:Body>
   </soap:Envelope>' \
     "https://mail.contoso.com/EWS/Exchange.asmx"
   ```

   - **Expected success:** HTTP `200 OK` with `<m:ResponseCode>NoError</m:ResponseCode>` in the SOAP response body.
   - **Expected failure control:** Repeating the `POST` without the `Authorization` header returns HTTP `401 Unauthorized`.

Mailbox existence alone doesn't authorize every operation. Validate permitted mail and calendar reads and permission-denied cases. Don't submit a shared mailbox password, service-account password, or client secret in the knowledge-source API request.

## 7. Publish the SharePoint configuration revision

Publish these resources in the installation's application namespace after the platform is ready and before creating the source:

| Resource | Example name | Contents |
|---|---|---|
| Immutable binding ConfigMap | sp-auth-corp-sharepoint.0000000000000001 | Non-secret binding.json |
| Immutable signing Secret | sp-cred-corp-sharepoint.0000000000000001 | cert.pfx and UTF-8 cert-password |
| Immutable source CA ConfigMap | sp-ca-corp-sharepoint.0000000000000001 | ca-bundle.pem; required for tls.caSource: source |

The configuration name can differ from the source name and can be shared only where the administrator approves that reuse. Each revision is exactly 16 lowercase hexadecimal characters. Never reuse the name of a deleted revision. To add another SharePoint farm or a different authentication context, publish a separate configuration and use it in a new source. See [Add more SharePoint or Exchange servers](connect-microsoft-365-local.md#add-more-sharepoint-or-exchange-servers).

### Publication permission

To publish signing material and approved bindings, you need both the relevant namespace resource permissions and native permission to update the installation's named SharePoint resource admission policy. A generic namespace role or developer app role isn't sufficient.

The Kubernetes security administrator must delegate this capability to the designated publisher, not to ordinary source authors or execution workloads. Check permission without changing the policy:

```bash
NAMESPACE="arc-rag" # Existing Agents and Tools with Foundry Local extension application namespace
kubectl auth can-i \
  update "validatingadmissionpolicies.admissionregistration.k8s.io/${NAMESPACE}-sharepoint-managed-resource-policy"
kubectl auth can-i create secrets --namespace "$NAMESPACE"
kubectl auth can-i create configmaps --namespace "$NAMESPACE"
```

Don't disable or edit admission protections to make a denied publication succeed.

### Strict binding JSON

Save the following file as `sharepoint-binding.json`. All fields are required. Replace the sample GUIDs and identifiers with the values you collected earlier.

```json
{
  "schemaVersion": 1,
  "configurationName": "corp-sharepoint",
  "credentialRevision": "0000000000000001",
  "querySiteUrl": "https://sharepoint.contoso.com/sites/knowledge",
  "s2s": {
    "realm": "aaaabbbb-0000-cccc-1111-dddd2222eeee",
    "clientId": "00001111-aaaa-2222-bbbb-3333cccc4444",
    "issuerId": "bbbbcccc-1111-dddd-2222-eeee3333ffff"
  },
  "tokenB": {
    "issuer": "http://fs.contoso.com/adfs/services/trust",
    "audience": "urn:contoso:sharepoint",
    "jwksUri": "https://fs.contoso.com/adfs/discovery/keys",
    "sidClaim": "primarysid",
    "requiredScope": "user_impersonation"
  },
  "login": {
    "authority": "https://fs.contoso.com/adfs",
    "clientId": "11112222-bbbb-3333-cccc-4444dddd5555",
    "redirectUri": "https://agentic.contoso.com/chat/connector-callback.html",
    "resource": "urn:contoso:sharepoint",
    "scope": "openid user_impersonation"
  },
  "tls": {
    "caSource": "source"
  }
}
```

`tokenB.issuer` is an exact issuer identifier, not an instruction to contact an HTTP endpoint. All discovery and data transport must use HTTPS. Unknown fields, user identities, credentials, and MCP destination URLs don't belong in this JSON.

### Publish protected resources

The following Bash examples assume `jq`, a secured local PFX, a protected UTF-8 password file without an unintended newline, and a verified CA PEM bundle. Don't write those secret files into the repository or print generated Secret manifests.

```bash
set -euo pipefail
NAMESPACE="arc-rag" # Existing Agents and Tools with Foundry Local extension application namespace
REVISION="0000000000000001"
CONFIGURATION="corp-sharepoint"
kubectl create secret generic "sp-cred-${CONFIGURATION}.${REVISION}" \
  --namespace "$NAMESPACE" \
  --from-file=cert.pfx=/secure/sharepoint-signing.pfx \
  --from-file=cert-password=/secure/certificate-password.txt \
  --dry-run=client --output=json \
  | jq '.immutable = true' \
  | kubectl create --filename=-
kubectl create configmap "sp-ca-${CONFIGURATION}.${REVISION}" \
  --namespace "$NAMESPACE" \
  --from-file=ca-bundle.pem=/secure/sharepoint-adfs-ca.pem \
  --dry-run=client --output=json \
  | jq '.immutable = true' \
  | kubectl create --filename=-
kubectl create configmap "sp-auth-${CONFIGURATION}.${REVISION}" \
  --namespace "$NAMESPACE" \
  --from-file=binding.json=./sharepoint-binding.json \
  --dry-run=client --output=json \
  | jq '.immutable = true' \
  | kubectl create --filename=-
```

These are create-only examples. An existing name is a conflict to inspect, not permission to overwrite a revision. For platform CA trust, set tls.caSource to platform and omit the source CA ConfigMap.

Keep these administrator resources independent of knowledge-source ownership so that source deletion doesn't delete the configuration or signing credentials.

## 8. Prepare an Exchange source CA bundle

When EWS requires source-specific CA trust, publish the approved issuing chain before creating the source:

```bash
NAMESPACE="arc-rag" # Existing Agents and Tools with Foundry Local extension application namespace
kubectl create configmap ex-ca-corp-exchange.0000000000000001 \
  --namespace "$NAMESPACE" \
  --from-file=ca-bundle.pem=/secure/exchange-ca.pem \
  --dry-run=client --output=json \
  | jq '.immutable = true' \
  | kubectl create --filename=-
```

Use the existing Agents and Tools with Foundry Local extension application namespace selected earlier. The name format is ex-ca-\<knowledge-source-name\>.\<16-lowercase-hex\>. Use that exact name in exchange_parameters.tls.ca_config_map_name. The service resolves its Kubernetes UID; the caller doesn't supply one. Don't delete and recreate the object under the same name. An additional Exchange source that needs source-specific trust gets its own correctly named CA ConfigMap in that same namespace.

Use `ca_source: platform` only when the installation's effective platform trust validates EWS. Don't disable TLS verification.

## 9. Disconnected environment checklist

Complete the same farm, mailbox, AD FS, claim, callback, and publication tasks in the disconnected environment. In addition:

- Import the supported extension package, images, and required model artifacts before installation.
- Use the local control plane, application registration, and sign-in authority for primary application authentication.
- Register the actual disconnected application's HTTPS connector callback, not the connected deployment's hostname.
- Ensure user devices as well as pods can resolve and reach the local AD FS endpoints.
- Pre-stage verified CA roots, signing material, configuration revisions, and the `m365-preview-entitlement` Secret described in [Preview entitlement and offline installation](#preview-entitlement-and-offline-installation).
- Use available local RWX storage with the required node/workload permissions.
- Confirm runtime operation doesn't depend on public Key Vault, public AD FS discovery, public DNS, or an internet-only model endpoint.

Disconnected operation doesn't mean that authentication or TLS can be skipped.

## 10. Pre-run acceptance checklist

Before you hand off the binding and connection values to the developer, confirm the following items:

> [!div class="checklist"]
> * Extension version **1.0.0 or later**, managed capabilities, and the subscription's [preview entitlement](#preview-entitlement-and-offline-installation) (`m365-preview-entitlement` Secret) are ready.
> * Gateway and AD FS hostnames resolve from the actual user network.
> * Browser and pod TLS validation succeeds without bypasses.
> * Primary sign-in redirects and both connector callbacks exactly match the deployed URLs.
> * AD FS CORS preserves existing origins and includes the application origin.
> * Each public client has its required Web API grant and requests the approved scopes.
> * SharePoint tokens have the correct scalar SID; Exchange tokens have the correct scalar UPN.
> * SharePoint issuer, audience, HTTPS JWKS, and required scope match the published binding.
> * SharePoint farm trust, app principal, Search, User Profile, and user permissions are ready.
> * EWS OAuth trust, TLS, mailbox enablement, audience, scope, and user permissions are ready, and [EWS OAuth validation](#validate-ews-oauth-before-developer-handoff) is complete.
> * Optional: the approved HTTPS OWA base URL for each Exchange source is handed to the developer. Users can reach it, trust its certificate, and sign in to their mailbox.
> * Protected SharePoint revision resources and optional Exchange CA exist in the correct namespace.
> * The existing Agents and Tools with Foundry Local extension namespace is confirmed. Each provider you use has a `Bound` RWX cache claim set in `sharepointMcp.cache.claimName` or `exchangeMcp.cache.claimName`, and its storage routes are ready; managed SharePoint doesn't use hostPath.
> * Two-user positive and negative permission tests are scheduled.
> * A certificate-rotation owner and expiry-monitoring process are assigned.

After these checks, follow [Connect Microsoft 365 Local to Agents and Tools with Foundry Local extension](connect-microsoft-365-local.md) to create sources through the API.

## Certificate rotation

Before the certificate expires, register the replacement public certificate with SharePoint, publish a new signing Secret and binding revision, and PATCH the existing source to reference that revision. Wait for current-generation readiness and verify per-user access before retiring old farm trust or credentials.

Account for every source sharing a revision. Source deletion doesn't remove administrator resources, and a failed rotation doesn't authorize automatic fallback to an old credential.
