---
title: SharePoint Server-to-Server Reference and Troubleshooting for Agents and Tools with Foundry Local
description: "Verify, troubleshoot, and maintain SharePoint server-to-server authentication for Agents and Tools with Foundry Local."
author: cwatson-cat
ms.author: cwatson
ms.topic: reference
ms.date: 05/17/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a platform administrator, I want to verify, troubleshoot, and maintain SharePoint server-to-server authentication for Agents and Tools with Foundry Local.
---

# SharePoint server-to-server reference and troubleshooting for Agents and Tools with Foundry Local

Use this reference to verify your SharePoint Server-to-Server (S2S) setup, diagnose issues, manage certificates, and review Helm values.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Verification

### Check pod is running with certificate mounted

```bash
# Find the ingestion pod
kubectl get pods -n arc-rag -l app=ingestion-publisher

# Verify cert is mounted
kubectl exec -n arc-rag <pod_name> -- ls -la /certs/
# Should show cert.pfx
```

### Check logs for successful connection

```bash
kubectl logs -n arc-rag <pod_name> | grep -i sharepoint
```

A successful connection shows:

```
SharePoint S2S: Certificate loaded from /certs/cert.pfx
SharePoint S2S: Token created for audience 00000003-0000-0ff1-ce00-000000000000/sharepoint.contoso.com@da72f0f6-...
SharePoint S2S: Connected to http://sharepoint.contoso.com -  200 OK
```

### Verify Container Storage Interface (CSI) secret sync

```bash
# Check the K8s Secret was created by the CSI driver
kubectl get secret sharepoint-s2s-cert -n arc-rag
# Should exist and show data keys: cert.pfx, cert-password
```

## Troubleshooting

