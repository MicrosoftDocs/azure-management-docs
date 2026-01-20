---
title: Overview of Signing and Verifying OCI Artifacts
description: Learn how to sign and verify container images and other OCI artifacts to help ensure integrity and authenticity across the software supply chain.
author: yizha1
ms.author: yizha1
ms.service: azure-container-registry
ms.topic: overview
ms.date: 09/05/2025
# Customer intent: As a developer, I want to sign and verify artifacts in registries so that I can help ensure the authenticity and integrity of artifacts throughout their lifecycle.
---

# Overview of signing and verifying OCI artifacts

Container images and other Open Container Initiative (OCI) artifacts are key components of modern cloud-native applications. These artifacts can include software bills of materials (SBOMs), Helm charts, configuration bundles, and AI models. They flow through the software supply chain from creation to deployment. They're built by publishers, stored in registries, and consumed in continuous integration and continuous delivery (CI/CD) pipelines or production environments.

Without safeguards, attackers could:

- Alter artifacts while they're stored or transferred.
- Swap trusted images or charts with malicious ones.
- Insert unverified base images or dependencies into builds.

Adding safeguards helps ensure:

- **Integrity**: The artifact that you use is exactly the same as the one that was published.
- **Authenticity**: The artifact truly came from the expected publisher.

Together, integrity and authenticity are critical for protecting the supply chain and preventing attacks.

## Signing and verification

These processes help provide integrity and authenticity:

- **Signing**: Produces cryptographic signatures that bind a publisher's identity to an artifact descriptor, including the digest. OCI artifacts like container images can be signed after they're built.
- **Verification**: Checks that a signature is valid, the publisher's identity is trusted, and the artifact isn't altered. Signatures can be verified before artifacts are consumed.

If verification fails, consumers can block the artifact from being pulled, used in builds, or deployed.

## Notary Project and Notation

[Notary Project](https://notaryproject.dev/) is an open-source project that provides signing and verification for OCI artifacts.

*Notation* is the Notary Project tooling for signing and verifying OCI artifacts. Notation integrates with multiple key providers, including:

- **Azure Key Vault**: Users manage their own certificate lifecycle, including issuance, rotation, and expiration. This choice provides strong control and flexibility for organizations that want to maintain direct management of their certificates.
- **Artifact Signing**: Provides *zero-touch certificate management* and issues *short-lived certificates* automatically. These features simplify the signing experience while maintaining strong security guarantees.

Organizations can choose Key Vault for full control or adopt Artifact Signing for a streamlined experience.

## Scenarios for signing and verifying

### Image publisher signs images in CI/CD pipelines such as GitHub Actions

A container image publisher builds an image and signs it as part of the GitHub Actions workflow before pushing it to Azure Container Registry. This process:

- Helps ensure that downstream consumers can verify the image origin.
- Adds trust metadata into the supply chain at build time.

### Image consumer verifies images during deployment on AKS

When an organization deploys workloads to Azure Kubernetes Service (AKS), a cluster policy can enforce that only signed and verified images are allowed to run. This process:

- Helps prevent the deployment of unauthorized or tampered-with images.
- Helps ensure that runtime workloads originate from trusted publishers.

### Image consumer verifies base images in CI/CD pipelines

Before developers build an application image, they can configure pipelines (for example, GitHub Actions) to verify the signatures of base images. This process:

- Helps protect builds from inheriting vulnerabilities or malicious code.
- Enforces the use of only trusted upstream components in application builds.

### Consumer verifies other OCI artifacts

Beyond container images, consumers can verify these artifacts stored in an OCI registry:

- **SBOMs**: Verify the signed SBOM of an image before using it for security analysis.
- **Helm charts**: Verify charts before installing them in Kubernetes clusters.
- **Configuration bundles or AI models**: Verify that these bundles or models originate from the intended publisher before integrating them into systems.

## Related content

This overview introduces the importance of signing and verifying container images and other OCI artifacts. Each of the following scenarios has its own dedicated guidance.

### Signing with Key Vault

Signing via the Notation command-line interface (CLI):

- [Sign container images by using Notation, Azure Key Vault, and a self-signed certificate](container-registry-tutorial-sign-build-push.md)
- [Sign container images by using Notation, Azure Key Vault, and a CA-issued certificate](container-registry-tutorial-sign-trusted-ca.md)

Signing in a GitHub workflow:

- [Sign a container image by using Notation in GitHub Actions](/azure/security/container-secure-supply-chain/articles/notation-sign-gha)

### Signing with Artifact Signing

Signing via the Notation CLI:

- [Sign container images by using Notation and Artifact Signing](container-registry-tutorial-sign-verify-notation-trusted-signing.md)

Signing in a GitHub workflow:

- [Sign container images in a GitHub workflow by using Notation and Artifact Signing](container-registry-tutorial-github-sign-notation-trusted-signing.md)

### Verification

Verification in a GitHub workflow:

- [Verify container images in a GitHub workflow by using Notation and Azure Key Vault](/azure/security/container-secure-supply-chain/articles/verify-gha)
- [Verify container images in a GitHub workflow by using Notation and Artifact Signing](container-registry-tutorial-github-verify-notation-trusted-signing.md)

Verification on AKS:

- [Verify container image signatures by using Ratify and Azure Policy](container-registry-tutorial-verify-with-ratify-aks.md)
