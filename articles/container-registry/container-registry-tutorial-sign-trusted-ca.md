---
title: Sign Container Images with Notation and Azure Key Vault by Using a CA-Issued Certificate
description: Learn how to create a CA-issued certificate in Azure Key Vault, sign a container image in Azure Container Registry with Notation and Key Vault, and verify the image.
author: chasedmicrosoft
ms.author: doveychase
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 9/5/2024
# Customer intent: "As a developer, I want to sign and verify container images by using a CA-issued certificate stored in a secure vault, so that I can ensure their integrity and authenticity throughout the deployment process."
---

# Sign container images by using Notation, Azure Key Vault, and a CA-issued certificate

This article is part of a series on ensuring integrity and authenticity of container images and other Open Container Initiative (OCI) artifacts.
For the complete picture, start with the [overview](overview-sign-verify-artifacts.md), which explains why signing matters and outlines the various scenarios.

Signing and verifying container images by using a certificate from a trusted certificate authority (CA) is a valuable security practice. It helps you responsibly identify, authorize, and validate the identity of both the publisher of a container image and the container image itself. Trusted CAs such as GlobalSign, DigiCert, and others play a crucial role in:

- Validating a user's or organization's identity.
- Maintaining the security of digital certificates.
- Revoking certificates immediately upon any risk or misuse.

Here are some essential components that help you to sign and verify container images by using a certificate from a trusted CA:

