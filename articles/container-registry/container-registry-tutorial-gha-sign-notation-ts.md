---
title: Sign container images in GitHub workflows with Notation and Trusted Signing
description: Learn how to use GitHub Actions with Notation and Trusted Signing to build, push, and sign container images in Azure Container Registry.
ms.topic: how-to
author: yizha1
ms.author: yizha1
ms.date: 09/07/2025
ms.service: security
# Customer intent: As a developer, I want to sign container images in GitHub Actions with Trusted Signing, so I can ensure their authenticity and integrity in CI/CD pipelines and during deployment.
---

# Sign container images in GitHub workflows with Notation and Trusted Signing

This article is part of a series on **ensuring integrity and authenticity of container images and OCI artifacts**.  
For the complete picture, start with the [overview](overview-sign-verify-oci-artifacts.md), which explains why signing matters and outlines the various scenarios.

In this article, you'll learn how to create a GitHub Actions workflow to:

1. Build an image and push it to Azure Container Registry (ACR).  
2. Sign the image with Notation GitHub actions and Trusted Signing
3. Automatically store the generated signature in ACR.  

## Prerequisites

> [!NOTE]
> At this time Trusted Signing is only available to organizations based in the USA and Canada that have a verifiable history of three years or more.

- Set up [Trusted Signing account and certificate profile](/azure/trusted-signing/quickstart).  
- Create or use an [Azure Container Registry](/azure/container-registry/container-registry-get-started-azure-cli).  
- Create or use a GitHub repository to store your workflow file and GitHub Secrets.

