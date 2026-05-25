---
title: Set Up SharePoint Server-to-Server Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure certificates, Azure Key Vault, Workload Identity, and SharePoint Server for server-to-server authentication with Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 05/25/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#customer intent: As a platform administrator, I want to set up SharePoint server-to-server authentication so that I can securely ingest on-premises SharePoint data with Agents and Tools with Foundry Local.
---

# Set up SharePoint server-to-server authentication for Agents and Tools with Foundry Local

This article provides a step-by-step guide for setting up SharePoint Server High-Trust Server-to-Server (S2S) authentication.

## Prerequisites

Before you begin, make sure you have:

- Access to a Windows machine or SharePoint server for running PowerShell in Steps 2 and 5.
- An Azure Key Vault, an Arc-enabled Kubernetes or AKS cluster, and permissions to create or manage a user-assigned managed identity.
- A reachable SharePoint Server Subscription Edition environment where you can run SharePoint Management Shell as a Farm Administrator.

Estimated time: approximately 2 hours.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Step 1: Install cluster prerequisites

You must install two cluster-level components before enabling SharePoint in Agents and Tools with Foundry Local.

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

### Workload Identity and OIDC issuer

By using Workload Identity, the CSI driver authenticates to Azure Key Vault by using a federated Kubernetes ServiceAccount. The cluster doesn't store any client secrets.

You use the OIDC issuer URL from this section in Step 4. Keep it available for the federated credential command.

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

### Verify network connectivity

Before you continue, verify that cluster network access meets SharePoint and Key Vault requirements.

Run these checks from a shell that can access your cluster with `kubectl`:

```bash
# Set target hosts
SHAREPOINT_HOST="<sharepoint_fqdn>"
KEYVAULT_HOST="<keyvault_name>.vault.azure.net"

# Start a temporary pod for network tests
kubectl run netcheck --image=busybox:1.36 --restart=Never -- sleep 300
kubectl wait --for=condition=Ready pod/netcheck --timeout=90s

# 1) DNS resolution for SharePoint
kubectl exec netcheck -- nslookup "$SHAREPOINT_HOST"

# 2) Reach SharePoint on 80 or 443 (use the one your environment uses)
kubectl exec netcheck -- nc -zvw5 "$SHAREPOINT_HOST" 80
kubectl exec netcheck -- nc -zvw5 "$SHAREPOINT_HOST" 443

# 3) Reach Azure Key Vault on 443
kubectl exec netcheck -- nc -zvw5 "$KEYVAULT_HOST" 443

# Clean up
kubectl delete pod netcheck --ignore-not-found
```

If any check fails, resolve DNS, firewall, proxy, or routing issues before you continue. These checks validate that pods can resolve and connect to SharePoint and Azure Key Vault.

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

# Export PFX (private key) -  this goes to Azure Key Vault
Export-PfxCertificate -Cert $cert -FilePath "C:\edgerag-s2s.pfx" -Password $pfxPassword

# Export CER (public key only) -  this stays on the SharePoint server
Export-Certificate -Cert $cert -FilePath "C:\edgerag-s2s.cer"

# Note the thumbprint for troubleshooting
Write-Output "Certificate Thumbprint: $($cert.Thumbprint)"
```

After this step, you have:

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

The secret names must match the values you provide during installation (`KV Cert Secret Name` and `KV Cert Password Secret Name` fields).

| Key Vault secret name | Default in extension | Contains |
|---|---|---|
| `edgerag-sp-s2s-cert` | `edgerag-sp-s2s-cert` | PFX certificate (base64) |
| `sp-cert-password` | `sp-cert-password` | PFX password (plaintext string) |

## Step 4: Set up Workload Identity for Key Vault access

In this step, you create and connect a managed identity so the cluster can securely read certificate secrets from Azure Key Vault without storing client secrets.

### Create a user-assigned managed identity

```azurecli
az identity create \
  --name "edgerag-kv-identity" \
  --resource-group "<resource_group>" \
  --location "<location>"

# Capture the Client ID -  you need this for the "Workload Identity Client ID" portal field
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

This step links a Kubernetes ServiceAccount to the managed identity, so the CSI driver can authenticate to Key Vault without storing any client secret in the cluster.

The ServiceAccount name depends on your configuration:

| Configuration | ServiceAccount | Subject for federated credential |
|---|---|---|
| SharePoint S2S only | `edgerag-sp-sa` | `system:serviceaccount:arc-rag:edgerag-sp-sa` |
| SharePoint S2S **+ Kerberos** | `kerberos-ingestion-sa` | `system:serviceaccount:arc-rag:kerberos-ingestion-sa` |

When Kerberos is enabled (`kerberos.enabled=true`), the ingestion pod uses `kerberos-ingestion-sa` instead of `edgerag-sp-sa` because it needs additional cluster-scoped permissions for persistent volume and node access. The SharePoint workload identity annotations are applied to whichever ServiceAccount the pod uses.

The `--subject` value must exactly match the extension's namespace and ServiceAccount in this format: `system:serviceaccount:<namespace>:<serviceaccount_name>`. If your deployment uses a different namespace, update the subject value to match it exactly. If this value doesn't match, the workload identity exchange fails and the cluster can't read Key Vault secrets.

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
# Use "edgerag-sp-sa" if SharePoint only, "kerberos-ingestion-sa" if Kerberos is also enabled
SA_NAME="edgerag-sp-sa"  # Change to "kerberos-ingestion-sa" if Kerberos is enabled

