---
title: Set Up SharePoint Server S2S Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure certificates, Azure Key Vault, Workload Identity, and SharePoint Server for S2S authentication with Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 05/17/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want to set up SharePoint Server S2S authentication so that I can securely ingest on-premises SharePoint data with Agents and Tools with Foundry Local.
---

# Set up SharePoint Server S2S authentication for Agents and Tools with Foundry Local

This article walks through the step-by-step setup for SharePoint Server High-Trust Server-to-Server (S2S) authentication. Before you begin, review the [prerequisites checklist](connect-sharepoint-overview.md#prerequisites-checklist).

Estimated time: approximately 2 hours.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Step 1: Install cluster prerequisites

Two cluster-level components must be installed before enabling SharePoint in Agents and Tools with Foundry Local.

### CSI Secrets Store Driver

The CSI driver syncs your certificate from Azure Key Vault into the Kubernetes cluster as a Kubernetes secret. Without it, the ingestion pod fails to start with `driver "secrets-store.csi.k8s.io" not found`.

**For AKS:**

```azurecli
az aks enable-addons \
  --resource-group <resource_group> \
  --name <cluster_name> \
  --addons azure-keyvault-secrets-provider \
  --enable-secret-rotation
```

**For Arc-enabled Kubernetes (Azure Local):**

```azurecli
az k8s-extension create \
  --cluster-name <cluster_name> \
  --resource-group <resource_group> \
  --cluster-type connectedClusters \
  --extension-type microsoft.azure.secretstore \
  --name ssarcextension \
  --scope cluster \
  --configuration-settings \
    rotationPollIntervalInSeconds=120 \
    secrets-store-csi-driver.syncSecret.enabled=true
```

### Workload Identity / OIDC Issuer

Workload Identity allows the CSI driver to authenticate to Azure Key Vault by using a federated Kubernetes ServiceAccount ΓÇö no client secrets are stored in the cluster.

**For AKS:**

```azurecli
az aks update \
  --resource-group <resource_group> \
  --name <cluster_name> \
  --enable-oidc-issuer \
  --enable-workload-identity
```

**For Arc-enabled Kubernetes:**

```azurecli
az connectedk8s update \
  --name <cluster_name> \
  --resource-group <resource_group> \
  --enable-oidc-issuer
```

## Step 2: Create the certificate

Run the following PowerShell on any Windows machine or on the SharePoint server:

```powershell
# Create a self-signed certificate (RSA 2048, SHA256, valid 2 years)
$cert = New-SelfSignedCertificate `
    -Subject "CN=EdgeRAG-SharePoint-S2S" `
    -CertStoreLocation "Cert:\LocalMachine\My" `
    -KeyExportPolicy Exportable `
    -KeySpec KeyExchange `
    -KeyLength 2048 `
    -HashAlgorithm SHA256 `
    -NotAfter (Get-Date).AddYears(2) `
    -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider"

# Choose a strong password for the PFX export
$pfxPassword = ConvertTo-SecureString "<your_pfx_password>" -AsPlainText -Force

# Export PFX (private key) ΓÇö this goes to Azure Key Vault
Export-PfxCertificate -Cert $cert -FilePath "C:\edgerag-s2s.pfx" -Password $pfxPassword

# Export CER (public key only) ΓÇö this stays on the SharePoint server
Export-Certificate -Cert $cert -FilePath "C:\edgerag-s2s.cer"

# Note the thumbprint for troubleshooting
Write-Output "Certificate Thumbprint: $($cert.Thumbprint)"
```

After this step you have:

| File | Contains | Destination |
|---|---|---|
| `edgerag-s2s.pfx` | Private key (for signing JWTs) | Azure Key Vault |
| `edgerag-s2s.cer` | Public key (for verification) | SharePoint Server |
| Thumbprint | Identifier for debugging | Your notes |

## Step 3: Upload certificate to Azure Key Vault

Upload the PFX and its password as two separate secrets:

```azurecli
# Upload PFX certificate (base64-encoded binary)
az keyvault secret set \
  --vault-name "<keyvault_name>" \
  --name "edgerag-sp-s2s-cert" \
  --file "edgerag-s2s.pfx" \
  --encoding base64

# Upload PFX password
az keyvault secret set \
  --vault-name "<keyvault_name>" \
  --name "sp-cert-password" \
  --value "<your_pfx_password>"
```

> [!NOTE]
> The secret names must match the values you provide during installation (`KV Cert Secret Name` and `KV Cert Password Secret Name` fields).

| Key Vault secret name | Default in extension | Contains |
|---|---|---|
| `edgerag-sp-s2s-cert` | `edgerag-sp-s2s-cert` | PFX certificate (base64) |
| `sp-cert-password` | `sp-cert-password` | PFX password (plaintext string) |

## Step 4: Set up Workload Identity for Key Vault access

### Create a user-assigned managed identity

```azurecli
az identity create \
  --name "edgerag-kv-identity" \
  --resource-group "<resource_group>" \
  --location "<location>"

# Capture the Client ID ΓÇö you need this for the "Workload Identity Client ID" portal field
MI_CLIENT_ID=$(az identity show \
  --name "edgerag-kv-identity" \
  --resource-group "<resource_group>" \
  --query clientId -o tsv)

echo "Managed Identity Client ID: $MI_CLIENT_ID"
```

### Grant the managed identity access to Key Vault

**Using access policies:**

```azurecli
az keyvault set-policy \
  --name "<keyvault_name>" \
  --secret-permissions get \
  --spn "$MI_CLIENT_ID"
```

**Or using Azure RBAC (if your Key Vault uses RBAC):**

```azurecli
KV_RESOURCE_ID=$(az keyvault show --name "<keyvault_name>" --query id -o tsv)

az role assignment create \
  --role "Key Vault Secrets User" \
  --assignee "$MI_CLIENT_ID" \
  --scope "$KV_RESOURCE_ID"
```

### Create a federated identity credential

This links the Kubernetes ServiceAccount `edgerag-sp-sa` to the managed identity, so the CSI driver can authenticate to Key Vault without storing any client secret in the cluster.

```azurecli
# Get the OIDC issuer URL
# For AKS:
ISSUER_URL=$(az aks show \
  --resource-group "<resource_group>" \
  --name "<cluster_name>" \
  --query "oidcIssuerProfile.issuerUrl" -o tsv)

# For Arc:
# ISSUER_URL=$(az connectedk8s show \
#   --name "<cluster_name>" \
#   --resource-group "<resource_group>" \
#   --query "oidcIssuerProfile.issuerUrl" -o tsv)

# Create the federated credential
az identity federated-credential create \
  --name "edgerag-sp-credential" \
  --identity-name "edgerag-kv-identity" \
  --resource-group "<resource_group>" \
  --issuer "$ISSUER_URL" \
  --subject "system:serviceaccount:arc-rag:edgerag-sp-sa" \
  --audience "api://AzureADTokenExchange"
```

> [!IMPORTANT]
> The `subject` must exactly match the ServiceAccount name (`edgerag-sp-sa`) and namespace (`arc-rag`) that the extension creates. If your namespace is different, adjust accordingly.

## Step 5: Configure SharePoint Server

All commands in this section run on the SharePoint server as a Farm Administrator using PowerShell.

### Register certificate as trusted root authority

```powershell
Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

# Load the public key certificate (.cer file from Step 2)
$cert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("C:\edgerag-s2s.cer")

# Register as trusted root authority
New-SPTrustedRootAuthority -Name "EdgeRAG-S2S" -Certificate $cert
```

### Register as trusted security token issuer

```powershell
# Choose a unique GUID for the Issuer ID ΓÇö SAVE THIS VALUE
$issuerId = [guid]::NewGuid().ToString()

# Get the SharePoint realm
$realm = Get-SPAuthenticationRealm
$fullIssuerId = "$issuerId@$realm"

New-SPTrustedSecurityTokenIssuer `
    -Name "EdgeRAG-S2S" `
    -Certificate $cert `
    -RegisteredIssuerName $fullIssuerId `
    -IsTrustBroker

Write-Output "================================================"
Write-Output "SAVE THESE VALUES:"
Write-Output "  Issuer ID : $issuerId"
Write-Output "  Realm     : $realm"
Write-Output "================================================"
```

### Enable OAuth over HTTP (skip if SharePoint uses HTTPS)

```powershell
# ONLY run this if your SharePoint web application uses HTTP (not HTTPS)
$sts = Get-SPSecurityTokenServiceConfig
$sts.AllowOAuthOverHttp = $true
$sts.AllowMetadataOverHttp = $true
$sts.Update()
```

### Create required service applications

S2S authentication requires three SharePoint service applications. They might already exist if your farm was configured with the wizard. This script checks first and only creates what's missing.

```powershell
Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

# Determine database server from config database
$configDb = Get-SPDatabase | Where-Object { $_.Name -eq "SharePoint_Config" }
$dbServer = $configDb.Server.Address

# Get or create the service application pool
$pool = Get-SPServiceApplicationPool "SharePoint Web Services Default" -ErrorAction SilentlyContinue
if (-not $pool) {
    $managedAccount = Get-SPManagedAccount "<DOMAIN\farm_account>"
    $pool = New-SPServiceApplicationPool `
        -Name "SharePoint Web Services Default" `
        -Account $managedAccount
}

# --- App Management Service Application ---
$appMgmt = Get-SPServiceApplication | Where-Object {
    $_.TypeName -eq "App Management Service Application"
}
if (-not $appMgmt) {
    Get-SPServiceInstance | Where-Object {
        $_.TypeName -eq "App Management Service"
    } | Start-SPServiceInstance
    Start-Sleep 5
    $appMgmt = New-SPAppManagementServiceApplication `
        -Name "App Management Service" `
        -ApplicationPool $pool `
        -DatabaseServer $dbServer `
        -DatabaseName "SP_AppManagement"
    New-SPAppManagementServiceApplicationProxy `
        -ServiceApplication $appMgmt `
        -Name "App Management Proxy"
    Write-Output "Created: App Management Service"
} else {
    Write-Output "Already exists: App Management Service"
}

# --- Subscription Settings Service Application ---
$subSettings = Get-SPServiceApplication | Where-Object {
    $_.TypeName -like "*Subscription*"
}
if (-not $subSettings) {
    Get-SPServiceInstance | Where-Object {
        $_.TypeName -like "*Subscription*"
    } | Start-SPServiceInstance
    Start-Sleep 5
    $subSettings = New-SPSubscriptionSettingsServiceApplication `
        -Name "Subscription Settings" `
        -ApplicationPool $pool `
        -DatabaseServer $dbServer `
        -DatabaseName "SP_SubscriptionSettings"
    New-SPSubscriptionSettingsServiceApplicationProxy `
        -ServiceApplication $subSettings
    Write-Output "Created: Subscription Settings"
} else {
    Write-Output "Already exists: Subscription Settings"
}

