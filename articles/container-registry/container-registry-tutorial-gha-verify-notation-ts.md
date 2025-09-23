---
title: Verify container images in GitHub workflows with Notation and Trusted Signing
description: Learn how to verify container images with Notation and Trusted Signing using a GitHub Actions workflow.
ms.topic: how-to
author: yizha1
ms.author: yizha1
ms.date: 09/08/2025
ms.service: security
# Customer intent: As a developer, I want to verify container image signatures in GitHub Actions with Trusted Signing, so I can ensure authenticity and integrity before using them in builds or deployments.
---

# Verify container images in GitHub workflows with Notation and Trusted Signing

This article is part of a series on **ensuring integrity and authenticity of container images and OCI artifacts**.  
For the complete picture, start with the [overview](overview-sign-verify-oci-artifacts.md), which explains why signing matters and outlines the various scenarios.

You can use this guide in two scenarios:

- **Consuming signed images**: Verify container images that have already been signed by other teams or organizations using Notation and Trusted Signing.  
- **Verifying your own images**: If you publish images yourself, first sign them using a [GitHub workflow](container-registry-tutorial-gha-sign-notation-ts.md) or the [Notation CLI](container-registry-tutorial-sign-verify-notation-ts.md), then follow this guide to verify the signatures.  

In this article, you'll learn how to:

1. Configure a GitHub workflow.  
2. Verify container image signed with Trusted signing using Notation GitHub actions

## Prerequisites

- An [Azure Container Registry](/azure/container-registry/container-registry-get-started-azure-cli) containing signed images.
- A GitHub repository to store your workflow file, trust policy, and trust store.

## Authenticate from Azure to GitHub

According to [Use GitHub Actions to connect to Azure](/azure/developer/github/connect-from-azure), you must first authenticate with Azure in your workflow by using the **Azure Login** action before running Azure CLI or Azure PowerShell commands. The Azure Login action supports multiple authentication methods.  

In this guide, we'll sign in with **OpenID Connect (OIDC)** using a user-assigned managed identity, following the steps in [Use the Azure login action with OpenID Connect](/azure/developer/github/connect-from-azure-openid-connect).


1. Create a user-assigned managed identity:

   Skip this step if you already have an existing managed identity.

# [Linux](#tab/linux)

```bash
az login
az identity create -g <identity-resource-group> -n <identity-name>
```

# [Windows](#tab/windows)

```powershell
az login
az identity create -g <identity-resource-group> -n <identity-name>
```

---

2. Get the client ID of your managed identity

# [Linux](#tab/linux)

```bash
CLIENT_ID=$(az identity show -g <identity-resource-group> -n <identity-name> --query clientId -o tsv)
```

# [Windows](#tab/windows)

```powershell
$CLIENT_ID = az identity show -g <identity-resource-group> -n <identity-name> --query clientId -o tsv
```

---

3. Assign roles to the managed identity for accessing ACR

    For **non-ABAC** registries, assign `AcrPull` roles:

# [Linux](#tab/linux)

```bash
ACR_SCOPE=/subscriptions/<subscription-id>/resourceGroups/<acr-resource-group>
az role assignment create --assignee $CLIENT_ID --scope $ACR_SCOPE --role "acrpush" --role "acrpull"
```

# [Windows](#tab/windows)

```powershell
$ACR_SCOPE = "/subscriptions/<subscription-id>/resourceGroups/<acr-resource-group>"
az role assignment create --assignee $CLIENT_ID --scope $ACR_SCOPE --role "acrpush" --role "acrpull"
```

---

For **ABAC-enabled** registries, assign the `Container Registry Repository Reader` roles:

# [Linux](#tab/linux)

```bash
ACR_SCOPE=/subscriptions/<subscription-id>/resourceGroups/<acr-resource-group>
az role assignment create --assignee $CLIENT_ID --scope $ACR_SCOPE --role "Container Registry Repository Reader" --role "Container Registry Repository Writer"
```

# [Windows](#tab/windows)

```powershell
$ACR_SCOPE = "/subscriptions/<subscription-id>/resourceGroups/<acr-resource-group>"
az role assignment create --assignee $CLIENT_ID --scope $ACR_SCOPE --role "Container Registry Repository Reader" --role "Container Registry Repository Writer"
```

---

5. Configure GitHub to trust your identity

   Follow [Configure an app to trust an external identity provider](/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity) to allow GitHub Actions to exchange OIDC tokens for this identity.

