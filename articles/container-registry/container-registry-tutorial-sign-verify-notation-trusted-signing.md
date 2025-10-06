---
title: Sign Container Images with Notation and Trusted Signing (Preview)
description: Learn how to sign and verify container images and OCI artifacts in Azure Container Registry by using Notary Project tooling, Notation, and Trusted Signing.
author: yizha1
ms.author: yizha1
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 9/5/2025
# Customer intent: As a developer, I want to sign and verify container images so that I can ensure the authenticity and integrity of those images throughout their lifecycle.
---

# Sign container images by using Notation and Trusted Signing (preview)

This article is part of a series on ensuring the integrity and authenticity of container images and other Open Container Initiative (OCI) artifacts. For the complete picture, start with the [overview](overview-sign-verify-artifacts.md), which explains why signing matters and outlines the various scenarios.

This article focuses on signing by using Notary Project tooling, Notation, and [Trusted Signing](/azure/trusted-signing/overview):

- **What you'll learn here**: How to use the Notation command-line interface (CLI) to sign artifacts by using Trusted Signing.
- **Where it fits**: Trusted Signing is an alternative to Azure Key Vault. Although Key Vault gives organizations full control of certificate lifecycle management, Trusted Signing provides streamlined signing experience with zero-touch certificate lifecycle management and short-lived certificates.
- **Why it matters**: Trusted Signing simplifies the developer experience while providing strong identity assurance. It helps teams reduce operational complexity without compromising security.

## Prerequisites

Before you can sign and verify container images with Notation and Trusted Signing, you need to set up the required Azure resources and install the necessary tools. This section walks you through preparing Azure Container Registry, configuring Trusted Signing, and setting up your development environment with the Azure CLI.

> [!NOTE]
> At this time, Trusted Signing is available only to organizations based in the United States and Canada that have a verifiable history of three years or more.

### Prepare container images in Azure Container Registry

1. Create or use a [container registry](../container-registry/container-registry-get-started-azure-cli.md) to store container images, OCI artifacts, and signatures.
1. Push or use a container image in your container registry.

### Set up Trusted Signing

Set up a [Trusted Signing account and certificate profile](/azure/trusted-signing/quickstart) in your Azure subscription.

