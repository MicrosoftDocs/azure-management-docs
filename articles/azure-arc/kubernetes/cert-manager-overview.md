---
title: "Cert-manager for Arc-enabled Kubernetes (preview)"
ms.date: 04/13/2026
ms.topic: overview
ms.custom: references_regions
description: "The cert-manager for Azure Arc-enabled Kubernetes (preview) extension provides a unified, automated solution for managing TLS certificates and trust bundles in Arc-connected clusters."
# Customer intent: As a customer using Azure Arc-enabled Kubernetes, I want to understand how cert-manager can help me manage TLS certificates and trust bundles in my Arc-connected clusters, so that I can ensure secure communication and compliance across my hybrid and multicloud environments.
---

# What is cert-manager for Azure Arc-enabled Kubernetes (preview)?

The cert-manager for Azure Arc-enabled Kubernetes (preview) extension provides a unified, automated solution for managing TLS certificates and trust bundles in Arc-connected clusters. It simplifies the process of issuing, renewing, and managing certificates across hybrid and multicloud environments, providing secure communication and compliance with organizational policies.

> [!IMPORTANT]
> Cert-manager for Azure Arc-enabled Kubernetes clusters is currently in public preview.
>
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

The cert-manager for Arc-enabled Kubernetes extension installs a Microsoft-supported version of [cert-manager](https://cert-manager.io/) and [trust-manager](https://cert-manager.io/docs/trust/trust-manager/). Using the cert-manager for Arc-enabled Kubernetes extension provides the following benefits:

- **Automated certificate lifecycle**: No more manual certificate creation or renewal. Use cert-manager to issue certificates for your Kubernetes workloads or cluster components and renew them before they expire. This dramatically reduces the risk of service outages due to expired certificates and minimizes human error in certificate handling.
- **Centralized trust management**: Use trust-manager to propagate certification authority (CA) certificates (trust bundles) across namespaces in the cluster. This ensures that all pods and services share a consistent set of trusted CA certificates for TLS verifications, enabling secure service-to-service communication without manual intervention.
- **Integration with enterprise Public Key Infrastructure (PKI)**: The cert-manager for Arc-enabled Kubernetes extension supports both self‑signed CA certificates and external enterprise CA certificates. You can let it create a self‑signed cluster‑local CA certificate for internal certificates, or configure it to request certificates from your organization's existing PKI (for example, an external corporate CA or an ACME provider). In addition, you can create a cluster‑local issuer that is either a root self‑signed issuer or an intermediate issuer that chains to a key and certificate obtained from your enterprise PKI, which can be useful for air‑gapped or offline certificate issuance scenarios. This flexibility allows you to enforce enterprise compliance requirements by ensuring all certificates are ultimately issued from, or chained to, a known and trusted authority, while still fully automating certificate lifecycle management.
- **Offline and edge friendly**: The cert-manager for Arc-enabled Kubernetes extension is designed for edge scenarios. The extension can function for a period even if the cluster is temporarily disconnected from Azure or public networks. For example, certificates and trust info remains usable offline (limited by configured expiration and renewal periods), ensuring continued secure operation in intermittent connectivity scenarios.
- **Microsoft-supported distribution**: Unlike a self-managed open source deployment of cert-manager and trust-manager, this extension is maintained by Microsoft as part of Azure Arc. When you use cert-manager for Arc-enabled Kubernetes, you receive proactive updates, security patches, and enterprise support for the cert-manager and trust-manager components. This extension aligns with Azure's security model and compliance standards, giving you peace of mind that certificate management meets corporate and regulatory requirements.

## How cert-manager for Arc-enabled Kubernetes works

After you install the cert-manager for Arc-enabled Kubernetes (preview) extension on your Arc-enabled Kubernetes cluster, Azure Arc deploys cert-manager and trust-manager services to the `cert-manager` namespace in the cluster. These services run with minimal footprint, and they continuously watch custom resources that describe certificate needs or trust bundles.

You can create Kubernetes custom resources such as Issuer/ClusterIssuer and Certificate objects to request certificates, or Bundle objects to distribute CA certificates. Cert-manager then works with the configured issuers to obtain the certificates (for example, generating a key pair and getting it signed by the specified CA) and stores the resulting certificate and key in a Kubernetes secret for your application to use. Trust-manager will pick up any Bundle resources and create the corresponding ConfigMaps of CA certificates in target namespaces, automatically managing updates and propagation.

## Validated Arc-enabled Kubernetes distributions

The current version of the cert-manager for Arc-enabled Kubernetes (preview) extension has been validated on the following Arc-enabled Kubernetes distributions:

- Azure Kubernetes Service (AKS) enabled by Azure Arc: v1.32.7
- VMWare Tanzu TKGm: v1.28.11
- K3s: v1.33.3+k3s1
- Red Hat Open Shift: v4.17.15, v4.18.9, v4.20.8
- Suse Rancher RKE2: v1.33.6+rke2r1, v1.34.2+rke2r, v1.35.0+rke2r3
- AKS Edge Essentials: v1.30.5 and v.1.31.5

While you can deploy the extension to any Kubernetes cluster connected to Azure Arc, some functionality may not work as expected if your cluster is running a distribution that hasn't been validated.

## Regional support

Currently, cert-manager for Azure Arc-enabled Kubernetes (preview) is supported in the following Azure regions:

- Australia East
- Brazil South
- Canada Central
- Canada East
- Central India
- Central US
- East Asia
- East US
- East US 2
- France Central
- Germany West Central
- Israel Central
- Italy North
- Japan East
- Korea Central
- North Central US
- North Europe
- Norway East
- South Africa North
- South Central US
- South India
- Southeast Asia
- Sweden Central
- Switzerland North
- UAE North
- UK South
- UK West
- West Central US
- West Europe
- West US
- West US 2
- West US 3

## Next steps

- Learn how to [deploy cert-manager for Arc-enabled Kubernetes (preview)](cert-manager-deploy.md).
- Learn how to [monitor and troubleshoot cert-manager for Arc-enabled Kubernetes (preview)](cert-manager-monitor-troubleshoot.md).
