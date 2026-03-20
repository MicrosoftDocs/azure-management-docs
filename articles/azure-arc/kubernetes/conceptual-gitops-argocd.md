---
title: "Application deployments with GitOps (ArgoCD)"
description: "This article provides a conceptual overview of GitOps with ArgoCD for use in Azure Arc-enabled Kubernetes and Azure Kubernetes Service (AKS) clusters."
ms.date: 03/23/2026
ms.topic: concept-article
# Customer intent: "As a DevOps engineer, I want to implement GitOps with Flux for managing application deployments on Azure Kubernetes clusters, so that I can ensure consistent configurations, streamline operations, and enhance visibility into the status of applications across multiple environments."
---

# Application deployments with GitOps (ArgoCD) for AKS and Azure Arc-enabled Kubernetes

When using [ArgoCD for GitOps on Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes clusters](tutorial-use-gitops-argocd.md), your Git repository becomes the source of truth for the desired state of your applications and cluster configurations. This approach offers several benefits:

- **Continuous synchronization**: ArgoCD runs natively within your cluster, pulling manifests, Helm charts, or Kustomize configurations directly from Git.
- **Drift detection and remediation**: The system constantly monitors the cluster's live state against the desired state in Git. Any "drift" is instantly detected and reconciled based on your specific automation policies.
- **Security-first pull model**: Unlike traditional CI/CD, ArgoCD uses a pure pull-based architecture. Since the cluster initiates the connection, you don't need to open inbound firewall ports or grant Git repositories network access to your private clusters. This makes ArgoCD ideal for high-security environments, edge computing, and massive hybrid-cloud scales.  

## ArgoCD cluster extension

To streamline management, Azure provides ArgoCD as a managed [cluster extension](conceptual-extensions.md) (`Microsoft.KubernetesConfiguration/extensions/microsoft.argocd`). To manage applications via GitOps, [deploy the ArgoCD extension](tutorial-use-gitops-argocd.md#create-gitops-argocd-extension-simple-installation) to your cluster.

## Managed components and controllers

The ArgoCD extension packages and manages the core components required for enterprise-grade GitOps:

| **Component** | **Responsibility** |
|-----------|-----------------|
| Application controller | Acts as the primary orchestrator, continuously monitoring cluster state and executing reconciliation workflows to maintain alignment with the desired state. |
| API server | Serves as the central interface, facilitating communication between the Web UI, CLI, and external third-party integrations. |
| Repository server | Manages the caching of Git metadata and automatically generates Kubernetes manifests from source configurations. |
| Redis | Provides a high-performance, in-memory data store to optimize caching of application states and authentication tokens. |
| Dex and Notifications | Oversees identity management through Single Sign-On (SSO) and automates the delivery of status alerts to platforms such as Microsoft Teams and email. |

## ArgoCD constructs

Once the ArgoCD extension is deployed to your cluster, you can manage your workloads using ArgoCD constructs, such as:

- **Application and ApplicationSet**: Define individual applications for single-target deployments, or use ApplicationSets to automate creation of multiple applications across different clusters and environments based on templates.

- **Management interfaces**: Management and monitor resources through the ArgoCD Web UI for a visual representation of cluster health, or by using the ArgoCD CLI for terminal-based operations and automation.

For more information, see the [ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/).

## Identity and access

To align with enterprise security requirements, the ArgoCD extension integrates with Azure identity for both AKS and Azure Arc‑enabled Kubernetes clusters.

The extension supports workload identity federation, enabling ArgoCD to securely access Azure resources such as Azure Container Registry (ACR) and Azure DevOps, without relying on long‑lived secrets. The ArgoCD extension also supports single sign‑on (SSO) using Microsoft Entra ID, allowing users to authenticate to ArgoCD using their existing enterprise identities. With this model, credentials and access policies are centrally governed through Azure, rather than embedded directly in cluster configuration or Git repositories.

## Next steps

- Learn how to deploy the ArgoCD extension and manage applications using GitOps in the [ArgoCD GitOps tutorial](tutorial-use-gitops-argocd.md).