Your certificate profile must include country/region (`C`), state or province (`ST` or `S`), and organization (`O`) in the certificate subject. The [Notary Project specification](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/trust-store-trust-policy.md#trusted-identities-constraints) requires these fields.

### Set up the Azure CLI

Install the [Azure CLI](/cli/azure/install-azure-cli), or use [Azure Cloud Shell](https://portal.azure.com/#cloudshell/).

## Install the Notation CLI and Trusted Signing plugin

This guide runs commands on Linux AMD64 and Windows as examples.

1. Install Notation CLI v1.3.2:

   # [Linux](#tab/linux)

   ```bash
   curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.3.2/notation_1.3.2_linux_amd64.tar.gz
   # Validate the checksum
   EXPECTED_SHA256SUM="e1a0f060308086bf8020b2d31defb7c5348f133ca0dba6a1a7820ef3cbb6dfe5"
   echo "$EXPECTED_SHA256SUM  notation.tar.gz" | sha256sum -c -
   # Continue if sha256sum matches
   tar xvzf notation.tar.gz
   cp ./notation /usr/local/bin
   ```

   # [Windows](#tab/windows)

   ```powershell
   # Download the Windows release
   Invoke-WebRequest -Uri "https://github.com/notaryproject/notation/releases/download/v1.3.2/notation_1.3.2_windows_amd64.zip" -OutFile notation.zip
   # Validate the checksum
   $EXPECTED_SHA256SUM = "014f25a530eee17520c8e1eb7380e4bd02ff6fc04479a07a890954e3b7ddfdc7"
   if ((Get-FileHash notation.zip -Algorithm SHA256).Hash -ne $EXPECTED_SHA256SUM) { Write-Error "Checksum mismatch"; exit 1 }
   # Expand and install
   Expand-Archive notation.zip -DestinationPath .
   # Create the installation location and move the binary to it
   New-Item -ItemType Directory -Force -Path "$Env:ProgramFiles\Notation" | Out-Null
   Move-Item -Path ".\notation\notation.exe" -Destination "$Env:ProgramFiles\Notation\notation.exe"
   # Add to PATH for the current session
   $env:PATH = "${Env:ProgramFiles}\Notation;${Env:PATH}"
   ```

   ---

   For other platforms, see the [Notation installation guide](https://notaryproject.dev/docs/user-guides/installation/cli/).

2. Install the Trusted Signing plugin:

   # [Linux](#tab/linux)

   ```bash
   notation plugin install --url "https://github.com/Azure/trustedsigning-notation-plugin/releases/download/v1.0.0-beta.1/notation-azure-trustedsigning_1.0.0-beta.1_linux_amd64.tar.gz" --sha256sum 538b497be0f0b4c6ced99eceb2be16f1c4b8e3d7c451357a52aeeca6751ccb44
   ```

   # [Windows](#tab/windows)

   ```powershell
   notation plugin install --url "https://github.com/Azure/trustedsigning-notation-plugin/releases/download/v1.0.0-beta.1/notation-azure-trustedsigning_1.0.0-beta.1_windows_amd64.zip" --sha256sum 778661034f98c455a86608b9a6426168fd81228b52112acdf75c367d5e463255
   ```

   ---

   Find the latest plugin URL and checksum on the [release page](https://github.com/Azure/azure-trustedsigning/releases).

3. Verify plugin installation:

   # [Linux](#tab/linux)

   ```bash
   notation plugin ls
   ```

   # [Windows](#tab/windows)

   ```powershell
   notation plugin ls
   ```

   ---

   Example output:

   ```text
   NAME                   DESCRIPTION                                            VERSION   CAPABILITIES                ERROR
   azure-trustedsigning   Sign OCI artifacts using the Trusted Signing Service   0.3.0     [SIGNATURE_GENERATOR.RAW]   <nil>
   ```

## Configure environment variables

Set the following environment variables for use in subsequent commands. Replace placeholders with your actual values.

You can find the required values in the Azure portal:

- For Trusted Signing account information, go to your account, and then select **Overview**.
- For certificate profile information, go to your account, and then select **Objects** > **Certificate Profiles**.

# [Linux](#tab/linux)

```bash
# Trusted Signing environment variables
TS_SUB_ID="<subscription-id>"
TS_ACCT_RG=<ts-account-resource-group>
TS_ACCT_NAME=<ts-account-name>
TS_ACCT_URL=<ts-account-url>
TS_CERT_PROFILE=<ts-cert-profile>
TS_CERT_SUBJECT=<ts-cert-subject>
TS_SIGNING_ROOT_CERT="https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt"
TS_TSA_URL="http://timestamp.acs.microsoft.com/"
TS_TSA_ROOT_CERT="http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt"

# Azure Container Registry and image environment variables
ACR_SUB_ID="<acr-subscription-id>"
ACR_RG=<acr-resource-group>
ACR_NAME=<registry-name>
ACR_LOGIN_SERVER=$ACR_NAME.azurecr.io
REPOSITORY=<repository>
TAG=<tag>
IMAGE=$ACR_LOGIN_SERVER/${REPOSITORY}:$TAG
```

# [Windows](#tab/windows)

```powershell
# Trusted Signing environment variables (current session)
$env:TS_SUB_ID = "<subscription-id>"
$env:TS_ACCT_RG = "<ts-account-resource-group>"
$env:TS_ACCT_NAME = "<ts-account-name>"
$env:TS_ACCT_URL = "<ts-account-url>"
$env:TS_CERT_PROFILE = "<ts-cert-profile>"
$env:TS_CERT_SUBJECT = "<ts-cert-subject>"
$env:TS_SIGNING_ROOT_CERT = "https://www.microsoft.com/pkiops/certs/Microsoft%20Enterprise%20Identity%20Verification%20Root%20Certificate%20Authority%202020.crt"
$env:TS_TSA_URL = "http://timestamp.acs.microsoft.com/"
$env:TS_TSA_ROOT_CERT = "http://www.microsoft.com/pkiops/certs/microsoft%20identity%20verification%20root%20certificate%20authority%202020.crt"

# Azure Container Registry and image environment variables (current session)
$env:ACR_SUB_ID = "<acr-subscription-id>"
$env:ACR_RG = "<acr-resource-group>"
$env:ACR_NAME = "<registry-name>"
$env:ACR_LOGIN_SERVER = "$($env:ACR_NAME).azurecr.io"
$env:REPOSITORY = "<repository>"
$env:TAG = "<tag>"
$env:IMAGE = "$($env:ACR_LOGIN_SERVER)/$($env:REPOSITORY):$($env:TAG)"
```

---

## Sign in to Azure

Use the Azure CLI to sign in with your user identity:

# [Linux](#tab/linux)

```bash
az login
USER_ID=$(az ad signed-in-user show --query id -o tsv)
```

# [Windows](#tab/windows)

```powershell
az login
$USER_ID = az ad signed-in-user show --query id -o tsv
```

---

> [!NOTE]
> This guide demonstrates signing in with a user account. For other identity options, including a managed identity, see [Authenticate to Azure by using the Azure CLI](/cli/azure/authenticate-azure-cli).

## Assign permissions for Azure Container Registry and Trusted Signing

Grant your identity the necessary roles to access Container Registry:

- For registries enabled with attribute-based access control (ABAC), assign:
  - `Container Registry Repository Reader`
  - `Container Registry Repository Writer`
- For non-ABAC registries, assign:
  - `AcrPull`
  - `AcrPush`

# [Linux](#tab/linux)

```bash
az role assignment create --role "Container Registry Repository Reader" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
az role assignment create --role "Container Registry Repository Writer" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
```

# [Windows](#tab/windows)

```powershell
az role assignment create --role "Container Registry Repository Reader" --assignee $USER_ID --scope "/subscriptions/$($env:ACR_SUB_ID)/resourceGroups/$($env:ACR_RG)/providers/Microsoft.ContainerRegistry/registries/$($env:ACR_NAME)"
az role assignment create --role "Container Registry Repository Writer" --assignee $USER_ID --scope "/subscriptions/$($env:ACR_SUB_ID)/resourceGroups/$($env:ACR_RG)/providers/Microsoft.ContainerRegistry/registries/$($env:ACR_NAME)"
```

---

Assign the role `Trusted Signing Certificate Profile Signer` to your identity so that you can sign with Trusted Signing:

# [Linux](#tab/linux)

```bash
az role assignment create --assignee $USER_ID --role "Trusted Signing Certificate Profile Signer" --scope "/subscriptions/$TS_SUB_ID/resourceGroups/$TS_ACCT_RG/providers/Microsoft.CodeSigning/codeSigningAccounts/$TS_ACCT_NAME/certificateProfiles/$TS_CERT_PROFILE"
```

# [Windows](#tab/windows)

```powershell
az role assignment create --assignee $USER_ID --role "Trusted Signing Certificate Profile Signer" --scope "/subscriptions/$($env:TS_SUB_ID)/resourceGroups/$($env:TS_ACCT_RG)/providers/Microsoft.CodeSigning/codeSigningAccounts/$($env:TS_ACCT_NAME)/certificateProfiles/$($env:TS_CERT_PROFILE)"
```

---

## Sign a container image

# [Linux](#tab/linux)

```bash
# Authenticate to Azure Container Registry
az acr login --name $ACR_NAME

# Download the timestamping root certificate
curl -o msft-tsa-root-certificate-authority-2020.crt $TS_TSA_ROOT_CERT

# Sign the image
notation sign --signature-format cose --timestamp-url $TS_TSA_URL --timestamp-root-cert "msft-tsa-root-certificate-authority-2020.crt" --id $TS_CERT_PROFILE --plugin azure-trustedsigning --plugin-config accountName=$TS_ACCT_NAME --plugin-config baseUrl=$TS_ACCT_URL --plugin-config certProfile=$TS_CERT_PROFILE $IMAGE
```

# [Windows](#tab/windows)

```powershell
# Authenticate to Azure Container Registry
az acr login --name $Env:ACR_NAME

# Download the timestamping root certificate
Invoke-WebRequest -Uri $Env:TS_TSA_ROOT_CERT -OutFile msft-tsa-root-certificate-authority-2020.crt

# Sign the image
notation sign --signature-format cose --timestamp-url $Env:TS_TSA_URL --timestamp-root-cert "msft-tsa-root-certificate-authority-2020.crt" --id $Env:TS_CERT_PROFILE --plugin azure-trustedsigning --plugin-config accountName=$Env:TS_ACCT_NAME --plugin-config baseUrl=$Env:TS_ACCT_URL --plugin-config certProfile=$Env:TS_CERT_PROFILE $Env:IMAGE
```

---

Key flags explained:

- `--signature-format cose`: Uses CBOR Object Signing and Encryption (COSE) format for signatures.
- `--timestamp-url`: Uses the timestamping server that Trusted Signing supports.
- `--plugin-config`: Passes configuration to the Trusted Signing plugin.

List signed images and signatures:

# [Linux](#tab/linux)

```bash
notation ls $IMAGE
```

# [Windows](#tab/windows)

```powershell
notation ls $Env:IMAGE
```

---

Example output:

```text
myregistry.azurecr.io/myrepo@sha256:5d0bf1e8f5a0c74a4c22d8c0f962a7cfa06a4f9d8423b196e482df8af23b5d55
└── application/vnd.cncf.notary.signature
    └── sha256:d3a4c9fbc17e27b19a0b28e7b6a33f2c0f541dbdf8d2e5e8d0d79a835e8a76f2a
```

## Verify a container image

1. Download and add root certificates:

   # [Linux](#tab/linux)

   ```bash
   curl -o msft-root-certificate-authority-2020.crt $TS_SIGNING_ROOT_CERT
   SIGNING_TRUST_STORE="myRootCerts"
   notation cert add --type ca --store $SIGNING_TRUST_STORE msft-root-certificate-authority-2020.crt
    
   curl -o msft-tsa-root-certificate-authority-2020.crt $TS_TSA_ROOT_CERT
   TSA_TRUST_STORE="myTsaRootCerts"
   notation cert add -t tsa -s $TSA_TRUST_STORE msft-tsa-root-certificate-authority-2020.crt
   notation cert ls
   ```

   # [Windows](#tab/windows)

   ```powershell
   Invoke-WebRequest -Uri $Env:TS_SIGNING_ROOT_CERT -OutFile msft-root-certificate-authority-2020.crt
   $SIGNING_TRUST_STORE = "myRootCerts"
   notation cert add --type ca --store $SIGNING_TRUST_STORE msft-root-certificate-authority-2020.crt
    
   Invoke-WebRequest -Uri $Env:TS_TSA_ROOT_CERT -OutFile msft-tsa-root-certificate-authority-2020.crt
   $TSA_TRUST_STORE = "myTsaRootCerts"
   notation cert add -t tsa -s $TSA_TRUST_STORE msft-tsa-root-certificate-authority-2020.crt
   notation cert ls
   ```

   ---

2. Create a trust policy JSON file:

   # [Linux](#tab/linux)

   ```bash
   cat <<EOF > trustpolicy.json
   {
       "version": "1.0",
       "trustPolicies": [
           {
               "name": "myPolicy",
               "registryScopes": [ "$ACR_LOGIN_SERVER/$REPOSITORY" ],
               "signatureVerification": {
                   "level" : "strict"
               },
               "trustStores": [ "ca:$SIGNING_TRUST_STORE", "tsa:$TSA_TRUST_STORE" ],
               "trustedIdentities": [
                   "x509.subject: $TS_CERT_SUBJECT"
               ]
           }
       ]
   }
   EOF
   ```

   Import and check the policy:

   ```bash
   notation policy import trustpolicy.json
   notation policy show
   ```

   # [Windows](#tab/windows)

   ```powershell
   @"
   {
       "version": "1.0",
       "trustPolicies": [
           {
               "name": "myPolicy",
               "registryScopes": [ "$($env:ACR_LOGIN_SERVER)/$($env:REPOSITORY)" ],
               "signatureVerification": {
                   "level" : "strict"
               },
               "trustStores": [ "ca:$($SIGNING_TRUST_STORE)", "tsa:$($TSA_TRUST_STORE)" ],
               "trustedIdentities": [
                   "x509.subject: $($env:TS_CERT_SUBJECT)"
               ]
           }
       ]
   }
   "@ | Out-File -FilePath trustpolicy.json -Encoding utf8
   ```

   Import and check the policy:

   ```powershell
   notation policy import trustpolicy.json
   notation policy show
   ```

   ---

3. Verify the image:

   # [Linux](#tab/linux)

   ```bash
   notation verify $IMAGE
   ```

   # [Windows](#tab/windows)

   ```powershell
   notation verify $Env:IMAGE
   ```

   ---

   Example output:

   ```text
   Successfully verified signature for myregistry.azurecr.io/myrepo@sha256:5d0bf1e8f5a0c74a4c22d8c0f962a7cfa06a4f9d8423b196e482df8af23b5d55
   ```

   If verification fails, ensure that your trust policy and certificates are configured correctly.

## Related content

- For signing in a GitHub workflow, see [Sign container images in a GitHub workflow by using Notation and Trusted Signing (preview)](container-registry-tutorial-github-sign-notation-trusted-signing.md).
- For verification in a GitHub workflow, see [Verify container images in a GitHub workflow by using Notation and Trusted Signing (preview)](container-registry-tutorial-github-verify-notation-trusted-signing.md).
- For verification on Azure Kubernetes Service (AKS), see [Verify container image signatures by using Ratify and Azure Policy](container-registry-tutorial-verify-with-ratify-aks.md).