| Symptom | Root cause | Fix |
|---|---|---|
| **Pod fails to start:** `driver "secrets-store.csi.k8s.io" not found` | CSI Secrets Store Driver not installed. | Install the driver. See [Step 1](connect-sharepoint-setup.md#step-1-install-cluster-prerequisites). |
| **CSI FailedMount:** `unauthorized_client` | `kvTenantId` doesn't match the managed identity's tenant. | Verify `sharepoint.s2s.kvTenantId` matches the tenant where the managed identity was created. If empty, it defaults to `auth.tenantId`. |
| **CSI FailedMount:** generic auth failure | Federated credential not set up, or subject mismatch. | Rerun [Step 4](connect-sharepoint-setup.md#step-4-set-up-workload-identity-for-key-vault-access). Verify subject is `system:serviceaccount:arc-rag:edgerag-sp-sa`. |
| **CSI FailedMount:** timeout or network error | Key Vault has firewall or private endpoint blocking cluster. | Add cluster outbound IPs to Key Vault firewall, or configure a private endpoint. |
| **Pod CrashLoopBackOff:** `cert.pfx not found at /certs/cert.pfx` | The CSI driver didn't sync the certificate yet, or the Kubernetes secret is missing. | Run `kubectl get secret sharepoint-s2s-cert -n arc-rag`. If missing, check CSI logs: `kubectl logs -n kube-system -l app=secrets-store-csi-driver`. |
| **401 Unauthorized**- response has `WWW-Authenticate: NTLM` only (no `Bearer`) | SharePoint not processing OAuth tokens. | On the SharePoint server, run: `$sts = Get-SPSecurityTokenServiceConfig; $sts.AllowOAuthOverHttp = $true; $sts.AllowMetadataOverHttp = $true; $sts.Update()` |
| **401 Unauthorized**- response has `Bearer` in `WWW-Authenticate` | The JSON Web Token (JWT) `nameid` claim uses the wrong format. | Verify you're providing the Windows Security Identifier (SID) (`S-1-5-21-...`), not `DOMAIN\username`. Rerun [Step 5](connect-sharepoint-setup.md#create-user-profile-and-get-windows-sid). |
| **403 Forbidden** | App principal registered but lacks permissions on target site. | Run `Set-SPAppPrincipalPermission` with `-Right FullControl` for the correct site. |
| **500 Internal Server Error** from SharePoint | Missing service application (App Management or User Profile). | Run the service application creation script from [Step 5](connect-sharepoint-setup.md#create-required-service-applications). Check SharePoint Unified Logging Service (ULS) logs for `UserProfileApplicationNotAvailableException` or `AppManagement` errors. |
| **`App Management Shared Service Proxy is not installed`** | App Management proxy wasn't created. | Run `New-SPAppManagementServiceApplicationProxy`. |
| **`UserProfileNoUserFoundException`** | No user profile for the service account. | Create it using `$upm.CreateUserProfile("<DOMAIN\service_account>")`. |
| **SharePoint datasource shows "not configured"** | Server-to-server identity parameters not provided. | Provide Client ID, Issuer ID, and SID either via `az k8s-extension update` or in the datasource Authentication section. |
| **SSL/TLS error** connecting to SharePoint | SharePoint uses internal CA not trusted by pod. | Set `SHAREPOINT_VERIFY_SSL` to your CA bundle path. Don't set to `false` in production. |

## Network requirements

| Source | Destination | Port | Protocol | Purpose |
|---|---|---|---|---|
| Cluster pods | SharePoint Server | 80 or 443 | HTTP/HTTPS | SharePoint API access |
| Cluster pods | DNS Server | 53 | TCP/UDP | Hostname resolution |
| Cluster pods | Azure Key Vault | 443 | HTTPS | Certificate sync (initial + rotation) |

> [!NOTE]
> No Active Directory Federation Services (ADFS), no ACS, and no internet is required for the authentication flow itself. Once the certificate is synced to the cluster as a Kubernetes secret, SharePoint authentication works fully offline and disconnected.

## Certificate rotation

Certificates should be rotated before expiry. The extension supports zero-downtime rotation:

1. **Create a new certificate** (repeat [Step 2](connect-sharepoint-setup.md#step-2-create-the-certificate)).

1. **Register the new public key on SharePoint** as a second trusted issuer. Both old and new certificates are trusted simultaneously:

   ```powershell
   $newCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2("C:\edgerag-s2s-new.cer")
   New-SPTrustedRootAuthority -Name "EdgeRAG-S2S-New" -Certificate $newCert
   ```

1. **Upload the new PFX file to Key Vault**, overwriting the existing secret:

   ```azurecli
   az keyvault secret set \
     --vault-name "<keyvault_name>" \
     --name "edgerag-sp-s2s-cert" \
     --file "edgerag-s2s-new.pfx" \
     --encoding base64
   ```

1. **CSI driver picks up the new certificate** on its next poll (default: every 2 minutes) and updates the Kubernetes secret.

1. **New pods use the new certificate** automatically. Existing pods pick it up on next volume refresh.

1. **Remove the old trusted issuer** from SharePoint after confirming the new certificate works.

## Security considerations

### No passwords in Helm values

The Personal Information Exchange (PFX) password is never stored in Helm values or ConfigMaps. The delivery chain is: Azure Key Vault secret  →  CSI driver  → Kubernetes secret  →  pod environment variable. The password is not exposed in cluster configuration at any point.

### Transport Layer Security (TLS) verification

By default, the extension verifies SharePoint's TLS certificate (`SHAREPOINT_VERIFY_SSL=true`). If your SharePoint uses an internal CA:

```azurecli
# Point to your CA bundle
az k8s-extension update \
  --configuration-settings sharepoint.s2s.verifySsl="/etc/ssl/certs/internal-ca.crt"
```

> [!IMPORTANT]
> Setting `SHAREPOINT_VERIFY_SSL=false` exposes the connection to man-in-the-middle attacks. Only use for nonproduction debugging.

### Per-datasource isolation

Different SharePoint data sources can use different client identifiers, issuer identifiers, and Windows SIDs. Per-datasource isolation allows you to:

- Scope each data source to a specific site collection with minimum required permissions.
- Use different service accounts with different access levels.
- Audit access per data source in SharePoint logs.

## Helm values reference

All SharePoint-related Helm values live under the `sharepoint` key.

### Install-time values (require Helm upgrade and pod restart to change)

| Helm key | Type | Default | Required | Description |
|---|---|---|---|---|
| `sharepoint.sharepointIngestionEnabled` | bool | `false` | Yes | Master toggle- gates all SharePoint config. |
| `sharepoint.s2s.keyvaultName` | string | `""` | Yes | Azure Key Vault name. |
| `sharepoint.s2s.kvCertSecretName` | string | `"edgerag-sp-s2s-cert"` | Yes | Key Vault secret name for PFX cert. |
| `sharepoint.s2s.kvCertPasswordSecretName` | string | `"sp-cert-password"` | Yes | Key Vault secret name for PFX password. |
| `sharepoint.s2s.workloadIdentityClientId` | string | `""` | Yes | Managed identity client ID with Key Vault access. |
| `sharepoint.s2s.kvTenantId` | string | `""` | No | Microsoft Entra tenant ID for managed identity. Falls back to `auth.tenantId`. |
| `sharepoint.s2s.certSecretName` | string | `"sharepoint-s2s-cert"` | No | Kubernetes secret name created by CSI (rarely changed). |
| `sharepoint.s2s.certPath` | string | `"/certs/cert.pfx"` | No | Mount path in pod (rarely changed). |
| `sharepoint.s2s.certPasswordSecretName` | string | `""` | No | Kubernetes secret for password. Defaults to `certSecretName` (single-secret). |
| `sharepoint.s2s.certPasswordSecretKey` | string | `"cert-password"` | No | Key within the Kubernetes secret for password. |

### Post-install values (updatable via `az k8s-extension update`, overridable per-datasource)

| Helm key | Type | Default | Description |
|---|---|---|---|
| `sharepoint.s2s.clientId` | GUID | `""` | SharePoint app Client ID. |
| `sharepoint.s2s.issuerId` | GUID | `""` | SharePoint token Issuer ID. |
| `sharepoint.s2s.defaultSid` | string | `""` | Windows SID of service account (`S-1-5-21-...`). |
| `sharepoint.s2s.realm` | GUID | `""` | SharePoint auth realm (autodiscovered if empty). |

## Related content

- [SharePoint server-to-server authentication overview](connect-sharepoint-overview.md)
- [Set up server-to-server authentication](connect-sharepoint-setup.md)