6. Create GitHub Secrets

    Follow [creating secrets for a repository](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets#creating-secrets-for-a-repository) to set up GitHub secrets, and map the managed identity values to those secrets:

    | GitHub secret	          | Managed identity values |
    | ----------------------- | ----------------------- |
    | `AZURE_CLIENT_ID`	      | Client ID               |
    | `AZURE_SUBSCRIPTION_ID` | Subscription ID         |
    | `AZURE_TENANT_ID`	      | Directory (tenant) ID   |

## Prepare trust policy and trust store

Verification requires a [Notary Project trust policy and trust store](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/trust-store-trust-policy.md) checked into your repository.

### Create the trust store

The **Trust store** (`.github/truststore/`) contains the CA certificates and TSA root certificates required for validation.

Download the **[Trusted Signing root certificate](https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt)** and store it under the `ca` directory.

# [Linux](#tab/linux)

```bash
curl -o .github/truststore/x509/ca/mycerts/msft-identity-verification-root-cert-2020.crt \
    "https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt"
```

# [Windows](#tab/windows)

```powershell
# Create directory if it doesn't exist
New-Item -ItemType Directory -Force -Path ".github\truststore\x509\ca\mycerts"

# Download the certificate
Invoke-WebRequest -Uri "https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt" `
    -OutFile ".github\truststore\x509\ca\mycerts\msft-identity-verification-root-cert-2020.crt"
```

---

Download the **[Trusted Signing TSA root certificate](https://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt)** and store it under the `tsa` directory.

# [Linux](#tab/linux)

```bash
curl -o .github/truststore/x509/tsa/mytsacerts/msft-identity-verification-tsa-root-cert-2020.crt \
  "http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt"
```

# [Windows](#tab/windows)

```powershell
# Create directory if it doesn't exist
New-Item -ItemType Directory -Force -Path ".github\truststore\x509\tsa\mytsacerts"

# Download the certificate
Invoke-WebRequest -Uri "http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt" `
    -OutFile ".github\truststore\x509\tsa\mytsacerts\msft-identity-verification-tsa-root-cert-2020.crt"
```

---


### Create the trust policy

The **Trust policy** (`.github/trustpolicy/trustpolicy.json`) defines which identities and CAs are trusted.

Example `trustpolicy.json` (replace repository URIs, trust store names, and Trusted Signing certificate profile subject with your values):

```json
{
 "version": "1.0",
 "trustPolicies": [
    {
         "name": "mypolicy",
         "registryScopes": [ "myregistry.azurecr.io/myrepo1","myregistry.azurecr.io/myrepo2" ],
         "signatureVerification": {
             "level" : "strict"
         },
         "trustStores": [ "ca:mycerts", "tsa:mytsacerts" ],
         "trustedIdentities": [
	         "x509.subject: C=US, ST=WA, L=Seattle, O=MyCompany.io, OU=Tools"
         ]
     }
 ]
}
```

### Confirm the directory structure

Your repository should look like this:

```text
.github/
├── trustpolicy/
│   └── trustpolicy.json
└── truststore/
    └── x509/
        ├── ca/
        │   └── mycerts/
        │       └── msft-identity-verification-root-cert-2020.crt
        └── tsa/
            └── mytsacerts/
                └── msft-identity-verification-tsa-root-cert-2020.crt
```

## Create the GitHub Actions workflow

Once authentication and trust configuration are ready, create the workflow.

1. Create a `.github/workflows` directory in your repo if it doesn’t exist.
2. Create a new workflow file, for example `verify-with-trusted-signing.yml`.
3. Copy the following workflow template into your file

<details>
<summary>Click here to see the verification workflow template.</summary>

```yaml
name: notation-verify-with-trusted-signing

on:
  push:

env:
  ACR_LOGIN_SERVER: <registry-name>.azurecr.io                      # example: myRegistry.azurecr.io
  ACR_REPO_NAME: <repository-name>                                  # example: myRepo
  IMAGE_TAG: <image-tag>                                            # example: v1
  #IMAGE_DIGEST: <image-digest>                                     # example: sha256:xxx
jobs:
  notation-verify:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      # Log in to Azure with your service principal secret
      # - name: Azure login
      #  uses: Azure/login@v1
      #  with:
      #    creds: ${{ secrets.AZURE_CREDENTIALS }}
      # If you are using OIDC and federated credential, make sure to replace the above step with below:
      - name: Azure login
        uses: Azure/login@v2
        with:
          client-id: ${{ secrets.AZURE_CLIENT_ID }}
          tenant-id: ${{ secrets.AZURE_TENANT_ID }}
          subscription-id: ${{ secrets.AZURE_SUBSCRIPTION_ID }}

      # Log in to your ACR registry
      - name: ACR login
        run: |
            az acr login --name ${{ env.ACR_LOGIN_SERVER }}

      # Set up Notation CLI
      - name: setup notation
        uses: notaryproject/notation-action/setup@v1.2.2
      
      # Verify the OCI artifact, such as container images
      - name: verify OCI artifact
        uses: notaryproject/notation-action/verify@v1
        with:
          target_artifact_reference: ${{ env.ACR_LOGIN_SERVER }}/${{ env.ACR_REPO_NAME }}:${{ env.IMAGE_TAG }}
          # Alternatively, use image digest
          # target_artifact_reference: ${{ env.ACR_LOGIN_SERVER }}/${{ env.ACR_REPO_NAME }}@${{ env.IMAGE_DIGEST }}
          trust_policy: .github/trustpolicy/trustpolicy.json
          trust_store: .github/truststore
```
</details>

4. Update the environment variables with your own registry, repository, and image tag/digest. Save and commit the file.

## Trigger the GitHub Actions workflow

The sample workflow is triggered on push. Commit the workflow file to your repository to start the job.

On success, you can view the workflow logs to confirm the trust policy is imported successfully, certificates from the trust store are loaded, and successful signature verification.