az identity federated-credential create \
  --name "edgerag-sp-credential" \
  --identity-name "edgerag-kv-identity" \
  --resource-group "<resource_group>" \
  --issuer "$ISSUER_URL" \
  --subject "system:serviceaccount:arc-rag:${SA_NAME}" \
  --audience "api://AzureADTokenExchange"
```

> [!TIP]
> If you plan to toggle Kerberos on or off, create federated credentials for **both** ServiceAccounts to avoid CSI mount failures when switching configurations.

If you're unsure which ServiceAccount is in use after installation, check with:

```bash
kubectl get pods -n arc-rag -l app=ingestion-publisher -o jsonpath='{.items[0].spec.serviceAccountName}'
```

## Step 5: Configure SharePoint Server

Run all commands in this section on the SharePoint server by using SharePoint Management Shell as a Farm Administrator. If you run this remotely, connect to the SharePoint server first and run the commands in that session.

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
# Choose a unique GUID for the Issuer ID -  SAVE THIS VALUE
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

Only use OAuth over HTTP for isolated lab environments that don't use HTTPS. For production, use HTTPS and skip this step.

```powershell
# ONLY run this if your SharePoint web application uses HTTP (not HTTPS)
$sts = Get-SPSecurityTokenServiceConfig
$sts.AllowOAuthOverHttp = $true
$sts.AllowMetadataOverHttp = $true
$sts.Update()
```

### Create required service applications

Server-to-server authentication requires three SharePoint service applications. They might already exist if your farm was configured with the wizard. This script checks first and only creates what's missing.

This script is idempotent. `Created:` and `Already exists:` outputs are both expected outcomes.

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

# Choose a unique GUID for the Client ID -  SAVE THIS VALUE
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


If Agents and Tools with Foundry Local needs to access multiple SharePoint site collections, run `Register-SPAppPrincipal` and `Set-SPAppPrincipalPermission` for each site.

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

The Windows SID must be in `S-1-5-21-...` format. Don't use `DOMAIN\username` or `i:0#.w|domain\user`. Only the SID format works with server-to-server JWT authentication.

## Summary: Values collected

After completing all steps, you should have these values ready for installation:

| Value | Example | Source |
|---|---|---|
| **Managed Identity Client ID** | `00001111-aaaa-2222-bbbb-3333cccc4444` | Step 4: `az identity show` |
| **Key Vault Name** | `contoso-edgerag-kv` | Your Azure Key Vault |
| **KV Cert Secret Name** | `contoso-edgerag-sp-s2s-cert` | Step 3: name you chose |
| **KV Cert Password Secret Name** | `contoso-sp-cert-password` | Step 3: name you chose |
| **Key Vault Tenant ID** | `aaaabbbb-0000-cccc-1111-dddd2222eeee` | Your Microsoft Entra tenant ID |
| **Client ID** | `11112222-3333-4444-aaaa-bbbbccccdddd` | Step 5: GUID you chose |
| **Issuer ID** | `22223333-4444-5555-aaaa-bbbbccccdddd` | Step 5: GUID you chose |
| **Realm** | `33334444-5555-6666-aaaa-bbbbccccdddd` | Step 5: `Get-SPAuthenticationRealm` |
| **Windows SID** | `S-1-5-21-1234567890-2345678901-3456789012-1234` | Step 5: translated from AD account |

## Deploy the extension with SharePoint ingestion enabled

After you complete this SharePoint setup, return to the [deployment prerequisites checklist](complete-prerequisites.md) and finish any remaining prerequisite steps. When you're ready to deploy, use [Deploy the extension for Agents and Tools with Foundry Local](deploy.md) and the SharePoint-specific values in this section.

During deployment:

- In the Azure portal, on the **Configurations** tab, turn on **SharePoint ingestion** and enter the Key Vault and workload identity values collected in this article.
- In Azure CLI, set `enableSharePoint="true"` and provide `keyVaultName`, `kvCertSecretName`, `kvCertPasswordSecretName`, and `workloadIdentityClientId` in the CLI script in [Deploy the extension for Agents and Tools with Foundry Local](deploy.md?tabs=azure-cli).

If you don't plan to use SharePoint ingestion, skip this article and continue with the standard deployment flow in [Deploy the extension for Agents and Tools with Foundry Local](deploy.md).

## Add a SharePoint data source

After you deploy Agents and Tools with Foundry Local and can access the developer portal, add your SharePoint Server data source.

For step-by-step instructions, see [Add a data source for Agents and Tools with Foundry Local](add-data-source.md).

When you add a SharePoint Server data source, you might need values collected in this article:

- **Client ID**
- **Issuer ID**
- **Windows SID**
- **Realm**

For details on how installation defaults and settings you enter for each data source work together, see [Configuration precedence](connect-sharepoint-overview.md#configuration-precedence).

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster](prepare-aks-cluster.md)

## Related content

- [SharePoint Server-to-Server connection reference and troubleshooting](connect-sharepoint-reference.md)
