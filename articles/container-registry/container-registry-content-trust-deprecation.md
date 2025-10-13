--- 
title: Transition from Docker Content Trust to the Notary Project
description: Learn how to transition from Docker Content Trust to the Notary Project to sign and verify container images.
ms.topic: how-to 
ms.date: 02/07/2024 
ms.author: rayoflores
author: rayoef
ms.service: azure-container-registry
# Customer intent: As a container image publisher, I want to transition from Docker Content Trust to the Notary Project, so that I can ensure the integrity and authenticity of my container images in a secure and standardized manner.
--- 

# Transition from Docker Content Trust to the Notary Project

Azure Container Registry will retire [Docker Content Trust](./container-registry-content-trust.md) on March 31, 2028. To help with this transition, we're providing guidance for how to disable Docker Content Trust and adopt the Notary Project to sign and verify container images.

## Docker Content Trust deprecation

Docker Content Trust allows image publishers to sign their images and image consumers to verify that the images they pull are signed. With advancements in technology, Docker Content Trust no longer meets the requirements of modern supply chain security for containers. As a result, Docker Content Trust will be deprecated starting from March 31, 2025 and completely removed from Azure Container Registry (ACR) on March 31, 2028.

Instead of using Docker Content Trust, Microsoft offers signing and verification solutions based on the Notary Project. The [Notary Project](https://notaryproject.dev/) is a set of specifications and tools intended to provide a cross-industry standard for securing software supply chains by using authentic container images and other OCI artifacts. [Notation](https://github.com/notaryproject/notation), a tool from the Notary Project, implements Notary Project specifications and includes CLI and libraries for signing and verifying container images and artifacts. The benefits of using the Notary Project solutions to ensure the integrity and authenticity of container images include:

-	**Portability and Interoperability**: [Notary Project signatures](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/signature-specification.md) adhere to [Open Container Initiative (OCI) standards](https://github.com/opencontainers/image-spec/tree/v1.1.0) and can be stored in OCI-compliant registries, such as ACR, facilitating signature portability and interoperability across different cloud environments.
-	**Secure Key Management**: Manage your signing keys and certificates securely with [Azure Key Vault (AKV)](/azure/key-vault/general/basic-concepts). More Key Management System (KMS) options are coming soon.
-	**CI/CD Pipeline Integration**: Implement signing in your CI/CD pipelines, including Azure DevOps (ADO) and GitHub workflows. More CI/CD integration options are coming soon.
-	**Comprehensive Verification**: Verify container images within your CI/CD pipelines, such as ADO and GitHub workflows, and on [Azure Kubernetes Service (AKS)](/azure/aks/) to prevent the use and deployment of untrusted images.

## Disable Docker Content Trust

Before you can transition to the Notation Project solutions, you have to disable Docker Content Trust. Use any of the following methods to disable Docker Content Trust:

- Disable Docker Content Trust from the shell by setting the `DOCKER_CONTENT_TRUST` environment variable to `0`. For example, in the Bash shell:

   ```bash
   export DOCKER_CONTENT_TRUST=0
   ```

   Alternatively, you can unset the environment variable:

   ```bash
   unset DOCKER_CONTENT_TRUST
   ```

- Disable Docker Content Trust from Azure portal. Navigate to the registry in the Azure portal. Under **Policies**, select **Content Trust**, then choose **Disabled** and select **Save**.

- Disable Docker Content Trust using the Azure CLI:

   ```
   az acr config content-trust update -r myregistry --status disabled
   ```

## Use the Notary Project to sign and verify container images

After you disable Docker Content Trust, you can sign and verify container images with the Notary Project. Use the following references to get started.

Sign container images:

- To use AKV with self-signed certificates, see [Sign container images with Notation and Azure Key Vault using a self-signed certificate](./container-registry-tutorial-sign-build-push.md)
- To use AKV with CA-issued certificates, see [Sign Container Images with Notation and a CA-issued certificate Azure Key Vault](./container-registry-tutorial-sign-trusted-ca.md)
- To sign in ADO pipelines, see [Sign and verify a container image with Notation in Azure Pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign)
- To sign in GitHub workflows, see [Sign a container image with Notation using a GitHub Workflows](/azure/security/container-secure-supply-chain/articles/notation-sign-gha)

Verify container images:

- To verify in ADO pipelines, see [Sign and verify a container image with Notation in Azure Pipeline](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign#verify-the-signed-image)
- To verify in GitHub workflows, see [Verify a container image with Notation using GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha)
- To verify on AKS, see [Securing AKS workloads: Validating container image signatures with Ratify and Azure Policy](/azure/security/container-secure-supply-chain/articles/validating-image-signatures-using-ratify-aks)

## Related content

- [Notary Project Terms](https://notaryproject.dev/docs/faq/#notary-project-terms)
- [Containers Secure Supply Chain Framework](https://aka.ms/csscframework)
