---
title: Sign Container Images with Notation and Azure Key Vault by Using a Self-Signed Certificate
description: Learn how to create a self-signed certificate in Azure Key Vault, build and sign a container image in Azure Container Registry, and verify it by using Notation.
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 9/3/2024
# Customer intent: As a developer, I want to sign and verify container images by using a self-signed certificate and secure storage solutions, so that I can ensure the authenticity and integrity of my container images throughout their lifecycle.
---

# Sign container images by using Notation, Azure Key Vault, and a self-signed certificate

This article is part of a series on ensuring the integrity and authenticity of container images and other Open Container Initiative (OCI) artifacts.
For the complete picture, start with the [overview](overview-sign-verify-artifacts.md), which explains why signing matters and outlines the various scenarios.

Signing container images is a process that helps ensure their authenticity and integrity. A digital signature that's added to a container image is verified during deployment. The signature helps to verify that the image is from a trusted publisher and isn't modified.

This article discusses the following tools involved in the signing process:

- [Notation](https://github.com/notaryproject/notation) is an open-source supply-chain security tool developed by the [Notary Project community](https://notaryproject.dev/) and backed by Microsoft. It supports signing and verifying container images and other artifacts.

  If you want to sign a container image by using Notation in continuous integration and continuous delivery (CI/CD) pipelines, follow the guidance for [Azure Pipelines](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign) or [GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha).
- Azure Key Vault is a service for storing certificates with signing keys. Notation can use these keys via the Key Vault plug-in (`notation-azure-kv`) to sign and verify container images and other artifacts.
- Azure Container Registry is a private registry that you can use to attach signatures to container images and other artifacts, along with viewing those signatures.

In this article, you learn how to:

- Install the Notation command-line interface (CLI) and the Key Vault plug-in.
- Create a self-signed certificate in Key Vault.
- Build and push a container image by using [Container Registry tasks](container-registry-tasks-overview.md).
- Sign a container image by using the Notation CLI and the Key Vault plug-in.
- Validate a container image against the signature by using the Notation CLI.
- Use timestamping.

## Prerequisites

- Create or use a [container registry](../container-registry/container-registry-get-started-azure-cli.md) for storing container images and signatures.
- Create or use a [key vault](/azure/key-vault/general/quick-create-cli) for managing certificates.
- Install and configure the latest [Azure CLI](/cli/azure/install-azure-cli) version, or run commands in [Azure Cloud Shell](https://portal.azure.com/#cloudshell/).

## Install the Notation CLI and Key Vault plug-in

1. Install Notation v1.3.2 in a Linux AMD64 environment. To download the package for other environments, follow the [Notation installation guide](https://notaryproject.dev/docs/user-guides/installation/cli/).

    ```bash
    # Download, extract, and install
    curl -Lo notation.tar.gz https://github.com/notaryproject/notation/releases/download/v1.3.2/notation_1.3.2_linux_amd64.tar.gz
    tar xvzf notation.tar.gz
            
    # Copy the Notation binary to the desired bin directory in $PATH, for example
    cp ./notation /usr/local/bin
    ```

2. Install Key Vault plug-in (`notation-azure-kv`) v1.2.1 in a Linux AMD64 environment.

    > [!NOTE]
    > You can find the URL and SHA256 checksum for the plug-in on the plug-in's [release page](https://github.com/Azure/notation-azure-kv/releases).

    ```bash
    notation plugin install --url https://github.com/Azure/notation-azure-kv/releases/download/v1.2.1/notation-azure-kv_1.2.1_linux_amd64.tar.gz --sha256sum 67c5ccaaf28dd44d2b6572684d84e344a02c2258af1d65ead3910b3156d3eaf5
    ```

3. List the available plug-ins and confirm that the `notation-azure-kv` plug-in with version `1.2.1` is included in the list:

    ```bash
    notation plugin ls
    ```

## Configure environment variables

For easy execution of commands in this article, provide values for the Azure resources to match the existing Container Registry and Key Vault resources.

1. Configure Key Vault resource names:

    ```bash
    AKV_SUB_ID=myAkvSubscriptionId
    AKV_RG=myAkvResourceGroup
    # Name of the existing key vault used to store the signing keys
    AKV_NAME=myakv
    # Name of the certificate created in the key vault
    CERT_NAME=wabbit-networks-io
    CERT_SUBJECT="CN=wabbit-networks.io,O=Notation,L=Seattle,ST=WA,C=US"
    CERT_PATH=./${CERT_NAME}.pem
    ```

2. Configure Container Registry and image resource names:

    ```bash
    ACR_SUB_ID=myAcrSubscriptionId
    ACR_RG=myAcrResourceGroup
    # Name of the existing registry (example: myregistry.azurecr.io)
    ACR_NAME=myregistry
    # Existing full domain of the container registry
    REGISTRY=$ACR_NAME.azurecr.io
    # Container name inside the container registry where the image will be stored
    REPO=net-monitor
    TAG=v1
    IMAGE=$REGISTRY/${REPO}:$TAG
    # Source code directory that contains the Dockerfile to build
    IMAGE_SOURCE=https://github.com/wabbit-networks/net-monitor.git#main
    ```

## Sign in by using the Azure CLI

```bash
az login
```

For more information, see [Authenticate to Azure by using the Azure CLI](/cli/azure/authenticate-azure-cli).

## Grant access permissions to Container Registry and Key Vault

When you're working with Container Registry and Key Vault, it's essential to grant the appropriate permissions to help ensure secure and controlled access. You can authorize access for various entities, such as user principals, service principals, or managed identities, depending on your specific scenarios. In this article, the access is authorized for a signed-in Azure user.

### Authorize access to Container Registry

For registries enabled for Microsoft Entra attribute-based access control (ABAC), the `Container Registry Repository Reader` and `Container Registry Repository Writer` roles are required for building and signing container images in Container Registry.

For registries not enabled for ABAC, the `AcrPull` and `AcrPush` roles are required.

For more information on ABAC, see [Microsoft Entra attribute-based access control for repository permissions (preview)](container-registry-rbac-abac-repository-permissions.md).

1. Set the subscription that contains the Container Registry resource:

    ```bash
    az account set --subscription $ACR_SUB_ID
    ```

2. Assign the roles. The correct role to use in the role assignment depends on whether the registry is [ABAC enabled or not](container-registry-rbac-abac-repository-permissions.md).

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    ROLE1="Container Registry Repository Reader" # For ABAC-enabled registries. Otherwise, use "AcrPull" for non-ABAC-enabled registries.
    ROLE2="Container Registry Repository Writer" # For ABAC-enabled registries. Otherwise, use "AcrPush" for non-ABAC-enabled registries.
    az role assignment create --role "$ROLE1" --role "$ROLE2" --assignee $USER_ID --scope "/subscriptions/$ACR_SUB_ID/resourceGroups/$ACR_RG/providers/Microsoft.ContainerRegistry/registries/$ACR_NAME"
    ```

### Authorize access to Key Vault

This section explores two options for authorizing access to Key Vault.

#### Use Azure RBAC (recommended)

The following roles are required for signing by using self-signed certificates:

- `Key Vault Certificates Officer` for creating and reading certificates
- `Key Vault Certificates User` for reading existing certificates
- `Key Vault Crypto User` for signing operations

To learn more about Key Vault access with Azure role-based access control (RBAC), see [Provide access to Key Vault keys, certificates, and secrets by using Azure role-based access control](/azure/key-vault/general/rbac-guide).

1. Set the subscription that contains the Key Vault resource:

    ```bash
    az account set --subscription $AKV_SUB_ID
    ```

2. Assign the roles:

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az role assignment create --role "Key Vault Certificates Officer" --role "Key Vault Crypto User" --assignee $USER_ID --scope "/subscriptions/$AKV_SUB_ID/resourceGroups/$AKV_RG/providers/Microsoft.KeyVault/vaults/$AKV_NAME"
    ```

#### Assign an access policy in Key Vault (legacy)

The following permissions are required for an identity:

- `Create` permissions for creating a certificate
- `Get` permissions for reading existing certificates
- `Sign` permissions for signing operations

To learn more about assigning a policy to a principal, see [Assign a Key Vault access policy (legacy)](/azure/key-vault/general/assign-access-policy).

1. Set the subscription that contains the Key Vault resource:

    ```bash
    az account set --subscription $AKV_SUB_ID
    ```

2. Set the access policy in Key Vault:

    ```bash
    USER_ID=$(az ad signed-in-user show --query id -o tsv)
    az keyvault set-policy -n $AKV_NAME --certificate-permissions create get --key-permissions sign --object-id $USER_ID
    ```

> [!IMPORTANT]
> This example shows the minimum permissions that you need for creating a certificate and signing a container image. Depending on your requirements, you might need to grant more permissions.

## Create a self-signed certificate in Key Vault (Azure CLI)

The following steps show how to create a self-signed certificate for testing purposes:

1. Create a certificate policy file.

    After the certificate policy file is executed via the following code, it creates a valid certificate compatible with the [Notary Project certificate requirements](https://github.com/notaryproject/specifications/blob/v1.0.0/specs/signature-specification.md#certificate-requirements) in Key Vault. The value for `ekus` is for code signing, but it isn't required for Notation to sign artifacts. The subject is used later as a trusted identity during verification.

    ```bash
    cat <<EOF > ./my_policy.json
    {
        "issuerParameters": {
        "certificateTransparency": null,
        "name": "Self"
        },
        "keyProperties": {
          "exportable": false,
          "keySize": 2048,
          "keyType": "RSA",
          "reuseKey": true
        },
        "secretProperties": {
          "contentType": "application/x-pem-file"
        },
        "x509CertificateProperties": {
        "ekus": [
            "1.3.6.1.5.5.7.3.3"
        ],
        "keyUsage": [
            "digitalSignature"
        ],
        "subject": "$CERT_SUBJECT",
        "validityInMonths": 12
        }
    }
    EOF
    ```

2. Create the certificate:

    ```bash
    az keyvault certificate create -n $CERT_NAME --vault-name $AKV_NAME -p @my_policy.json
    ```

## Sign a container image by using the Notation CLI and Key Vault plug-in

1. Authenticate to your container registry by using your individual Azure identity:

    ```bash
    az acr login --name $ACR_NAME
    ```

   > [!IMPORTANT]
   > If you have Docker installed on your system and you used `az acr login` or `docker login` to authenticate to your container registry, your credentials are already stored and available to Notation. In this case, you don't need to run `notation login` again to authenticate to your container registry. To learn more about authentication options for Notation, see [Authenticate with OCI-compliant registries](https://notaryproject.dev/docs/user-guides/how-to/registry-authentication/).

2. Build and push a new image by using Azure Container Registry tasks. Always use the digest value to identify the image for signing, because tags are mutable and can be overwritten.

    ```bash
    DIGEST=$(az acr build -r $ACR_NAME -t $REGISTRY/${REPO}:$TAG $IMAGE_SOURCE --no-logs --query "outputImages[0].digest" -o tsv)
    IMAGE=$REGISTRY/${REPO}@$DIGEST
    ```

    In this article, if the image is already built and is stored in the registry, the tag serves as an identifier for that image for convenience:

    ```bash
    IMAGE=$REGISTRY/${REPO}:$TAG
    ```

3. Get the ID of the signing key. A certificate in Key Vault can have multiple versions. The following command gets the key ID of the latest version:

    ```bash
    KEY_ID=$(az keyvault certificate show -n $CERT_NAME --vault-name $AKV_NAME --query 'kid' -o tsv)
    ```

4. Sign the container image with the [CBOR Object Signing and Encryption (COSE)](https://datatracker.ietf.org/doc/html/rfc9052) signature format, by using the signing key ID. To sign with a self-signed certificate, you need to set the plug-in configuration value `self_signed=true`.

    ```bash
    notation sign --signature-format cose --id $KEY_ID --plugin azure-kv --plugin-config self_signed=true $IMAGE
    ```

    To authenticate with Key Vault, by default, the following credential types (if enabled) are tried in order:

    - [Environment credential](/dotnet/api/azure.identity.environmentcredential)
    - [Workload identity credential](/dotnet/api/azure.identity.workloadidentitycredential)
    - [Managed identity credential](/dotnet/api/azure.identity.managedidentitycredential)
    - [Azure CLI credential](/dotnet/api/azure.identity.azureclicredential)

    If you want to specify a credential type, use an additional plug-in configuration called `credential_type`. For example, you can explicitly set `credential_type` to `azurecli` for using an Azure CLI credential, as demonstrated in this example:

    ```bash
    notation sign --signature-format cose --id $KEY_ID --plugin azure-kv --plugin-config self_signed=true --plugin-config credential_type=azurecli $IMAGE
    ```

    The following table shows the values of `credential_type` for various credential types.

    | Credential type              | Value for `credential_type` |
    | ---------------------------- | -------------------------- |
    | Environment credential       | `environment`              |
    | Workload identity credential | `workloadid`               |
    | Managed identity credential  | `managedid`                |
    | Azure CLI credential         | `azurecli`                 |

    > [!NOTE]
    > Since Notation v1.2.0, Notation uses the [OCI referrers tag schema](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#referrers-tag-schema) to store the signature in Container Registry by default. You can also enable the [OCI Referrers API](https://github.com/opencontainers/distribution-spec/blob/main/spec.md#listing-referrers) by using the flag `--force-referrers-tag false`, if necessary. Container Registry features support the OCI Referrers API, except for the registry encrypted via customer-managed keys (CMKs).

5. View the graph of signed images and associated signatures:

   ```bash
   notation ls $IMAGE
   ```

## Verify a container image by using the Notation CLI

To verify the container image, add the root certificate that signs the leaf certificate to the trust store, and create trust policies for verification. For the self-signed certificate that this article uses, the root certificate is the self-signed certificate itself.

1. Download a public certificate:

    ```bash
    az keyvault certificate download --name $CERT_NAME --vault-name $AKV_NAME --file $CERT_PATH
    ```

2. Add the downloaded public certificate to named trust store for signature verification:

   ```bash
   STORE_TYPE="ca"
   STORE_NAME="wabbit-networks.io"
   notation cert add --type $STORE_TYPE --store $STORE_NAME $CERT_PATH
   ```

3. List the certificate to confirm:

   ```bash
   notation cert ls
   ```

4. Configure a trust policy before verification.

   Trust policies enable users to specify fine-tuned verification policies. The following example configures a trust policy named `wabbit-networks-images`. This policy applies to all artifacts in `$REGISTRY/$REPO` and uses the named trust store `$STORE_NAME` of type `$STORE_TYPE`. It also assumes that the user trusts a specific identity with the X.509 subject `$CERT_SUBJECT`. For more information, see [Trust store and trust policy specification](https://github.com/notaryproject/notaryproject/blob/v1.0.0/specs/trust-store-trust-policy.md).

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

5. Use `notation policy` to import the trust policy configuration from the JSON file that you created previously:

    ```bash
    notation policy import ./trustpolicy.json
    notation policy show
    ```

6. Use `notation verify` to verify that the container image wasn't altered after build time:

    ```bash
    notation verify $IMAGE
    ```

   Upon successful verification of the image via the trust policy, the SHA256 digest of the verified image is returned in a successful output message.

## Use timestamping

Since the Notation v1.2.0 release, Notation supports [RFC 3161](https://www.rfc-editor.org/rfc/rfc3161)-compliant timestamping. This enhancement extends the trust of signatures created within the certificate's validity period by trusting a time stamp authority (TSA). This trust enables successful signature verification even after the certificates expire.

As an image signer, you should ensure that you sign container images with time stamps that a trusted TSA generated. As an image verifier, you should ensure that you trust both the image signer and the associated TSA, and establish trust through trust stores and trust policies.

Timestamping reduces costs by eliminating the need to periodically re-sign images due to certificate expiry. This ability is especially critical when you use short-lived certificates. For detailed instructions on how to sign and verify images by using timestamping, refer to the [Notary Project timestamping guide](https://v1-2.notaryproject.dev/docs/user-guides/how-to/timestamping/).

## Related content

Notation provides CI/CD solutions on Azure Pipelines and GitHub Actions:

- To sign and verify container images in Azure DevOps pipelines, see [Sign and verify a container image by using Notation in an Azure pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign).
- To sign container images by using GitHub Actions, see [Sign a container image by using Notation in GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha).
- To verify container images by using GitHub Actions, see [Verify a container image by using Notation in GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha).

To ensure that only trusted container images are deployed on Azure Kubernetes Service (AKS):

- Use Azure Policy Image Integrity (preview) by following the guide [Use Image Integrity to validate signed images before deploying them to your Azure Kubernetes Service clusters (preview)](/azure/aks/image-integrity?tabs=azure-cli).
- Use [Ratify](https://ratify.dev/) and Azure Policy by following the guide [Verify container image signatures by using Ratify and Azure Policy](container-registry-tutorial-verify-with-ratify-aks.md).