# --- User Profile Service Application ---
$upa = Get-SPServiceApplication | Where-Object {
    $_.TypeName -eq "User Profile Service Application"
}
if (-not $upa) {
    Get-SPServiceInstance | Where-Object {
        $_.TypeName -eq "User Profile Service"
    } | Start-SPServiceInstance
    Start-Sleep 5
    $upa = New-SPProfileServiceApplication `
        -Name "User Profile Service" `
        -ApplicationPool $pool `
        -ProfileDBServer $dbServer -ProfileDBName "SP_UPA_Profile" `
        -SocialDBServer $dbServer -SocialDBName "SP_UPA_Social" `
        -ProfileSyncDBServer $dbServer -ProfileSyncDBName "SP_UPA_Sync"
    New-SPProfileServiceApplicationProxy `
        -ServiceApplication $upa `
        -Name "User Profile Proxy" `
        -DefaultProxyGroup
    Write-Output "Created: User Profile Service"
} else {
    Write-Output "Already exists: User Profile Service"
}
```

### Register the app principal

```powershell
# Set the app domain (required for app registration)
Set-SPAppDomain "<your_ad_domain>"
Set-SPAppSiteSubscriptionName -Name "app" -Confirm:$false

# Choose a unique GUID for the Client ID ΓÇö SAVE THIS VALUE
$clientId = [guid]::NewGuid().ToString()

