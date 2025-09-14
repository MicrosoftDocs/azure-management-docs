---
title: Signing container images with Notation and Trusted Signing (Preview)
description: Learn how to sign and verify container images and OCI artifacts in Azure Container Registry using Notary Project tooling Notation and Trusted Signing for enhanced security and compliance.
author: yizha1
ms.author: yizha1
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 9/5/2025
# Customer intent: As a developer, I want to sign and verify container images, so that I can ensure the authenticity and integrity of my container images throughout their lifecycle.
---

# Signing container images with Notation and Trusted Signing (Preview)

This article is part of a series on ensuring integrity and authenticity of container images and OCI artifacts. For the complete picture, start with the [overview](overview-sign-and-verify-oci-artifacts.md), which explains why signing matters and various scenarios.

This article focuses on signing with Notary Project tooling, Notation and [Trusted Signing](/azure/trusted-signing/overview):

- What you’ll learn here: How to use Notation CLI to sign artifacts using Microsoft Trusted Signing.
- Where it fits: Trusted Signing is an alternative to Azure Key Vault (AKV). While AKV gives organizations full control of certificate lifecycle management, Trusted Signing provides streamlined signing experience with zero-touch certificate lifecycle management and short-lived certificates.
- Why it matters: Trusted Signing simplifies the developer experience while providing strong identity assurance, helping teams reduce operational complexity without compromising security.

## Prerequisites

### Prepare container images in your Azure Container Registry (ACR)

- Create or use an [Azure Container Registry](../container-registry/container-registry-get-started-azure-cli.md) to store container images, OCI artifacts and signatures
- Push or use a container image in your ACR

### Set up Trusted Signing

- Set up [Trusted Signing account and certificate profile](/azure/trusted-signing/quickstart) in your Azure subscription

