---
title: Sign Container Images with Notation and a CA-issued certificate Azure Key Vault
description: Learn to create a CA-issued certificate in Azure Key Vault, sign a container image in Azure Container Registry with Notation and AKV, and verify it.
author: yizha1
ms.author: yizha1
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 9/5/2024
#customer intent: As a developer, I want to sign and verify container images using a CA-issued certificate in Azure Key Vault so that I can ensure the integrity and authenticity of my images.
---

# Sign container images with Notation and Azure Key Vault using a CA-issued certificate

Signing and verifying container images with a certificate issued by a trusted Certificate Authority (CA) is a valuable security practice. This security measure will help you to responsibly identify, authorize, and validate the identity of both the publisher of the container image and the container image itself. The Trusted Certificate Authorities (CAs) such as GlobalSign, DigiCert, and others play a crucial role in the validation of a user's or organization's identity, maintaining the security of digital certificates, and revoking the certificate immediately upon any risk or misuse.

Here are some essential components that help you to sign and verify container images with a certificate issued by a trusted CA:

* The [Notation](https://github.com/notaryproject/notation) is an open-source supply chain security tool developed by [Notary Project community](https://notaryproject.dev/) and backed by Microsoft, which supports signing and verifying container images and other artifacts.
* The Azure Key Vault (AKV), a cloud-based service for managing cryptographic keys, secrets, and certificates will help you ensure to securely store and manage a certificate with a signing key.
* The [Notation AKV plugin azure-kv](https://github.com/Azure/notation-azure-kv), the extension of Notation uses the keys stored in Azure Key Vault for signing and verifying the digital signatures of container images and artifacts.
* The Azure Container Registry (ACR) allows you to attach these signatures to the signed image and helps you to store and manage these container images.

When you verify the image, the signature is used to validate the integrity of the image and the identity of the signer. This helps to ensure that the container images are not tampered with and are from a trusted source.

In this article:

> [!div class="checklist"]
> * Install the notation CLI and AKV plugin
> * Create or import a certificate issued by a CA in AKV
> * Build and push a container image with ACR task
> * Sign a container image with Notation CLI and AKV plugin 
> * Verify a container image signature with Notation CLI
> * Timestamping

## Prerequisites

* Create or use an [Azure Container Registry](../container-registry/container-registry-get-started-azure-cli.md) for storing container images and signatures
* Create or use an [Azure Key Vault.](/azure/key-vault/general/quick-create-cli)
* Install and configure the latest [Azure CLI](/cli/azure/install-azure-cli), or run commands in the [Azure Cloud Shell](https://portal.azure.com/#cloudshell/)

> [!NOTE]
> We recommend creating a new Azure Key Vault for storing certificates only.

## Install the notation CLI and AKV plugin

1. Install Notation v1.3.0 on a Linux amd64 environment. Follow the [Notation installation guide](https://notaryproject.dev/docs/user-guides/installation/cli/) to download the package for other environments.

    ```bash
    # Download, extract and install
    curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.3.0/notation_1.3.0_linux_amd64.tar.gz
    tar xvzf notation.tar.gz

    # Copy the notation cli to the desired bin directory in your PATH, for example
    cp ./notation /usr/local/bin
    ```

2. Install the Notation Azure Key Vault plugin `azure-kv` v1.2.1 on a Linux amd64 environment.

    > [!NOTE]
    > The URL and SHA256 checksum for the Notation Azure Key Vault plugin can be found on the plugin's [release page](https://github.com/Azure/notation-azure-kv/releases).

    ```bash
    notation plugin install --url https://github.com/Azure/notation-azure-kv/releases/download/v1.2.1/notation-azure-kv_1.2.1_linux_amd64.tar.gz --sha256sum 67c5ccaaf28dd44d2b6572684d84e344a02c2258af1d65ead3910b3156d3eaf5
    ```

3. List the available plugins and confirm that the `azure-kv` plugin with version `1.2.1` is included in the list. 

    ```bash
    notation plugin ls
    ```

## Configure environment variables 

> [!NOTE]
> This guide uses environment variables for convenience when configuring the AKV and ACR. Update the values of these environment variables for your specific resources.

1. Configure environment variables for AKV and certificates

    ```bash
    AKV_SUB_ID=myAkvSubscriptionId
    AKV_RG=myAkvResourceGroup
    AKV_NAME=myakv 
    
    # Name of the certificate created or imported in AKV 
    CERT_NAME=wabbit-networks-io 
    
    # X.509 certificate subject
    CERT_SUBJECT="CN=wabbit-networks.io,O=Notation,L=Seattle,ST=WA,C=US"
    ```

2. Configure environment variables for ACR and images. 
    
    ```bash
    ACR_SUB_ID=myAcrSubscriptionId
    ACR_RG=myAcrResourceGroup
    # Name of the existing registry example: myregistry.azurecr.io 
    ACR_NAME=myregistry 
    # Existing full domain of the ACR 
    REGISTRY=$ACR_NAME.azurecr.io 
    # Container name inside ACR where image will be stored 
    REPO=net-monitor 
    TAG=v1 
    # Source code directory containing Dockerfile to build 
    IMAGE_SOURCE=https://github.com/wabbit-networks/net-monitor.git#main  
    ```

## Sign in with Azure CLI

```bash
az login
```

To learn more about Azure CLI and how to sign in with it, see [Sign in with Azure CLI](/cli/azure/authenticate-azure-cli).


## Create or import a certificate issued by a CA in AKV

### Certificate requirements

When creating certificates for signing and verification, the certificates must meet the [Notary Project certificate requirement](https://github.com/notaryproject/specifications/blob/v1.0.0/specs/signature-specification.md#certificate-requirements). 

Here are the requirements for root and intermediate certificates:
- The `basicConstraints` extension must be present and marked as critical. The `CA` field must be set `true`.
- The `keyUsage` extension must be present and marked `critical`. Bit positions for `keyCertSign` MUST be set. 

Here are the requirements for certificates issued by a CA:
- X.509 certificate properties:
  - Subject must contain common name (`CN`), country (`C`), state or province (`ST`), and organization (`O`). In this tutorial, `$CERT_SUBJECT` is used as the subject.
  - X.509 key usage flag must be `DigitalSignature` only.
  - Extended Key Usages (EKUs) must be empty or `1.3.6.1.5.5.7.3.3` (for Codesigning).
- Key properties:
  - The `exportable` property must be set to `false`.
  - Select a supported key type and size from the [Notary Project specification](https://github.com/notaryproject/specifications/blob/v1.0.0/specs/signature-specification.md#algorithm-selection).

> [!IMPORTANT]
> To ensure successful integration with [Image Integrity](/azure/aks/image-integrity), the content type of certificate should be set to PEM.

> [!NOTE]
> This guide uses version 1.0.1 of the AKV plugin. Prior versions of the plugin had a limitation that required a specific certificate order in a certificate chain. Version 1.0.1 of the plugin does not have this limitation so it is recommended that you use version 1.0.1 or later.

### Create a certificate issued by a CA

Create a certificate signing request (CSR) by following the instructions in [create certificate signing request](/azure/key-vault/certificates/create-certificate-signing-request). 

> [!IMPORTANT]
> When merging the CSR, make sure you merge the entire chain that brought back from the CA vendor.

### Import the certificate in AKV

To import the certificate:

1. Get the certificate file from CA vendor with entire certificate chain.
2. Import the certificate into Azure Key Vault by following the instructions in [import a certificate](/azure/key-vault/certificates/tutorial-import-certificate).

> [!NOTE]
> If the certificate does not contain a certificate chain after creation or importing, you can obtain the intermediate and root certificates from your CA vendor. You can ask your vendor to provide you with a PEM file that contains the intermediate certificates (if any) and root certificate. This file can then be used at step 5 of [signing container images](#sign-a-container-image-with-notation-cli-and-akv-plugin).

## Sign a container image with Notation CLI and AKV plugin

When working with ACR and AKV, it’s essential to grant the appropriate permissions to ensure secure and controlled access. You can authorize access for different entities, such as user principals, service principals, or managed identities, depending on your specific scenarios. In this tutorial, the access is authorized to a signed-in Azure user.

### Authoring access to ACR

For registries enabled for Microsoft Entra attribute-based access control (ABAC), the `Container Registry Repository Reader` and `Container Registry Repository Writer` roles are required for building and signing container images in ACR.

For registries not enabled for ABAC, the `AcrPull` and `AcrPush` roles are required.

For more information on Microsoft Entra ABAC, see [Microsoft Entra-based repository permissions](container-registry-rbac-abac-repository-permissions.md).

1. Set the subscription that contains the ACR resource

    ```bash
    az account set --subscription $ACR_SUB_ID
    ```

1. Assign the roles

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    ROLE1="Container Registry Repository Reader" # For ABAC-enabled registries. Otherwise, use "AcrPull" for non-ABAC-enabled registries.
    ROLE2="Container Registry Repository Writer" # For ABAC-enabled registries. Otherwise, use "AcrPush" for non-ABAC-enabled registries.
    az role assignment create --role "$ROLE1" --role "$ROLE2" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
    ```

### Build and push container images to ACR

1. Authenticate to your ACR by using your individual Azure identity.

    ```bash
    az acr login --name $ACR_NAME
    ```

    > [!IMPORTANT]
    > If you have Docker installed on your system and used `az acr login` or `docker login` to authenticate to your ACR, your credentials are already stored and available to notation. In this case, you don’t need to run `notation login` again to authenticate to your ACR. To learn more about authentication options for notation, see [Authenticate with OCI-compliant registries](https://notaryproject.dev/docs/user-guides/how-to/registry-authentication/).

1. Build and push a new image with ACR Tasks. Always use `digest` to identify the image for signing, since tags are mutable and can be overwritten.

    ```bash
    DIGEST=$(az acr build -r $ACR_NAME -t $REGISTRY/${REPO}:$TAG $IMAGE_SOURCE --no-logs --query "outputImages[0].digest" -o tsv)
    IMAGE=$REGISTRY/${REPO}@$DIGEST
    ```

In this tutorial, if the image has already been built and is stored in the registry, the tag serves as an identifier for that image for convenience.

```bash
IMAGE=$REGISTRY/${REPO}@$TAG
```

### Authoring access to AKV

#### Use Azure RBAC (Recommended)

1. Set the subscription that contains the AKV resource

    ```bash
    az account set --subscription $AKV_SUB_ID
    ```

1. Assign the roles

    If the certificate contains the entire certificate chain, the principal must be assigned with the following roles: 
    - `Key Vault Secrets User` for reading secrets
    - `Key Vault Certificates User`for reading certificates
    - `Key Vault Crypto User` for signing operations

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az role assignment create --role "Key Vault Secrets User" --role "Key Vault Certificates User" --role "Key Vault Crypto User" --assignee $USER_ID --scope "/subscriptions/$AKV_SUB_ID/resourceGroups/$AKV_RG/providers/Microsoft.KeyVault/vaults/$AKV_NAME"
    ```

    If the certificate doesn't contain the chain, the principal must be assigned with the following roles:
    - `Key Vault Certificates User`for reading certificates
    - `Key Vault Crypto User` for signing operations

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az role assignment create --role "Key Vault Certificates User" --role "Key Vault Crypto User" --assignee $USER_ID --scope "/subscriptions/$AKV_SUB_ID/resourceGroups/$AKV_RG/providers/Microsoft.KeyVault/vaults/$AKV_NAME"
    ```

To learn more about Key Vault access with Azure RBAC, see [Use an Azure RBAC for managing access](/azure/key-vault/general/rbac-guide).

#### Use access policy (Legacy)
    
To set the subscription that contains the AKV resources, run the following command:

```bash
az account set --subscription $AKV_SUB_ID
```

If the certificate contains the entire certificate chain, the principal must be granted key permission `Sign`, secret permission `Get`, and certificate permissions `Get`. To grant these permissions to the principal:

```bash
USER_ID=$(az ad signed-in-user show --query id -o tsv)
az keyvault set-policy -n $AKV_NAME --key-permissions sign --secret-permissions get --certificate-permissions get --object-id $USER_ID
```

If the certificate doesn't contain the chain, the principal must be granted key permission `Sign`, and certificate permissions `Get`. To grant these permissions to the principal:

```bash
USER_ID=$(az ad signed-in-user show --query id -o tsv)
az keyvault set-policy -n $AKV_NAME --key-permissions sign --certificate-permissions get --object-id $USER_ID
```

To learn more about assigning policy to a principal, see [Assign Access Policy](/azure/key-vault/general/assign-access-policy). 

### Sign container images using the certificate in AKV

1. Get the Key ID for a certificate. A certificate in AKV can have multiple versions, the following command gets the Key ID for the latest version of the `$CERT_NAME` certificate.

   ```bash
   KEY_ID=$(az keyvault certificate show -n $CERT_NAME --vault-name $AKV_NAME --query 'kid' -o tsv) 
   ```

1. Sign the container image with the COSE signature format using the Key ID. 

   If the certificate contains the entire certificate chain, run the following command:

   ```bash
   notation sign --signature-format cose $IMAGE --id $KEY_ID --plugin azure-kv 
   ```

   If the certificate does not contain the chain, use the `--plugin-config ca_certs=<ca_bundle_file>` parameter to pass the CA certificates in a PEM file to AKV plugin, run the following command:

   ```bash
   notation sign --signature-format cose $IMAGE --id $KEY_ID --plugin azure-kv --plugin-config ca_certs=<ca_bundle_file> 
   ```

   To authenticate with AKV, by default, the following credential types if enabled will be tried in order:
   - [Environment credential](/dotnet/api/azure.identity.environmentcredential)
   - [Workload identity credential](/dotnet/api/azure.identity.workloadidentitycredential)
   - [Managed identity credential](/dotnet/api/azure.identity.managedidentitycredential)
   - [Azure CLI credential](/dotnet/api/azure.identity.azureclicredential)
    
   If you want to specify a credential type, use an additional plugin configuration called `credential_type`. For example, you can explicitly set `credential_type` to `azurecli` for using Azure CLI credential, as demonstrated below:
    
   ```bash
   notation sign --signature-format cose --id $KEY_ID --plugin azure-kv --plugin-config credential_type=azurecli $IMAGE
   ```

   See below table for the values of `credential_type` for various credential types.

   | Credential type              | Value for `credential_type` |
   | ---------------------------- | -------------------------- |
   | Environment credential       | `environment`              |
   | Workload identity credential | `workloadid`               |
   | Managed identity credential  | `managedid`                |
   | Azure CLI credential         | `azurecli`                 |

1. View the graph of signed images and associated signatures. 

    ```bash
    notation ls $IMAGE
    ```

    In the following example of output, a signature of type `application/vnd.cncf.notary.signature` identified by digest `sha256:d7258166ca820f5ab7190247663464f2dcb149df4d1b6c4943dcaac59157de8e` is associated to the `$IMAGE`.

    ```
    myregistry.azurecr.io/net-monitor@sha256:17cc5dd7dfb8739e19e33e43680e43071f07497ed716814f3ac80bd4aac1b58f
    └── application/vnd.cncf.notary.signature
        └── sha256:d7258166ca820f5ab7190247663464f2dcb149df4d1b6c4943dcaac59157de8e
    ```

## Verify a container image with Notation CLI 

1. Add the root certificate to a named trust store for signature verification. If you do not have the root certificate, you can obtain it from your CA. The following example adds the root certificate `$ROOT_CERT` to the `$STORE_NAME` trust store. 

    ```bash
    STORE_TYPE="ca" 
    STORE_NAME="wabbit-networks.io" 
    notation cert add --type $STORE_TYPE --store $STORE_NAME $ROOT_CERT  
    ```

2. List the root certificate to confirm the `$ROOT_CERT` is added successfully.

    ```bash
    notation cert ls 
    ```

3. Configure trust policy before verification.

   Trust policies allow users to specify fine-tuned verification policies. Use the following command to configure trust policy.

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

    The above `trustpolicy.json` file defines one trust policy named `wabbit-networks-images`. This trust policy applies to all the artifacts stored in the `$REGISTRY/$REPO` repositories. The named trust store `$STORE_NAME` of type `$STORE_TYPE` contains the root certificates. It also assumes that the user trusts a specific identity with the X.509 subject `$CERT_SUBJECT`. For more details, see [Trust store and trust policy specification](https://github.com/notaryproject/notaryproject/blob/v1.0.0/specs/trust-store-trust-policy.md).

4. Use `notation policy` to import the trust policy configuration from `trustpolicy.json`. 

    ```bash
    notation policy import ./trustpolicy.json
    ```

5. Show the trust policy configuration to confirm its successful import.

    ```bash
    notation policy show
    ```
    
5. Use `notation verify` to verify the integrity of the image:

    ```bash
    notation verify $IMAGE
    ```

   Upon successful verification of the image using the trust policy, the sha256 digest of the verified image is returned in a successful output message. An example of output:

   `Successfully verified signature for myregistry.azurecr.io/net-monitor@sha256:17cc5dd7dfb8739e19e33e43680e43071f07497ed716814f3ac80bd4aac1b58f`

## Timestamping

Since Notation v1.2.0 release, Notation supports [RFC 3161](https://www.rfc-editor.org/rfc/rfc3161) compliant timestamping. This enhancement extends the trust of signatures created within the certificate's validity period by trusting a Timestamping Authority (TSA), enabling successful signature verification even after the certificates have expired. As an image signer, you should ensure that you sign container images with timestamps generated by a trusted TSA. As an image verifier, to verify timestamps, you should ensure that you trust both the image signer and the associated TSA, and establish trust through trust stores and trust policies. Timestamping reduces costs by eliminating the need to periodically re-sign images due to certificate expiry, which is especially critical when using short-lived certificates. For detailed instructions on how to sign and verify using timestamping, refer to the [Notary Project timestamping guide](https://v1-2.notaryproject.dev/docs/user-guides/how-to/timestamping/).

## FAQ

- What should I do if the certificate is expired?

  If your certificate has expired, you need to obtain a new one from a trusted CA vendor along with a new private key. An expired certificate cannot be used to sign container images. For images that were signed before the certificate expired, they may still be validated successfully if they were signed with [timestamping](#timestamping). Without timestamping, the signature verification will fail, and you will need to re-sign those images with the new certificate for successful verification.

- What should I do if the certificate is revoked?
  
  If your certificate is revoked, it invalidates the signature. This can happen for several reasons, such as the private key being compromised or changes in the certificate holder's affiliation. To resolve this issue, you should first ensure your source code and build environment are up-to-date and secure. Then, build container images from the source code, obtain a new certificate from a trusted CA vendor along with a new private key, and sign new container images with the new certificate by following this guide.

## Next steps

Notation provides CI/CD solutions on Azure Pipelines and GitHub Actions:

- To sign and verify container images in ADO pipelines, see [Sign and verify a container image with Notation in Azure Pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign)
- To sign container images using GitHub Actions, see [Sign a container image with Notation using GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha)
- To verify container images using GitHub Actions, see [Verify a container image with Notation using GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha)

To ensure only trusted container images are deployed on Azure Kubernetes Service (AKS):
- Use Azure Policy Image Integrity (Preview) by following the guide [Use Image Integrity to validate signed images before deploying them to your Azure Kubernetes Service (AKS) clusters (Preview)](/azure/aks/image-integrity?tabs=azure-cli)
- Use [Ratify](https://ratify.dev/) and Azure Policy by following the guide [Securing AKS workloads: Validating container image signatures with Ratify and Azure Policy](/azure/security/container-secure-supply-chain/articles/validating-image-signatures-using-ratify-aks)

[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
