---
title: Overview of Ensuring Integrity and Authenticity for Container Images and OCI Artifacts
description: Learn how to secure container images and OCI artifacts by signing and verifying them to ensure integrity and authenticity across the software supply chain.
author: yizha1
ms.author: yizha1
ms.service: azure-container-registry
ms.topic: overview
ms.date: 09/05/2025
# Customer intent: As a developer, I want to sign and verify artifacts in registries, so that I can ensure the authenticity and integrity of artifacts throughout their lifecycle.
---

# Ensuring Integrity and Authenticity of Container Images and OCI Artifacts  

## Why integrity and authenticity matter

Container images and other OCI artifacts (such as SBOMs, Helm charts, configuration bundles, or AI models) are key components of modern cloud-native applications. These artifacts flow through the software supply chain, from creation to deployment: built by publishers, stored in registries, and consumed in CI/CD pipelines or production environments.  

Without safeguards, attackers could:  
- Alter artifacts while they're stored or transferred.  
- Swap trusted images or charts with malicious ones.  
- Insert unverified base images or dependencies into builds.  

**Integrity** ensures that the artifact you use is exactly the same as the one that was published.  
**Authenticity** ensures the artifact truly came from the expected publisher.  

Together, they are critical for protecting the supply chain and preventing attacks.  

## How signing and verification help

To provide integrity and authenticity, OCI artifacts (including container images) can be **signed** after they are built, and those signatures can be **verified** before artifacts are consumed.

- **Signing** produces cryptographic signatures that bind a publisher’s identity to an artifact descriptor including the digest.  
- **Verification** checks that the signature is valid, the publisher identity is trusted, and the artifact hasn’t been altered.  

If verification fails, consumers can block the artifact from being pulled, used in builds, or deployed.  

## Notary Project and Notation

[Notary Project](https://notaryproject.dev/) is an open-source project that provides signing and verification for OCI artifacts, including container images, SBOMs, Helm charts, AI models, and more.

**Notation** is the Notary Project tooling for signing and verifying OCI artifacts. Notation integrates with multiple key providers, including:  

- **Azure Key Vault (AKV):** Users manage their own certificate lifecycle, including issuance, rotation, and expiration. This provides strong control and flexibility for organizations that want to maintain direct management of their certificates.  
- **Trusted Signing:** Provides **zero-touch certificate management** and issues **short-lived certificates** automatically. This eliminates simplifies the signing experience while maintaining strong security guarantees.  

Organizations can choose AKV for full control or adopt Trusted Signing for a streamlined experience.  

## Scenarios for signing and verifying  

### 1. Image publisher signs images in CI/CD pipelines such as GitHub Actions

A container image publisher builds an image and signs it as part of their GitHub Actions workflow before pushing it to Azure Container Registry.  
- Ensures that downstream consumers can verify the image origin.  
- Adds trust metadata into the supply chain at build time.  

### 2. Image consumer verifies during deployment on AKS

When deploying workloads to Azure Kubernetes Service (AKS), a cluster policy can enforce that only signed and verified images are allowed to run.  
- Prevents unauthorized or tampered images from being deployed.  
- Ensures runtime workloads originate from trusted publishers.  

### 3. Image consumer verifies base images in CI/CD pipelines  

Before building an application image, developers can configure pipelines (for example, GitHub Actions) to verify the signatures of base images.  
- Protects builds from inheriting vulnerabilities or malicious code.  
- Enforces use of only trusted upstream components in application builds.  

### 4. Verifying other OCI artifacts  

Beyond container images, the same process applies to other artifacts stored in an OCI registry:  
- **SBOMs**: Verify the signed SBOM of an image before before using it for security analysis.  
- **Helm charts**: Verify charts before installing them to Kubernetes clusters.  
- **Configuration bundles or AI models**: Verify they originate from the intended publisher before integrating into systems.  

## Next steps

This overview introduces the importance of signing and verifying both container images and other OCI artifacts. Each of the following scenarios has its own dedicated guidance:  

### Signing with AKV

Signing using Notation CLI:
- [Sign container images with Notation and Azure Key Vault using a self-signed certificate](container-registry-tutorial-sign-build-push.md)  
- [Sign container images with Notation and Azure Key Vault using a CA-issued certificate](container-registry-tutorial-sign-trusted-ca.md)

Signing in GitHub workflow:
- [Sign container images in GitHub workflow with Notation and Azure Key Vault](/azure/security/container-secure-supply-chain/articles/notation-sign-gha)

### Signing with Trusted Signing

Signing using Notation CLI:
- [Sign container images with Notation and Trusted Signing (Preview)](container-registry-tutorial-sign-verify-notation-trusted-signing.md)

Signing in GitHub workflow:
- [Sign container images in GitHub workflow with Notation and Trusted Signing (Preview)](container-registry-tutorial-github-sign-notation-trusted-signing.md)

### Verfication

Verification in GitHub workflow:
- [Verify container images in GitHub workflow with Notation and Azure Key Vault](/azure/security/container-secure-supply-chain/articles/verify-gha)
- [Verify container images in GitHub workflow with Notation and Trusted Signing (Preview)](container-registry-tutorial-github-verify-notation-trusted-signing.md)

Verification on AKS:
- [Validate container image signatures in AKS with Ratify and Azure Policy](container-registry-tutorial-verify-with-ratify-aks.md)