> [!IMPORTANT]  
> Your certificate profile must include **country (C)**, **state or province (ST or S)**, and **organization (O)** in the certificate subject, as required by the [Notary Project specification](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/trust-store-trust-policy.md#trusted-identities-constraints).

### Set up Azure CLI

- Install [Azure CLI](/cli/azure/install-azure-cli), or use [Azure Cloud Shell](https://portal.azure.com/#cloudshell/)

## Install Notation CLI and Trusted Signing plugin

>[!NOTE]
>This guide runs commands on Linux amd64 as examples. 

1. **Install Notation CLI v1.3.2**

    ```bash
    curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.3.2/notation_1.3.2_linux_amd64.tar.gz
    # Validate the checksum
    EXPECTED_SHA256SUM="e1a0f060308086bf8020b2d31defb7c5348f133ca0dba6a1a7820ef3cbb6dfe5"
    echo "$EXPECTED_SHA256SUM  notation.tar.gz" | sha256sum -c -
    # Continue if the sha256sum matches
    tar xvzf notation.tar.gz
    cp ./notation /usr/local/bin
    ```

    For other platforms, see the [Notation installation guide](https://notaryproject.dev/docs/user-guides/installation/cli/).

2. **Install the Trusted Signing plugin**

    ```bash
    notation plugin install --url https://github.com/Azure/trustedsigning-notation-plugin/releases/download/v1.0.0-beta1/notation-azure-trustedsigning_1.0.0-beta1_linux_amd64.tar.gz --sha256sum 50258aad83e2fbb592ef548bf7ef4abf903590b62fd1f43a4ef4d60c201f0db5
    ```
    Find the latest plugin URL and checksum on the [release page](https://github.com/Azure/azure-trustedsigning/releases).

3. **Verify plugin installation**

    ```bash
    notation plugin ls
    ```

    Example output:
    
    ```text
    NAME                   DESCRIPTION                                            VERSION   CAPABILITIES                ERROR
    azure-trustedsigning   Sign OCI artifacts using the Trusted Signing Service   0.3.0     [SIGNATURE_GENERATOR.RAW]   <nil>
    ```

## Configure environment variables

Set the following environment variables for use in subsequent commands. Replace placeholders with your actual values.

> [!NOTE]  
> When setting up **Trusted Signing environment variables**, you can find the required values in the Azure portal:  
>  
> - **Trusted Signing account information:**  
>   `Portal > Account name > Overview`  
> - **Certificate profile information:**  
>   `Portal > Account name > Objects > Certificate Profiles`  


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

# ACR and image environment variables
ACR_SUB_ID="<acr-subscription-id>"
ACR_RG=<acr-resource-group>
ACR_NAME=<registry-name>
ACR_LOGIN_SERVER=$ACR_NAME.azurecr.io
REPOSITORY=<repository>
TAG=<tag>
IMAGE=$ACR_LOGIN_SERVER/${REPOSITORY}:$TAG
```

## Sign in to Azure

Use the Azure CLI to sign in with your **user identity**:

```bash
az login
USER_ID=$(az ad signed-in-user show --query id -o tsv)
```

> [!NOTE]
> This guide demonstrates signing in with a **user account**.  
> For other identity options, including managed identity, see [Authenticate with the Azure CLI](/cli/azure/authenticate-azure-cli).

## Assign permissions for ACR and Trusted Signing

Grant your identity the necessary roles to access ACR:
- For **ABAC-enabled** registries, assign:
  - `Container Registry Repository Reader`
  - `Container Registry Repository Writer`
- For **non-ABAC** registries, assign:
  - `AcrPull`
  - `AcrPush`

```bash
az role assignment create --role "Container Registry Repository Reader" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
az role assignment create --role "Container Registry Repository Writer" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
```

Assign the role to `Trusted Signing Certificate Profile Signer` to your identity to sign with Trusted Signing:

```bash
az role assignment create --assignee $USER_ID --role "Trusted Signing Certificate Profile Signer" --scope "/subscriptions/$TS_SUB_ID/resourceGroups/$TS_ACCT_RG/providers/Microsoft.CodeSigning/codeSigningAccounts/$TS_ACCT_NAME/certificateProfiles/$TS_CERT_PROFILE"
```

## Sign a container image

1. **Authenticate to ACR**
    
    ```bash
    az acr login --name $ACR_NAME
    ```

2. **Download the Timestamping root certificate**

    ```bash
    curl -o msft-tsa-root-certificate-authority-2020.crt $TS_TSA_ROOT_CERT
    ```

3. **Sign the image**
    
    ```bash
    notation sign --signature-format cose --timestamp-url $TS_TSA_URL --timestamp-root-cert "msft-tsa-root-certificate-authority-2020.crt" --id $TS_CERT_PROFILE --plugin azure-trustedsigning --plugin-config accountName=$TS_ACCT_NAME --plugin-config baseUrl=$TS_ACCT_URL --plugin-config certProfile=$TS_CERT_PROFILE $IMAGE
    ```
    
    Key flags explained:
    - `--signature-format cose`: Uses COSE format for signatures.
    - `--timestamp-url`: Use the timestamping server that Trusted Signing supports.
    - `--plugin-config`: Passes config to the Trusted Signing plugin.

4. **List signed images and signatures**

    ```bash
    notation ls $IMAGE
    ```

    Example output:

    ```text
    myregistry.azurecr.io/myrepo@sha256:5d0bf1e8f5a0c74a4c22d8c0f962a7cfa06a4f9d8423b196e482df8af23b5d55
    └── application/vnd.cncf.notary.signature
        └── sha256:d3a4c9fbc17e27b19a0b28e7b6a33f2c0f541dbdf8d2e5e8d0d79a835e8a76f2a
    ```

## Verify a container image

1. **Download and add root certificates**

    ```bash
    curl -o msft-root-certificate-authority-2020.crt $TS_SIGNING_ROOT_CERT
    SIGNING_TRUST_STORE="myRootCerts"
    notation cert add --type ca --store $SIGNING_TRUST_STORE msft-root-certificate-authority-2020.crt

    curl -o msft-tsa-root-certificate-authority-2020.crt $TS_TSA_ROOT_CERT
    TSA_TRUST_STORE="myTsaRootCerts"
    notation cert add -t tsa -s $TSA_TRUST_STORE msft-tsa-root-certificate-authority-2020.crt
    notation cert ls
    ```

2. **Configure trust policy**

    Create a trust policy JSON file:

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

3. **Verify the image**
    
    ```bash
    notation verify $IMAGE
    ```
    
    Example output:
    
    ```text
    Successfully verified signature for myregistry.azurecr.io/myrepo@sha256:5d0bf1e8f5a0c74a4c22d8c0f962a7cfa06a4f9d8423b196e482df8af23b5d55
    ```
    
    If verification fails, ensure your trust policy and certificates are configured correctly.


## Next steps

- [Validate container image signatures in AKS with Ratify and Azure Policy](/azure/security/container-secure-supply-chain/articles/validating-image-signatures-using-ratify-aks)
