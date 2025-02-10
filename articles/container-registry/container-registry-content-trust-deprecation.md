--- 
title: Transition from Docker Content Trust to the Notary Project for signing and verifying container images
description: Learn how to Transition from Docker Content Trust to the Notary Project for signing and verifying container images.
ms.topic: whats-new #Don't change. 
ms.date: 02/07/2024 
ms.author: yizha1
author: yizha1
ms.service: azure-container-registry
--- 

# Transition from Docker Content Trust to the Notary Project for signing and verifying container images

Azure Container Registry will be deprecating [Docker Content Trust](./container-registry-content-trust.md). To help with this transition, we will provide guidance on disabling Docker Content Trust and adopting the Notary Project for signing and verifying container images.

## Docker Content Trust Deprecation

Docker Content Trust allows image publishers to sign their images and image consumers to verify that the images they pull are signed. With advancements in technology, Docker Content Trust no longer meets the requirements of modern supply chain security for containers. As a result, Docker Content Trust will be deprecated and not available in ACR.

The [Notary Project](https://notaryproject.dev/) is a set of specifications and tools intended to provide a cross-industry standard for securing software supply chains by using authentic container images and other OCI artifacts. [Notation](https://github.com/notaryproject/notation), a tool from the Notary Project, implements Notary Project specifications and includes CLI and libraries for signing and verifying container images and artifacts. Microsoft now offers signing and verification solutions based on the Notary Project to help ACR customers to ensure the integrity and authenticity of container images:

-	**Portability and Interoperability**: [Notary Project signatures](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/signature-specification.md) adhere to [Open Container Initiative (OCI) standards](https://github.com/opencontainers/image-spec/tree/v1.1.0) and can be stored in OCI-compliant registries, such as ACR, facilitating signature portability and interoperability across different cloud environments.
-	**Secure Key Management**: Manage your signing keys and certificates securely with [Azure Key Vault (AKV)](/azure/key-vault/general/basic-concepts), with more Key Management System (KMS) options coming soon.
-	**CI/CD Pipeline Integration**: Implement signing in your CI/CD pipelines, including Azure DevOps (ADO) and GitHub workflows, with more options coming soon.
-	**Comprehensive Verification**: Verify container images within your CI/CD pipelines, such as ADO and GitHub workflows, and on [Azure Kubernetes Service (AKS)](/azure/aks/) to prevent the use and deployment of untrusted images.

## Disable Docker Content Trust

There are several ways to disable Docker Content Trust:

- Disable Docker Content Trust from the shell by setting the `DOCKER_CONTENT_TRUST` environment variable to `0`. For example, in the Bash shell:

   ```bash
   export DOCKER_CONTENT_TRUST=0
   ```

   Alternatively, you can unset the environment variable:

   ```bash
   unset DOCKER_CONTENT_TRUST
   ```

- Disable Docker Content Trust from Azure portal. Navigate to the registry in the Azure portal. Under `Policies`, select `Content Trust`, then choose `Disabled` and click `Save`.

- Disable Docker Content Trust using Azure CLI:

   ```
   az acr config content-trust update -r myregistry --status disabled
   ```

## Use the Notary Project for signing and verifying container images

Here are the references to help you get started.

Signing container images:

- Using AKV with self-signed certificates, see [Sign container images with Notation and Azure Key Vault using a self-signed certificate](./container-registry-tutorial-sign-build-push.md)
- Using AKV with CA-issued certificates, see [Sign Container Images with Notation and a CA-issued certificate Azure Key Vault](./container-registry-tutorial-sign-trusted-ca.md)
- Signing in ADO pipelines, see [Sign and verify a container image with Notation in Azure Pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign)
- Signing in GitHub workflows, see [Sign a container image with Notation using a GitHub Workflows](/azure/security/container-secure-supply-chain/articles/notation-sign-gha)

Verifying container images:

- Verification in ADO pipelines, see [Sign and verify a container image with Notation in Azure Pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign#verify-the-signed-image)
- Verification in GitHub workflows, see [Verify a container image with Notation using a GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha)
- Verification on AKS, see [Securing AKS workloads: Validating container image signatures with Ratify and Azure Policy](/azure/security/container-secure-supply-chain/articles/validating-image-signatures-using-ratify-aks)

## Addtional references

- [Containers Secure Supply Chain Framework](https://aka.ms/csscframework)
- [Notary Project terms](https://notaryproject.dev/docs/faq/#notary-project-terms)