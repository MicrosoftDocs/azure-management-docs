--- 
title: Transition from Docker Content Trust to the Notary Project
description: Learn how to transition from Docker Content Trust to the Notary Project to sign and verify container images.
ms.topic: how-to 
ms.date: 02/07/2024 
ms.author: doveychase
author: chasedmicrosoft
ms.service: azure-container-registry
# Customer intent: As a container image publisher, I want to transition from Docker Content Trust to the Notary Project so that I can ensure the integrity and authenticity of my container images in a secure and standardized manner.
--- 

# Transition from Docker Content Trust to the Notary Project

Azure Container Registry will retire [Docker Content Trust (DCT)](./container-registry-content-trust.md) on March 31, 2028. To help with this transition, we're providing guidance for how to disable DCT and adopt the Notary Project to sign and verify container images.

## DCT deprecation

DCT allows image publishers to sign their images and image consumers to verify that the images they pull are signed. With advancements in technology, DCT no longer meets the requirements of modern supply chain security for containers. As a result, deprecation of DCT started on March 31, 2025. DCT will be completely removed from Azure Container Registry on March 31, 2028.

As an alternative to DCT, Microsoft offers signing and verification solutions based on the Notary Project. The [Notary Project](https://notaryproject.dev/) is a set of specifications and tools intended to provide a cross-industry standard for securing software supply chains by using authentic container images and other Open Container Initiative (OCI) artifacts.

[Notation](https://github.com/notaryproject/notation), a tool from the Notary Project, implements Notary Project specifications. It includes a command-line interface (CLI) and libraries for signing and verifying container images and artifacts. The benefits of using the Notary Project solutions to ensure the integrity and authenticity of container images include:

- **Portability and interoperability**: [Notary Project signatures](https://github.com/notaryproject/specifications/blob/v1.1.0/specs/signature-specification.md) adhere to [OCI standards](https://github.com/opencontainers/image-spec/tree/v1.1.0) and can be stored in OCI-compliant registries like Container Registry. These capabilities facilitate signature portability and interoperability across cloud environments.
- **Secure key management**: You can use [Azure Key Vault](/azure/key-vault/general/basic-concepts) to manage your signing keys and certificates.
- **Integraton with continuous integration and continuous delivery (CI/CD)CI/CD pipelines**: Implement signing in your CI/CD pipelines, including Azure DevOps and GitHub workflows.
- **Comprehensive verification**: Verify container images within your CI/CD pipelines (such as Azure DevOps and GitHub workflows) and on [Azure Kubernetes Service (AKS)](/azure/aks/) to prevent the use and deployment of untrusted images.

## Disable DCT

Before you can transition to the Notation Project solutions, you have to disable DCT. Use any of the following methods to disable DCT:

- Disable DCT from the shell by setting the `DOCKER_CONTENT_TRUST` environment variable to `0`. For example, in the Bash shell, use this command:

  ```bash
  export DOCKER_CONTENT_TRUST=0
  ```

  Alternatively, you can unset the environment variable:

  ```bash
  unset DOCKER_CONTENT_TRUST
  ```

- Disable DCT from the Azure portal. Go to the registry, and then under **Policies**, select **Content Trust** > **Disabled** > **Save**.

- Disable DCT by using the Azure CLI:

   ```
   az acr config content-trust update -r myregistry --status disabled
   ```

## Use the Notary Project to sign and verify container images

After you disable DCT, you can sign and verify container images by using the Notary Project. Use the following references to get started.

Sign container images:

- To use Key Vault with self-signed certificates, see [Sign container images by using Notation, Azure Key Vault, and a self-signed certificate](./container-registry-tutorial-sign-build-push.md).
- To use Key Vault with certificates issued by a certificate authority (CA), see [Sign container images by using Notation, Azure Key Vault, and a CA-issued certificate](./container-registry-tutorial-sign-trusted-ca.md).
- To sign in ADO pipelines, see [Sign and verify a container image by using Notation in Azure Pipelines](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign).
- To sign in GitHub workflows, see [Sign a container image by using Notation and GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha).

Verify container images:

- To verify in ADO pipelines, see [Sign and verify a container image by using Notation in Azure Pipelines](/azure/security/container-secure-supply-chain/articles/notation-ado-task-sign#verify-the-signed-image).
- To verify in GitHub workflows, see [Verify a container image by using Notation and GitHub Actions](/azure/security/container-secure-supply-chain/articles/verify-gha).
- To verify on AKS, see [Verify container image signatures by using Ratify and Azure Policy](./container-registry-tutorial-verify-with-ratify-aks.md).

## Related content

- [Notary Project Terms](https://notaryproject.dev/docs/faq/#notary-project-terms)
- [Containers Secure Supply Chain Framework](https://aka.ms/csscframework)