- [Notation](https://github.com/notaryproject/notation) is an open-source supply-chain security tool developed by the [Notary Project community](https://notaryproject.dev/) and backed by Microsoft. It supports signing and verifying container images and other artifacts.
- Azure Key Vault is a cloud-based service for managing cryptographic keys, secrets, and certificates. It helps you securely store and manage a certificate with a signing key.
- The [Key Vault plugin (`notation-azure-kv`)](https://github.com/Azure/notation-azure-kv) is an extension of Notation. It uses the keys stored in Key Vault for signing and verifying the digital signatures of container images and artifacts.
- Azure Container Registry is a private registry that you can use to attach signatures to container images, along with storing and managing these images.

When you verify an image, the signature is used to validate the integrity of the image and the identity of the signer. This validation helps ensure that container images aren't tampered with and are from a trusted source.

In this article, you learn how to:

- Install the Notation command-line interface (CLI) and the Key Vault plugin.
- Create or import a certificate issued by a CA in Key Vault.
- Build and push a container image by using Container Registry tasks.
- Sign a container image by using the Notation CLI and the Key Vault plugin.
- Verify a container image signature by using the Notation CLI.
- Use timestamping.

## Prerequisites

- Create or use a [container registry](../container-registry/container-registry-get-started-azure-cli.md) for storing container images and signatures.
- Create or use a [key vault](/azure/key-vault/general/quick-create-cli). We recommend that you create a new key vault for storing certificates only.
- Install and configure the latest [Azure CLI](/cli/azure/install-azure-cli) version, or run commands in [Azure Cloud Shell](https://portal.azure.com/#cloudshell/).

## Install the Notation CLI and Key Vault plugin

1. Install Notation v1.3.2 in a Linux AMD64 environment. To download the package for other environments, follow the [Notation installation guide](https://notaryproject.dev/docs/user-guides/installation/cli/).

    ```bash
    # Download, extract, and install
    curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.3.2/notation_1.3.2_linux_amd64.tar.gz
    tar xvzf notation.tar.gz

    # Copy the Notation CLI to the desired bin directory in PATH, for example
    cp ./notation /usr/local/bin
    ```

2. Install Key Vault plugin (`notation-azure-kv`) v1.2.1 in a Linux AMD64 environment.

    > [!NOTE]
    > You can find the URL and SHA256 checksum for the plugin on the plugin's [release page](https://github.com/Azure/notation-azure-kv/releases).

    ```bash
    notation plugin install --url https://github.com/Azure/notation-azure-kv/releases/download/v1.2.1/notation-azure-kv_1.2.1_linux_amd64.tar.gz --sha256sum 67c5ccaaf28dd44d2b6572684d84e344a02c2258af1d65ead3910b3156d3eaf5
    ```

3. List the available plugins and confirm that the `notation-azure-kv` plugin with version `1.2.1` is included in the list:

    ```bash
    notation plugin ls
    ```

## Configure environment variables

This article uses environment variables for convenience in the configuration of Key Vault and Container Registry. Update the values of these environment variables for your specific resources.

1. Configure environment variables for Key Vault and certificates:

    ```bash
    AKV_SUB_ID=myAkvSubscriptionId
    AKV_RG=myAkvResourceGroup
    AKV_NAME=myakv 
    
    # Name of the certificate created or imported in Key Vault 
    CERT_NAME=wabbit-networks-io 
    
    # X.509 certificate subject
    CERT_SUBJECT="CN=wabbit-networks.io,O=Notation,L=Seattle,ST=WA,C=US"
    ```

2. Configure environment variables for Container Registry and images:

    ```bash
    ACR_SUB_ID=myAcrSubscriptionId
    ACR_RG=myAcrResourceGroup
    # Name of the existing registry example: myregistry.azurecr.io 
    ACR_NAME=myregistry 
    # Existing full domain of the container registry 
    REGISTRY=$ACR_NAME.azurecr.io 
    # Container name inside the container registry where the image will be stored 
    REPO=net-monitor 
    TAG=v1 
    # Source code directory that contains the Dockerfile to build 
    IMAGE_SOURCE=https://github.com/wabbit-networks/net-monitor.git#main  
    ```

## Sign in by using the Azure CLI

```bash
az login
```

For more information, see [Authenticate to Azure by using the Azure CLI](/cli/azure/authenticate-azure-cli).

## Create or import a certificate issued by a CA in Key Vault

### Understand certificate requirements

When you're creating certificates for signing and verification, the certificates must meet the [Notary Project certificate requirements](https://github.com/notaryproject/specifications/blob/v1.0.0/specs/signature-specification.md#certificate-requirements).

Here are the requirements for root and intermediate certificates:

- The `basicConstraints` extension must be present and marked as `critical`. The `CA` field must be set to `true`.
- The `keyUsage` extension must be present and marked as `critical`. Bit positions for `keyCertSign` must be set.

Here are the requirements for certificates that a CA issues:

- X.509 certificate properties:
  - Subject must contain common name (`CN`), country/region (`C`), state or province (`ST`), and organization (`O`). This article uses `$CERT_SUBJECT` as the subject.
  - X.509 key usage flag must be `DigitalSignature` only.
  - Extended Key Usages (EKUs) must be empty or `1.3.6.1.5.5.7.3.3` (for code signing).
- Key properties:
  - The `exportable` property must be set to `false`.
  - Select a supported key type and size from the [Notary Project specification](https://github.com/notaryproject/specifications/blob/v1.0.0/specs/signature-specification.md#algorithm-selection).

> [!IMPORTANT]
> To ensure successful integration with [Image Integrity](/azure/aks/image-integrity), the content type of certificate should be set to PEM.
>
> This guide uses version 1.0.1 of the Key Vault plugin. Prior versions of the plugin had a limitation that required a specific certificate order in a certificate chain. Version 1.0.1 of the plugin doesn't have this limitation, so we recommend that you use version 1.0.1 or later.

### Create a certificate issued by a CA

Create a certificate signing request (CSR) by following the instructions in [Create and merge a certificate signing request in Key Vault](/azure/key-vault/certificates/create-certificate-signing-request).

When you're merging the CSR, make sure that you merge the entire chain that you brought back from the CA vendor.

### Import the certificate in Key Vault

To import the certificate:

1. Get the certificate file from CA vendor with the entire certificate chain.
2. Import the certificate into Key Vault by following the instructions in [Import a certificate in Azure Key Vault](/azure/key-vault/certificates/tutorial-import-certificate).

If the certificate doesn't contain a certificate chain after creation or importing, you can obtain the intermediate and root certificates from your CA vendor. You can ask your vendor to provide you with a PEM file that contains the intermediate certificates (if any) and root certificate. You can then use this file when you're signing container images.

## Sign a container image by using the Notation CLI and Key Vault plugin

When you're working with Container Registry and Key Vault, it's essential to grant the appropriate permissions to help ensure secure and controlled access. You can authorize access for various entities, such as user principals, service principals, or managed identities, depending on your specific scenarios. In this article, the access is authorized to a signed-in Azure user.

### Authorize access to Container Registry

For registries enabled for Microsoft Entra attribute-based access control (ABAC), the `Container Registry Repository Reader` and `Container Registry Repository Writer` roles are required for building and signing container images in Container Registry.

For registries not enabled for ABAC, the `AcrPull` and `AcrPush` roles are required.

For more information on ABAC, see [Microsoft Entra attribute-based access control for repository permissions (preview)](container-registry-rbac-abac-repository-permissions.md).

1. Set the subscription that contains the Container Registry resource:

    ```bash
    az account set --subscription $ACR_SUB_ID
    ```

1. Assign the roles:

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    ROLE1="Container Registry Repository Reader" # For ABAC-enabled registries. Otherwise, use "AcrPull" for non-ABAC-enabled registries.
    ROLE2="Container Registry Repository Writer" # For ABAC-enabled registries. Otherwise, use "AcrPush" for non-ABAC-enabled registries.
    az role assignment create --role "$ROLE1" --role "$ROLE2" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
    ```

### Build and push container images to Container Registry

1. Authenticate to your container registry by using your individual Azure identity:

    ```bash
    az acr login --name $ACR_NAME
    ```

    > [!IMPORTANT]
    > If you have Docker installed on your system and used `az acr login` or `docker login` to authenticate to your container registry, your credentials are already stored and available to Notation. In this case, you don't need to run `notation login` again to authenticate to your container registry. To learn more about authentication options for Notation, see [Authenticate with OCI-compliant registries](https://notaryproject.dev/docs/user-guides/how-to/registry-authentication/).

1. Build and push a new image by using Container Registry tasks. Always use `digest` to identify the image for signing, because tags are mutable and can be overwritten.

    ```bash
    DIGEST=$(az acr build -r $ACR_NAME -t $REGISTRY/${REPO}:$TAG $IMAGE_SOURCE --no-logs --query "outputImages[0].digest" -o tsv)
    IMAGE=$REGISTRY/${REPO}@$DIGEST
    ```

    In this article, if the image is already built and is stored in the registry, the tag serves as an identifier for that image for convenience:

    ```bash
    IMAGE=$REGISTRY/${REPO}@$TAG
    ```

### Authorize access to Key Vault

This section explores two options for authorizing access to Key Vault.

#### Use Azure RBAC (recommended)

1. Set the subscription that contains the Key Vault resource:

    ```bash
    az account set --subscription $AKV_SUB_ID
    ```

1. Assign the roles.

    If the certificate contains the entire certificate chain, the principal must be assigned with the following roles:

    - `Key Vault Secrets User` for reading secrets
    - `Key Vault Certificates User` for reading certificates
    - `Key Vault Crypto User` for signing operations

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az role assignment create --role "Key Vault Secrets User" --role "Key Vault Certificates User" --role "Key Vault Crypto User" --assignee $USER_ID --scope "/subscriptions/$AKV_SUB_ID/resourceGroups/$AKV_RG/providers/Microsoft.KeyVault/vaults/$AKV_NAME"
    ```

    If the certificate doesn't contain the chain, the principal must be assigned with the following roles:

    - `Key Vault Certificates User` for reading certificates
    - `Key Vault Crypto User` for signing operations

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az role assignment create --role "Key Vault Certificates User" --role "Key Vault Crypto User" --assignee $USER_ID --scope "/subscriptions/$AKV_SUB_ID/resourceGroups/$AKV_RG/providers/Microsoft.KeyVault/vaults/$AKV_NAME"
    ```

To learn more about Key Vault access with Azure role-based access control (RBAC), see [Provide access to Key Vault keys, certificates, and secrets with Azure role-based access control](/azure/key-vault/general/rbac-guide).

#### Use an access policy (legacy)

To set the subscription that contains the Key Vault resources, run the following command:

```bash
az account set --subscription $AKV_SUB_ID
```

If the certificate contains the entire certificate chain, the principal must be granted the key permission `Sign`, the secret permission `Get`, and the certificate permission `Get`. To grant these permissions to the principal, use this command:

```bash
USER_ID=$(az ad signed-in-user show --query id -o tsv)
az keyvault set-policy -n $AKV_NAME --key-permissions sign --secret-permissions get --certificate-permissions get --object-id $USER_ID
```

If the certificate doesn't contain the chain, the principal must be granted the key permission `Sign` and the certificate permission `Get`. To grant these permissions to the principal, use this command:

```bash
USER_ID=$(az ad signed-in-user show --query id -o tsv)
az keyvault set-policy -n $AKV_NAME --key-permissions sign --certificate-permissions get --object-id $USER_ID
```

To learn more about assigning a policy to a principal, see [Assign a Key Vault access policy (legacy)](/azure/key-vault/general/assign-access-policy).

### Sign container images by using the certificate in Key Vault

1. Get the key ID for a certificate. A certificate in Key Vault can have multiple versions. The following command gets the key ID for the latest version of the `$CERT_NAME` certificate:

   ```bash
   KEY_ID=$(az keyvault certificate show -n $CERT_NAME --vault-name $AKV_NAME --query 'kid' -o tsv) 
   ```

1. Sign the container image with the CBOR Object Signing and Encryption (COSE) signature format by using the key ID.

   If the certificate contains the entire certificate chain, run the following command:

   ```bash
   notation sign --signature-format cose $IMAGE --id $KEY_ID --plugin azure-kv 
   ```

   If the certificate doesn't contain the chain, use the `--plugin-config ca_certs=<ca_bundle_file>` parameter to pass the CA certificates in a PEM file to the Key Vault plugin. Run the following command:

   ```bash
   notation sign --signature-format cose $IMAGE --id $KEY_ID --plugin azure-kv --plugin-config ca_certs=<ca_bundle_file> 
   ```

   To authenticate with Key Vault, by default, the following credential types (if enabled) are tried in order:

   - [Environment credential](/dotnet/api/azure.identity.environmentcredential)
   - [Workload identity credential](/dotnet/api/azure.identity.workloadidentitycredential)
   - [Managed identity credential](/dotnet/api/azure.identity.managedidentitycredential)
   - [Azure CLI credential](/dotnet/api/azure.identity.azureclicredential)

   If you want to specify a credential type, use an additional plugin configuration called `credential_type`. For example, you can explicitly set `credential_type` to `azurecli` for using an Azure CLI credential, as demonstrated in this example:

   ```bash
   notation sign --signature-format cose --id $KEY_ID --plugin azure-kv --plugin-config credential_type=azurecli $IMAGE
   ```

   The following table shows the values of `credential_type` for various credential types.

   | Credential type              | Value for `credential_type` |
   | ---------------------------- | -------------------------- |
   | Environment credential       | `environment`              |
   | Workload identity credential | `workloadid`               |
   | Managed identity credential  | `managedid`                |
   | Azure CLI credential         | `azurecli`                 |

1. View the graph of signed images and associated signatures:

    ```bash
    notation ls $IMAGE
    ```

    In the following example output, a signature of type `application/vnd.cncf.notary.signature` identified by digest `sha256:d7258166ca820f5ab7190247663464f2dcb149df4d1b6c4943dcaac59157de8e` is associated with `$IMAGE`:

    ```output
    myregistry.azurecr.io/net-monitor@sha256:17cc5dd7dfb8739e19e33e43680e43071f07497ed716814f3ac80bd4aac1b58f
    └── application/vnd.cncf.notary.signature
        └── sha256:d7258166ca820f5ab7190247663464f2dcb149df4d1b6c4943dcaac59157de8e
    ```

> [!NOTE]
> Since Notation v1.2.0, Notation uses the [OCI referrers tag schema](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#referrers-tag-schema) to store the signature in Container Registry by default. You can also enable the [OCI Referrers API](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-referrers) by using the flag `--force-referrers-tag false`, if necessary. Container Registry features support the OCI Referrers API, except for the registry encrypted via customer-managed keys (CMKs).

## Verify a container image by using the Notation CLI

1. Add the root certificate to a named trust store for signature verification. If you don't have the root certificate, you can obtain it from your CA. The following example adds the root certificate `$ROOT_CERT` to the `$STORE_NAME` trust store:

    ```bash
    STORE_TYPE="ca" 
    STORE_NAME="wabbit-networks.io" 
    notation cert add --type $STORE_TYPE --store $STORE_NAME $ROOT_CERT  
    ```

2. List the root certificate to confirm that the `$ROOT_CERT` is added successfully:

    ```bash
    notation cert ls 
    ```

3. Configure a trust policy before verification. Trust policies enable users to specify fine-tuned verification policies. Use the following command:

    ```bash
    cat <<EOF > ./trustpolicy.json
    {
        "version": "1.0",
        "trustPolicies": [
            {
                "name": "wabbit-networks-images",
                "registryScopes": [ "$REGISTRY/$REPO" ],
                "signatureVerification": {
                    "level" : "strict" 
                },
                "trustStores": [ "$STORE_TYPE:$STORE_NAME" ],
                "trustedIdentities": [
                    "x509.subject: $CERT_SUBJECT"
                ]
            }
        ]
    }
    EOF
    ```

    The preceding `trustpolicy.json` file defines one trust policy named `wabbit-networks-images`. This trust policy applies to all the artifacts stored in the `$REGISTRY/$REPO` repositories. The named trust store `$STORE_NAME` of type `$STORE_TYPE` contains the root certificates. This policy also assumes that the user trusts a specific identity with the X.509 subject `$CERT_SUBJECT`. For more information, see [Trust store and trust policy specification](https://github.com/notaryproject/notaryproject/blob/v1.0.0/specs/trust-store-trust-policy.md).

4. Use `notation policy` to import the trust policy configuration from `trustpolicy.json`:

    ```bash
    notation policy import ./trustpolicy.json
    ```

5. Show the trust policy configuration to confirm its successful import:

    ```bash
    notation policy show
    ```

6. Use `notation verify` to verify the integrity of the image:

    ```bash
    notation verify $IMAGE
    ```

   Upon successful verification of the image via the trust policy, the SHA256 digest of the verified image is returned in a successful output message. Here's an example of the output:

   `Successfully verified signature for myregistry.azurecr.io/net-monitor@sha256:17cc5dd7dfb8739e19e33e43680e43071f07497ed716814f3ac80bd4aac1b58f`

## Use timestamping

Since the Notation v1.2.0 release, Notation supports [RFC 3161](https://www.rfc-editor.org/rfc/rfc3161)-compliant timestamping. This enhancement extends the trust of signatures created within the certificate's validity period by trusting a Time Stamping Authority (TSA). This trust enables successful signature verification even after the certificates expire.

As an image signer, you should ensure that you sign container images with time stamps that a trusted TSA generated. As an image verifier, you should ensure that you trust both the image signer and the associated TSA, and establish trust through trust stores and trust policies.

Timestamping reduces costs by eliminating the need to periodically re-sign images due to certificate expiry. This ability is especially critical when you use short-lived certificates. For detailed instructions on how to sign and verify images by using timestamping, refer to the [Notary Project timestamping guide](https://v1-2.notaryproject.dev/docs/user-guides/how-to/timestamping/).

## FAQ

- What should I do if the certificate expires?

  If your certificate expires, you need to obtain a new one from a trusted CA vendor, along with a new private key. You can't use an expired certificate to sign container images.
  
  Images that were signed before the certificate expired might still be validated successfully if they were signed with [timestamping](#use-timestamping). Without timestamping, the signature verification fails, and you need to re-sign those images with the new certificate for successful verification.

- What should I do if the certificate is revoked?
  
  Revocation of your certificate invalidates the signature. This situation can happen for several reasons, such as compromise of the private key or changes in the certificate holder's affiliation.
  
  To resolve this problem, you should first ensure that your source code and build environment are up to date and secure. Then, build container images from the source code, obtain a new certificate from a trusted CA vendor along with a new private key, and sign new container images with the new certificate by following this guide.

## Related content

Notation provides continuous integration and continuous delivery (CI/CD) solutions on Azure Pipelines and GitHub Actions:

- To sign and verify container images in an Azure DevOps pipeline, see [Sign and verify a container image by using Notation in an Azure pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign).
- To sign container images by using GitHub Actions, see [Sign a container image by using Notation in GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha).
- To verify container images by using GitHub Actions, see [Verify a container image by using Notation in GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha).

To ensure that only trusted container images are deployed on Azure Kubernetes Service (AKS):

- Use Azure Policy Image Integrity (preview) by following the guide [Use Image Integrity to validate signed images before deploying them to your Azure Kubernetes Service clusters (preview)](/azure/aks/image-integrity?tabs=azure-cli).
- Use [Ratify](https://ratify.dev/) and Azure Policy by following the guide [Securing AKS workloads: validating container image signatures with Ratify and Azure Policy](container-registry-tutorial-verify-with-ratify-aks.md).
