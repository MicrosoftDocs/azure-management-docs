---
title: Set up SharePoint server-to-server authentication
description: Learn how to configure a certificate Secret and SharePoint Server authentication for indexed ingestion with Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - sfi-ropc-nochange
#customer intent: As a platform administrator, I want to set up SharePoint server-to-server authentication so that I can securely ingest on-premises SharePoint data with Agents and Tools with Foundry Local.
---

# Set up SharePoint server-to-server authentication

This article explains how to set up High-Trust Server-to-Server (S2S) authentication for indexed ingestion from SharePoint Server Subscription Edition.

SharePoint ingestion is enabled by default in Knowledge and combined deployments. You don't provide deployment-wide SharePoint credentials. For each new SharePoint source, you select an existing Kubernetes Secret and provide the SharePoint identity values for that source.

## Prerequisites

Before you begin, ensure you have:

- Farm Administrator access to SharePoint Management Shell.
- Access to the cluster by using `kubectl`.
- A SharePoint Server Subscription Edition site and document library.
- A Windows account for ingestion, with a Windows SID, SharePoint User Profile, and read access to the selected content.
- DNS and HTTPS connectivity from the application namespace to the SharePoint Windows-claims zone.
- A certificate chain that the ingestion workers trust. Don't disable TLS verification in production.

Estimated time: approximately 2 hours.

## Step 1: Verify network connectivity

Set the application namespace and SharePoint host. Replace `arc-rag` if your extension uses another namespace.

```bash
NAMESPACE="arc-rag"
SHAREPOINT_HOST="<sharepoint_fqdn>"

kubectl run netcheck \
  --namespace "$NAMESPACE" \
  --image=busybox:1.36 \
  --restart=Never \
  -- sleep 300

kubectl wait \
  --namespace "$NAMESPACE" \
  --for=condition=Ready pod/netcheck \
  --timeout=90s

kubectl exec --namespace "$NAMESPACE" netcheck -- nslookup "$SHAREPOINT_HOST"
kubectl exec --namespace "$NAMESPACE" netcheck -- nc -zvw5 "$SHAREPOINT_HOST" 443
kubectl delete pod --namespace "$NAMESPACE" netcheck --ignore-not-found
```

Resolve DNS, firewall, proxy, routing, or certificate-trust issues before you continue.

## Step 2: Create the signing certificate

Run the following PowerShell on a protected Windows machine:

```powershell
$cert = New-SelfSignedCertificate `
    -Subject "CN=FoundryLocal-SharePoint-S2S" `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -KeyExportPolicy Exportable `
    -KeySpec KeyExchange `
    -KeyLength 2048 `
    -HashAlgorithm SHA256 `
    -NotAfter (Get-Date).AddYears(2) `
    -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider"

$pfxPassword = ConvertTo-SecureString "<your_pfx_password>" -AsPlainText -Force

Export-PfxCertificate `
    -Cert $cert `
    -FilePath "C:\sharepoint-s2s.pfx" `
    -Password $pfxPassword

Export-Certificate `
    -Cert $cert `
    -FilePath "C:\sharepoint-s2s.cer"

Write-Output "Certificate thumbprint: $($cert.Thumbprint)"
```

Protect the PFX file and its password. Don't commit them to source control.

## Step 3: Create the certificate Secret

The person creating the Secret needs `create` permission on Secrets in the application namespace. Rotation also requires `update` or `patch`, depending on the command you use.

```bash
NAMESPACE="arc-rag"
kubectl auth can-i create secrets --namespace "$NAMESPACE"
```

The extension installs the ingestion service account's permission to read selected Secrets. A person who submits ingestion through the external API needs an access token and an Azure role that allows ingestion, not direct Kubernetes Secret-read access. Assign **Cognitive Services Contributor** at the extension scope, or **Contributor** in disconnected deployments. The `EdgeRAGDeveloper` app role also works as a fallback. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).

Create the Secret in the application namespace. The certificate data key must be `cert.pfx` and must be nonempty. The password data key must exist even when the PFX password is empty.

```powershell
$namespace = "arc-rag"
[System.IO.File]::WriteAllText("$PWD\cert-password.txt", "<your_pfx_password>")

kubectl create secret generic sharepoint-s2s-cert `
  --namespace $namespace `
  --from-file=cert.pfx=C:\sharepoint-s2s.pfx `
  --from-file=cert-password="$PWD\cert-password.txt"

Remove-Item cert-password.txt
```

For an unencrypted PFX, use an empty password file so the `cert-password` key still exists.

Check only the Secret metadata and key names. Don't print or decode credential values.

```bash
kubectl describe secret sharepoint-s2s-cert --namespace "$NAMESPACE"
```

## Step 4: Configure SharePoint Server

Run this step on the SharePoint server in SharePoint Management Shell as a Farm Administrator.

For the complete SharePoint high-trust model and farm prerequisites, see [Create high-trust SharePoint Add-ins](/sharepoint/dev/sp-add-ins/create-high-trust-sharepoint-add-ins) and [Package and publish high-trust SharePoint Add-ins](/sharepoint/dev/sp-add-ins/package-and-publish-high-trust-sharepoint-add-ins).

### Verify required service applications

The target web application needs App Management and Subscription Settings service applications and proxies. This ingestion flow also needs a User Profile service application and proxy so SharePoint can resolve the selected ingestion-account SID.

```powershell
Get-SPServiceApplication |
    Select-Object DisplayName, TypeName