$realm = Get-SPAuthenticationRealm
$appId = "$clientId@$realm"

# Register the app principal on the target SharePoint site
$site = Get-SPSite "<your_sharepoint_site_url>"
$web = $site.RootWeb

$principal = Register-SPAppPrincipal `
    -NameIdentifier $appId `
    -Site $web `
    -DisplayName "Agents and Tools Ingestion Service"

# Grant FullControl at site collection level
Set-SPAppPrincipalPermission `
    -Site $web `
    -AppPrincipal $principal `
    -Scope SiteCollection `
    -Right FullControl

Write-Output "================================================"
Write-Output "SAVE THIS VALUE:"
Write-Output "  Client ID : $clientId"
Write-Output "================================================"
```

> [!NOTE]
> If Agents and Tools with Foundry Local needs to access multiple SharePoint site collections, run `Register-SPAppPrincipal` and `Set-SPAppPrincipalPermission` for each site.

### Create user profile and get Windows SID

```powershell
# The service account the extension impersonates
$account = "<DOMAIN\service_account>"

# Create a user profile if one doesn't exist
$site = Get-SPSite "<your_sharepoint_site_url>"
$context = Get-SPServiceContext $site
$upm = New-Object Microsoft.Office.Server.UserProfiles.UserProfileManager($context)

if (-not $upm.UserExists($account)) {
    $upm.CreateUserProfile($account)
    Write-Output "Created user profile for $account"
} else {
    Write-Output "User profile already exists for $account"
}

# Get the Windows SID
$adUser = New-Object System.Security.Principal.NTAccount($account)
$sid = $adUser.Translate([System.Security.Principal.SecurityIdentifier])

Write-Output "================================================"
Write-Output "SAVE THIS VALUE:"
Write-Output "  Windows SID : $($sid.Value)"
Write-Output "================================================"
```

