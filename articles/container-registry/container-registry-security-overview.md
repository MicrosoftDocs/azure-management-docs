---
title: Security overview for Azure Container Registry
description: Understand the Azure Container Registry security model, shared responsibility, threat surface, and defense-in-depth layers that protect your registry.
author: msmbaldwin
ms.author: mbaldwin
ms.service: azure-container-registry
ms.topic: concept-article
ms.date: 09/28/2026
ai-usage: ai-generated
# Customer intent: As a platform engineer, I want to understand how Azure Container Registry security works so that I can reason about the controls I need to configure.
---

# Security overview for Azure Container Registry

Azure Container Registry stores and distributes the container images and related artifacts that run across your compute platforms. Because those platforms treat the registry as a trusted source, the registry is part of your software supply chain: an attacker who tampers with an image, steals a credential, or reaches an unprotected registry can spread a compromise to every workload that pulls from it.

This article explains how registry security works so that you can reason about which controls apply to your environment. It covers the shared responsibility model, the registry threat surface, and the defense-in-depth layers that protect a registry. For the prescriptive checklist of what to configure and where to do it, see [Secure your Azure Container Registry deployment](secure-container-registry.md).

## Shared responsibility

Registry security is a shared responsibility between Microsoft and you.

Microsoft operates the registry service. Microsoft secures the physical infrastructure, the host platform, and the service software, and encrypts all registry content at rest by default with Microsoft-managed keys.

You control how the registry is exposed and who can reach it. You decide which identities authenticate, whether the registry is reachable from the public internet, whether you supply your own encryption key, how you enforce image integrity, and how you monitor and govern registry activity. Most registry compromises trace back to configuration you own, such as a registry-wide credential left enabled or a registry left open to the internet, rather than to the underlying service.

## The registry threat surface

A registry faces threats at both its access boundary and its content. Understanding these threats clarifies why each defense-in-depth layer exists.

- **Unauthorized access**: An attacker who obtains registry credentials or reaches an unauthenticated registry can pull proprietary images or push malicious ones. Shared, registry-wide credentials such as the admin account widen this risk because their use isn't attributable to an individual identity.
- **Public network exposure**: A registry is reachable over the public internet by default. An internet-exposed registry is discoverable and subject to credential-based attacks from anywhere.
- **Supply-chain tampering**: An attacker who modifies or substitutes an image injects vulnerable or malicious code into every workload that later pulls it. Without image integrity controls, a consumer can't distinguish a trusted image from a tampered one.
- **Data exfiltration**: An identity with export or pull rights can move proprietary images out of a controlled environment to an external registry.
- **Loss of availability or content**: A regional outage can interrupt image pulls, and an accidental or malicious deletion can remove images that running workloads depend on.

## Defense in depth

No single control secures a registry. The registry security model layers independent controls so that a gap in one layer doesn't expose the registry. Each layer below maps to a section of the [security checklist](secure-container-registry.md), which prescribes the specific settings to apply.

### Identity and access

Azure Container Registry authenticates through Microsoft Entra ID and authorizes through Azure role-based access control. The registry separates control-plane access, which manages the registry resource, from data-plane access, which pulls and pushes images. Entra identities make access attributable and let you apply least-privilege roles, scope permissions to individual repositories, and govern access with Conditional Access. Preferring managed identities and Entra-based authentication over shared credentials such as the admin account is the foundation of registry security. For the concepts behind registry authentication, see [Authentication options for Azure Container Registry](container-registry-authentication.md).

### Network isolation

Restricting how the registry is reached shrinks its attack surface. A private endpoint gives the registry a private IP address in your virtual network and keeps pull and push traffic off the public internet, and IP network rules limit public access to known address ranges. These controls, available in the Premium service tier, turn an internet-reachable registry into one that only trusted networks can resolve and reach. For the private-connectivity concept, see [Connect privately to a container registry](container-registry-private-endpoints.md).

### Content integrity and data protection

Protecting registry content spans encryption and image trust. All content is encrypted at rest by default, and a customer-managed key gives you control over the encryption key when your compliance requirements demand it. Signing images and verifying those signatures before deployment protects image integrity, so that consumers accept only images from a trusted source. Locking production images and disabling artifact export further protect content from mutation and exfiltration. For image signing, see [Content trust in Azure Container Registry](container-registry-content-trust.md).

### Detection and monitoring

Visibility into registry activity lets you detect misuse and investigate incidents. The registry records authentication attempts and repository operations as Azure Monitor resource logs, which you route to a Log Analytics workspace for retention and analysis. Vulnerability scanning through Microsoft Defender for Cloud assesses pushed images so that known vulnerabilities surface before an image is deployed. For registry monitoring concepts, see [Monitor Azure Container Registry](monitor-container-registry.md).

### Governance and compliance

Consistent security depends on enforcing configuration across every registry, not one at a time. Azure Policy audits and enforces registry settings such as disabled anonymous pull and disabled public network access at scale, and maps those controls to the regulatory standards your organization must meet. For the available controls, see [Azure Policy Regulatory Compliance controls for Azure Container Registry](security-controls-policy.md).

### Resilience

Availability and recoverability protect against outages and accidental loss. Geo-replication keeps a single logical registry available from multiple regions, and soft delete retains deleted artifacts for a recovery window. Planning these controls together, and understanding how they interact, keeps images reachable and recoverable. For the geo-replication concept, see [Geo-replication in Azure Container Registry](container-registry-geo-replication.md).

## Related content

- [Secure your Azure Container Registry deployment](secure-container-registry.md)
- [Best practices for Azure Container Registry](container-registry-best-practices.md)
- [About registries, repositories, and artifacts](container-registry-concepts.md)