> [!IMPORTANT]  
> Your Trusted Signing certificate profile must include the following attributes in its subject:  
> - **Country (C)**  
> - **State or province (ST or S)**  
> - **Organization (O)**  
>  
> These fields are required by the [Notary Project specification](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/trust-store-trust-policy.md#trusted-identities-constraints).

## Authenticate from Azure to GitHub

According to [Use GitHub Actions to connect to Azure](/azure/developer/github/connect-from-azure), you must first authenticate with Azure in your workflow by using the **Azure Login** action before running Azure CLI or Azure PowerShell commands. The Azure Login action supports multiple authentication methods.  

In this guide, you sign in with **OpenID Connect (OIDC)**, use a user-assigned managed identity, and follow the steps in [Use the Azure login action with OpenID Connect](/azure/developer/github/connect-from-azure-openid-connect).


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

    For **non-ABAC** registries, assign the `AcrPush` and `AcrPull` roles:

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

    For **ABAC-enabled** registries, assign the `Container Registry Repository Reader` and `Container Registry Repository Writer` roles:

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

4. Assign roles to the managed identity for accessing Trusted Signing
    
    Assign the `Trusted Signing Certificate Profile Signer` role:

# [Linux](#tab/linux)

```bash
TS_SCOPE=/subscriptions/<subscription-id>/resourceGroups/<ts-account-resource-group>/providers/Microsoft.CodeSigning/codeSigningAccounts/<ts-account>/certificateProfiles/<ts-cert-profile>
az role assignment create --assignee $CLIENT_ID --scope $TS_SCOPE --role "Trusted Signing Certificate Profile Signer"
```

# [Windows](#tab/windows)

```powershell
$TS_SCOPE = "/subscriptions/<subscription-id>/resourceGroups/<ts-account-resource-group>/providers/Microsoft.CodeSigning/codeSigningAccounts/<ts-account>/certificateProfiles/<ts-cert-profile>"
az role assignment create --assignee $CLIENT_ID --scope $TS_SCOPE --role "Trusted Signing Certificate Profile Signer"
```

---

5. Configure GitHub to trust your identity

   Follow [Configure an app to trust an external identity provider](/entra/workload-id/workload-identity-federation-create-trust-user-assigned-managed-identity) to allow GitHub Actions to exchange OIDC tokens for this identity.

6. Create GitHub Secrets

   Follow [creating secrets for a repository](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/use-secrets#creating-secrets-for-a-repository) to set up GitHub secrets, and map the managed identity values to those secrets:

    | GitHub secret	          | Managed identity values |
    | ----------------------- | ----------------------- |
    | `AZURE_CLIENT_ID`	      | Client ID               |
    | `AZURE_SUBSCRIPTION_ID`	| Subscription ID         |
    | `AZURE_TENANT_ID`	      | Directory (tenant) ID   |

## Store Time Stamping Authority (TSA) root certificate

Timestamping ([RFC 3161](https://www.rfc-editor.org/rfc/rfc3161)) extends trust for signatures beyond the signing certificate's validity period. Trusted Signing uses short-lived certificates, so timestamping is critical. The TSA server URL is http://timestamp.acs.microsoft.com/, as recommended per [Time stamp countersignatures](/azure/trusted-signing/concept-trusted-signing-cert-management#time-stamp-countersignatures)

1. Download the TSA root certificate

# [Linux](#tab/linux)

```bash
curl -o msft-tsa-root-certificate-authority-2020.crt "http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt"
```

# [Windows](#tab/windows)

```powershell
Invoke-WebRequest -Uri "http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt" `
    -OutFile "msft-tsa-root-certificate-authority-2020.crt"
```

---

2. Store it in your repository, for example, `.github/certs/msft-identity-verification-root-cert-authority-2020.crt`. The filepath will be used in the workflow.

## Create the GitHub Actions workflow

1. Create a `.github/workflows` directory in your repository if it doesn’t exist.  
2. Create a new workflow file, for example `.github/workflows/sign-with-trusted-signing.yml` 
3. Copy the signing workflow template below into your file.

<details>
<summary>Click here to see the signing workflow template.</summary>

```yaml
# Build and push an image to ACR, setup notation and sign the image
name: notation-github-actions-sign-with-trusted-signing-template

on:
  push:

env:
  ACR_LOGIN_SERVER: <registry-login-server>             # example: myregistry.azurecr.io
  ACR_REPO_NAME: <repository-name>                      # example: myrepo
  IMAGE_TAG: <image-tag>                                # example: v1
  PLUGIN_NAME: azure-trustedsigning                     # name of Notation Trusted Signing plugin, do not change
  PLUGIN_DOWNLOAD_URL: <plugin-download-url>            # example: "https://github.com/Azure/trustedsigning-notation-plugin/releases/download/v1.0.0-beta.1/notation-azure-trustedsigning_1.0.0-beta.1_linux_amd64.tar.gz"
  PLUGIN_CHECKSUM: <plugin-package-checksum>            # example: 538b497be0f0b4c6ced99eceb2be16f1c4b8e3d7c451357a52aeeca6751ccb44
  TSA_URL: "http://timestamp.acs.microsoft.com/"        # timestamping server URL
  TSA_ROOT_CERT: <root-cert-file-path>                  # example: .github/certs/msft-identity-verification-root-cert-authority-2020.crt
  TS_ACCOUNT_NAME: <trusted-signing-account-name>       # Trusted signing account name
  TS_CERT_PROFILE: <trusted-signing-cert-profile-name>  # Trusted signing certificate profile name 
  TS_ACCOUNT_URI: <trusted-signing-account-uri>         # Trusted signing account URI, example "https://eus.codesigning.azure.net/"

jobs:
  notation-sign:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read
     # packages: write
    steps:
      - name: Checkout
        uses: actions/checkout@v3
      - name: prepare
        id: prepare
        run: |
          echo "target_artifact_reference=${{ env.ACR_LOGIN_SERVER }}/${{ env.ACR_REPO_NAME }}:${{ env.IMAGE_TAG }}" >> "$GITHUB_ENV"
      
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
      # Build and push an image to the registry
      # Use `Dockerfile` as an example to build an image
      - name: Build and push
        id: push
        uses: docker/build-push-action@v4
        with:
          push: true
          tags: ${{ env.target_artifact_reference }}
          build-args: |
            IMAGE_TAG={{ env.IMAGE_TAG }}
      # Get the manifest digest of the OCI artifact
      - name: Retrieve digest
        run: |
          echo "target_artifact_reference=${{ env.ACR_LOGIN_SERVER }}/${{ env.ACR_REPO_NAME }}@${{ steps.push.outputs.digest }}" >> "$GITHUB_ENV" 
      # Set up Notation CLI
      - name: setup notation
        uses: notaryproject/notation-action/setup@v1.2.2
      # Sign your container images and OCI artifact using private key stored in AKV
      - name: sign OCI artifacts with Trusted Signing
        uses: notaryproject/notation-action/sign@v1
        with:
          timestamp_url: ${{ env.TSA_URL}}
          timestamp_root_cert: ${{env.TSA_ROOT_CERT }}
          plugin_name: ${{ env.PLUGIN_NAME }}
          plugin_url: ${{ env.PLUGIN_DOWNLOAD_URL }}
          plugin_checksum: ${{ env.PLUGIN_CHECKSUM }}
          key_id: ${{ env.TS_CERT_PROFILE }}
          target_artifact_reference: ${{ env.target_artifact_reference }}
          signature_format: cose
          plugin_config: |-
            accountName=${{ env.TS_ACCOUNT_NAME }}
            baseUrl=${{ env.TS_ACCOUNT_URI }}
            certProfile=${{ env.TS_CERT_PROFILE }}
          force_referrers_tag: 'false'
```
</details>

Notes on environment variables:
- PLUGIN_NAME: Always `azure-trustedsigning`.
- PLUGIN_DOWNLOAD_URL: Get from the [Trusted Signing plugin release page](https://github.com/Azure/trustedsigning-notation-plugin/releases/).
- PLUGIN_CHECKSUM: Use the checksum file on the release page, e.g. `notation-azure-trustedsigning_<version>_checksums.txt`.
- TS_ACCOUNT_URI: The endpoint for your Trusted Signing account, specific to its region, e.g. `https://eus.codesigning.azure.net/`

## Trigger the GitHub Actions workflow

The sample workflow is triggered by `on:push`. Committing changes starts the workflow. Under your GitHub repository name, select the **Actions** to view the workflow logs. 

On success, the workflow builds the image, pushes it to ACR, and signs it with Trusted Signing. You can view the workflow logs to confirm that the `azure-trustedsigning` plugin was installed and the image was successfully signed. Additionally, you can open your ACR in the Azure portal, navigate to **Repositories** -> your image, select **Referrers**, and confirm that artifacts(signatures) of type `application/vnd.cncf.notary.signature` are listed.

## Next steps

- [Verify container images in GitHub workflows with Notation and Trusted Signing](container-registry-tutorial-gha-verify-notation-ts.md)