Get-SPServiceApplicationProxy |
    Select-Object DisplayName, TypeName

Get-SPDatabase |
    Select-Object Name, TypeName, Server
```

Confirm that the required service applications and proxies are present and associated with the target web application. In the database inventory, identify the row whose `TypeName` is **Configuration Database**. Don't assume that its name is `SharePoint_Config`.

If a required service application is missing, create it through Central Administration or your approved farm-provisioning process. For Microsoft's reference scripts, see [High-trust configuration scripts for SharePoint](/sharepoint/dev/sp-add-ins/high-trust-configuration-scripts-for-sharepoint). Don't run a generic database-creation script without the farm administrator approving the application pool, managed account, database server, and database names.

### Register the certificate and trusted issuer

```powershell
Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2(
    "C:\sharepoint-s2s.cer"
)

New-SPTrustedRootAuthority `
    -Name "FoundryLocal-SharePoint-S2S" `
    -Certificate $cert

$issuerId = [guid]::NewGuid().ToString()
$realm = Get-SPAuthenticationRealm
$registeredIssuerName = "$issuerId@$realm"

New-SPTrustedSecurityTokenIssuer `
    -Name "FoundryLocal-SharePoint-S2S" `
    -Certificate $cert `
    -RegisteredIssuerName $registeredIssuerName `
    -IsTrustBroker

Write-Output "Issuer ID: $issuerId"
Write-Output "Realm: $realm"
```

Save the Issuer ID and realm. Each SharePoint source must use the Issuer ID registered for its signing certificate.

### Register the app principal with read access

```powershell
Set-SPAppDomain "<your_ad_domain>"
Set-SPAppSiteSubscriptionName -Name "app" -Confirm:$false

$clientId = [guid]::NewGuid().ToString()
$realm = Get-SPAuthenticationRealm
$appId = "$clientId@$realm"
$site = Get-SPSite "<your_sharepoint_site_url>"
$web = $site.RootWeb

$principal = Register-SPAppPrincipal `
    -NameIdentifier $appId `
    -Site $web `
    -DisplayName "Foundry Local SharePoint ingestion"

Set-SPAppPrincipalPermission `
    -Site $web `
    -AppPrincipal $principal `
    -Scope SiteCollection `
    -Right Read

Write-Output "Client ID: $clientId"
```

Register the app principal and grant read access in every site collection that the source must access.

### Prepare the ingestion account

The selected ingestion account also needs read access to the target library. A SharePoint User Profile makes its SID resolvable but doesn't grant content access.

```powershell
$account = "<DOMAIN\service_account>"
$site = Get-SPSite "<your_sharepoint_site_url>"
$context = Get-SPServiceContext $site
$upm = New-Object Microsoft.Office.Server.UserProfiles.UserProfileManager($context)

if (-not $upm.UserExists($account)) {
    $upm.CreateUserProfile($account)
}

$adUser = New-Object System.Security.Principal.NTAccount($account)
$sid = $adUser.Translate([System.Security.Principal.SecurityIdentifier])
Write-Output "Windows SID: $($sid.Value)"
```

Grant this account read access through your normal SharePoint permissions process. The SID must use the `S-1-5-21-...` format.

### HTTP is for isolated labs only

Production guidance requires HTTPS and valid CA trust. If an isolated lab uses HTTP, SharePoint OAuth and metadata over HTTP must be enabled deliberately for that lab. Don't enable these settings as generic `401` troubleshooting in production.

## Step 5: Record the source values

For each new SharePoint source, collect:

| Value | Ingestion API field |
|---|---|
| SharePoint site URL | `sharePointUrl` |
| Server-relative library or folder path starting with `/` (for example, `/Shared Documents`) | `sharePointFolderPath` |
| App principal Client ID | `sharePointClientId` |
| Trusted issuer ID | `sharePointIssuerId` |
| Ingestion-account Windows SID | `sharePointSid` |
| Certificate Secret name | `sharePointCertSecretName` |
| Realm, if you don't use discovery | `sharePointRealm` |
| Separate password Secret, if used | `sharePointCertPasswordSecretName` |
| Password data key, if not `cert-password` | `sharePointCertPasswordSecretKey` |

Different sources can select different identities and Secrets without redeploying the extension.

## Step 6: Confirm indexed sources are enabled

In the Azure portal:

1. Go to **Components**.
1. Set **Enable indexed sources** to **Enabled**.
1. Select **Configure** for **Indexed sources**.
1. Choose the processing mode and save.

There's no deployment-wide SharePoint certificate or Key Vault form. SharePoint identity and signing-Secret selection happens when you add the data source.

## Step 7: Add and validate the SharePoint source

Ensure the target collection exists before you submit an ingestion job. Add the SharePoint source in the developer portal or by using the [Ingestion REST API](reference/ingestion-api-reference.md). For portal steps, see [Add a data source](add-data-source.md).

The API preflight proves only that the selected Secrets and required data keys are readable in the application namespace. It doesn't validate the PFX password, certificate trust, trusted issuer, realm, or SharePoint content permissions.

Complete an ingestion run and read an indexed document before you treat the setup as operationally validated. Monitor the run by using the returned `operation-location`.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster](prepare-aks-cluster.md)

## Related content

- [SharePoint authentication overview](connect-sharepoint-overview.md)
- [SharePoint connection reference and troubleshooting](connect-sharepoint-reference.md)