> [!IMPORTANT]
> The Windows SID must be in `S-1-5-21-...` format. Don't use `DOMAIN\username` or `i:0#.w|domain\user` ΓÇö only the SID format works with S2S JWT authentication.

## Summary: Values collected

After completing all steps, you should have these values ready for installation:

| Value | Example | Source |
|---|---|---|
| **Managed Identity Client ID** | `cccccccc-cccc-cccc-cccc-cccccccccccc` | Step 4: `az identity show` |
| **Key Vault Name** | `contoso-edgerag-kv` | Your Azure Key Vault |
| **KV Cert Secret Name** | `edgerag-sp-s2s-cert` | Step 3: name you chose |
| **KV Cert Password Secret Name** | `sp-cert-password` | Step 3: name you chose |
| **Key Vault Tenant ID** | `dddddddd-dddd-dddd-dddd-dddddddddddd` | Your Microsoft Entra tenant ID |
| **Client ID** | `11111111-1111-1111-1111-111111111111` | Step 5: GUID you chose |
| **Issuer ID** | `22222222-2222-2222-2222-222222222222` | Step 5: GUID you chose |
| **Realm** | `da72f0f6-9536-4cd1-b941-9bb9a73aabce` | Step 5: `Get-SPAuthenticationRealm` |
| **Windows SID** | `S-1-5-21-1352341417-2961234301-8923471-1001` | Step 5: translated from AD account |

