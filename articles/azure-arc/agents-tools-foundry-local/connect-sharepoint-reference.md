---
title: SharePoint server-to-server reference and troubleshooting
description: Verify, troubleshoot, and maintain SharePoint server-to-server authentication for Agents and Tools with Foundry Local.
ms.reviewer: assafakiva
ms.topic: reference
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a platform administrator, I want to verify, troubleshoot, and maintain SharePoint server-to-server authentication for Agents and Tools with Foundry Local.
---

# SharePoint server-to-server reference and troubleshooting

Use this reference to verify and maintain certificate-based authentication for indexed SharePoint ingestion. The public setup uses customer-managed Kubernetes Secrets selected by each source. Older deployments can contain legacy Key Vault or CSI configuration for compatibility; don't use that path for new customer setup.

## Verification

Set the application namespace and Secret name:

```bash
NAMESPACE="arc-rag"
CERT_SECRET="sharepoint-s2s-cert"
```

### Verify the Secret metadata

Check key names without printing or decoding credential values:

```bash
kubectl describe secret "$CERT_SECRET" --namespace "$NAMESPACE"
```

Confirm that:

- The Secret is in the extension's application namespace.
- The `cert.pfx` data key exists and has a nonzero size.
- The selected password data key exists, even when the PFX password is empty.

Secret preflight proves only that the resource and keys are readable. It doesn't validate the PFX password, private key, farm trust, issuer, realm, TLS trust, or SharePoint permissions.

### Verify an ingestion run

Create a SharePoint source and start an ingestion run. Use the returned `operation-location` to monitor it. Treat the setup as operationally validated only after the run succeeds and an indexed document can be read.

For deeper diagnostics, use DEBUG logging temporarily. Current DEBUG messages include:

- `SharePoint S2S connection initialized`
- `Created S2S access token (...)`
- `SharePoint S2S connection test successful. Site title: ...`

These messages aren't emitted at the normal information log level. Don't log or share certificate data, tokens, document content, or user information.

## Troubleshooting

| Symptom | Likely cause | Action |
|---|---|---|
| Secret preflight reports that the Secret wasn't found | The source references a missing Secret or the wrong application namespace. | Create the Secret in the extension namespace or correct the source reference. |
| Secret preflight reports a missing or empty data key | `cert.pfx` is absent or empty, or the selected password key is absent. | Recreate or patch the Secret with the required key names. An empty password is valid only when its key exists. |
| `401 Unauthorized` | Wrong Windows-claims zone, issuer, realm, token audience, SID, or HTTPS trust. | Verify the SharePoint URL, trusted issuer registration, source Issuer ID, realm, and ingestion-account SID. Use HTTPS in production. |
| `403 Forbidden` | The app principal or ingestion account lacks access to the selected content. | Grant both identities read access at the required SharePoint scope. A User Profile alone doesn't grant content access. |
| `500 Internal Server Error`, `App Management Shared Service Proxy is not installed`, or `UserProfileApplicationNotAvailableException` | A required App Management, Subscription Settings, or User Profile service application or proxy is missing or isn't associated with the target web application. | Verify the service applications and proxies as described in [Verify required service applications](connect-sharepoint-setup.md#verify-required-service-applications). |
| `UserProfileNoUserFoundException` | The selected ingestion account doesn't have a SharePoint User Profile. | Create or synchronize the profile, then verify its Windows SID and content permissions. |
| PFX or signing failure | The password doesn't open the PFX, the private key is absent, or the certificate doesn't match the trusted issuer. | Validate the PFX in a protected administration environment, then correct the Secret and SharePoint trust. |
| TLS verification failure | The SharePoint HTTPS certificate chains to a CA that the ingestion worker doesn't trust. | Use a certificate chain already trusted by the ingestion worker image. Don't disable certificate verification. |

The current release doesn't expose a supported extension setting that mounts a customer CA bundle into every SharePoint ingestion worker. If the required CA isn't already trusted, don't use an undocumented setting; use an approved trusted certificate or obtain product support for the target release.

Don't enable `AllowOAuthOverHttp` or `AllowMetadataOverHttp` as generic `401` troubleshooting. Those settings are limited to isolated HTTP labs. Production guidance requires the correct Windows-claims/OAuth zone, HTTPS, and valid CA trust.

## Network requirements

| Source | Destination | Port | Purpose |
|---|---|---|---|
| Ingestion workers | SharePoint Server Windows-claims zone | 443 TCP | HTTPS REST enumeration and document download |
| Ingestion workers | DNS servers | 53 TCP/UDP | Name resolution |

Authentication doesn't require an external token service. The worker signs the S2S token locally by using the certificate selected for the source.

## Certificate rotation

Rotate before the current certificate expires. Existing workers load the PFX and password when they start; they don't automatically switch when a Secret volume refreshes.

1. Create a new certificate and export its PFX and public certificate.
1. Register the new public certificate and a new trusted issuer in SharePoint while the old trust remains valid:

   ```powershell
   Add-PSSnapin Microsoft.SharePoint.PowerShell -ErrorAction SilentlyContinue

   $newCert = New-Object System.Security.Cryptography.X509Certificates.X509Certificate2(
       "C:\sharepoint-s2s-new.cer"
   )
   New-SPTrustedRootAuthority `
       -Name "FoundryLocal-SharePoint-S2S-New" `
       -Certificate $newCert

   $newIssuerId = [guid]::NewGuid().ToString()
   $realm = Get-SPAuthenticationRealm

   New-SPTrustedSecurityTokenIssuer `
       -Name "FoundryLocal-SharePoint-S2S-New" `
       -Certificate $newCert `
       -RegisteredIssuerName "$newIssuerId@$realm" `
       -IsTrustBroker

   Write-Output "New Issuer ID: $newIssuerId"
   ```

1. Create a new Kubernetes Secret with the new PFX and password.
1. Create or update each affected source or refresh configuration to use the new Secret and the new Issuer ID. Selecting a new Secret for another source doesn't rewrite existing saved configuration.
1. Start new workers or new ingestion runs with the updated configuration. Keep the old certificate, Secret, and SharePoint trust valid while earlier workers finish.
1. Validate a complete indexed ingestion and read.
1. Drain or recreate any workers that still use the old credential. Remove the old issuer and certificate trust only after no active worker depends on them.

This is a controlled rotation, not an atomic or zero-downtime Secret refresh.

## Security considerations

- Keep the PFX and password in Kubernetes Secrets. Don't put them in Helm values, ConfigMaps, source control, logs, or support bundles.
- Give Secret creation and rotation rights only to the designated cluster administrator.
- The external ingestion caller needs an Azure role that allows ingestion, such as **Cognitive Services Contributor** at the extension scope, or **Contributor** in disconnected deployments. The `EdgeRAGDeveloper` app role also works as a fallback. The caller doesn't need direct Kubernetes Secret-read permission. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- Scope the app principal and ingestion account to the SharePoint content they need.
- Use HTTPS and certificate validation in production.

## Related content

- [SharePoint server-to-server authentication overview](connect-sharepoint-overview.md)
- [Set up server-to-server authentication](connect-sharepoint-setup.md)