## Install Agents and Tools with Foundry Local with SharePoint enabled

### Azure portal (recommended)

1. In the Azure portal, navigate to your cluster.
1. Go to **Extensions** > **Add** > **Agentic RAG**.
1. In the **Data Source Connection / Authentication** section, under **SharePoint S2S**:

   | Portal field | Enter this value |
   |---|---|
   | **Enable SharePoint Ingestion** | Toggle **On** |
   | **Key Vault Name** | Your Key Vault name |
   | **KV Cert Secret Name** | `edgerag-sp-s2s-cert` (pre-filled default) |
   | **KV Cert Password Secret Name** | `sp-cert-password` (pre-filled default) |
   | **Workload Identity Client ID** | Managed identity Client ID (from Step 4) |
   | **Key Vault Tenant ID** | Usually auto-populated from your portal session |

1. Optionally enter the S2S identity parameters (Client ID, Issuer ID, SID, Realm) ΓÇö or configure them per-datasource later in the UI.
1. Complete the rest of the installation wizard.

### Azure CLI

```azurecli
az k8s-extension create \
  --cluster-name "<cluster_name>" \
  --resource-group "<resource_group>" \
  --cluster-type connectedClusters \
  --extension-type microsoft.arc.rag \
  --name <extension_name> \
  --configuration-settings \
    sharepoint.sharepointIngestionEnabled=true \
    sharepoint.s2s.keyvaultName="<keyvault_name>" \
    sharepoint.s2s.kvCertSecretName="edgerag-sp-s2s-cert" \
    sharepoint.s2s.kvCertPasswordSecretName="sp-cert-password" \
    sharepoint.s2s.workloadIdentityClientId="<managed_identity_client_id>" \
    sharepoint.s2s.kvTenantId="<tenant_id>" \
    sharepoint.s2s.clientId="<sharepoint_client_id>" \
    sharepoint.s2s.issuerId="<issuer_id>" \
    sharepoint.s2s.defaultSid="<windows_sid>" \
    sharepoint.s2s.realm="<sharepoint_realm>"
```

> [!NOTE]
> You can update S2S identity values after install without reinstalling:
>
> ```azurecli
> az k8s-extension update \
>   --cluster-name "<cluster_name>" \
>   --resource-group "<resource_group>" \
>   --cluster-type connectedClusters \
>   --name <extension_name> \
>   --configuration-settings \
>     sharepoint.s2s.clientId="<sharepoint_client_id>" \
>     sharepoint.s2s.issuerId="<issuer_id>" \
>     sharepoint.s2s.defaultSid="<windows_sid>"
> ```

## Create a SharePoint data source

1. Open the developer portal.
1. Navigate to **Data Sources** > **Add new data source**.
1. Select **SharePoint Server** as the data source type.
1. Enter:
   - **SharePoint URL**: Your SharePoint web application URL (for example, `http://sharepoint.contoso.com`).
   - **Folder Path**: Server-relative path to the document library (for example, `/sites/docs/Shared Documents`).
1. If you didn't set S2S identity parameters during install, expand **Authentication** and enter:
   - **Client ID**: The GUID from Step 5.
   - **Issuer ID**: The GUID from Step 5.
   - **Windows SID**: The SID from Step 5.
   - **Realm**: Leave blank for auto-discovery, or enter the realm GUID.
1. Select **Connect & Ingest**.

> [!TIP]
> If you provided S2S identity values during installation, they serve as cluster-wide defaults. You can override any of them per-datasource ΓÇö per-datasource values take precedence when non-empty. This allows different data sources to connect to different SharePoint sites with different identities.

## Next step

> [!div class="nextstepaction"]
> [Reference and troubleshooting](connect-sharepoint-reference.md)
