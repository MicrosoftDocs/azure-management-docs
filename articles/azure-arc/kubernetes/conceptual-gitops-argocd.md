---
title: "Application deployments with GitOps (Argo CD)"
description: "This article provides a conceptual overview of GitOps with Argo CD for use in Azure Arc-enabled Kubernetes and Azure Kubernetes Service (AKS) clusters."
ms.date: 03/23/2026
ms.topic: concept-article
# Customer intent: "As a DevOps engineer, I want to implement GitOps with Argo CD for managing application deployments on AKS amd Arc-enabled clusters, so that I can ensure consistent configurations, streamline operations, and enhance visibility into the status of applications across multiple environments."
---

# Application deployments with GitOps (Argo CD) for AKS and Azure Arc-enabled Kubernetes

When you use [Argo CD for GitOps on Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes clusters](tutorial-use-gitops-argocd.md), your Git repository becomes the source of truth for the desired state of your applications and cluster configurations. This approach offers several benefits:

- **Continuous synchronization**: Argo CD runs natively within your cluster, pulling manifests, Helm charts, or Kustomize configurations directly from Git.
- **Drift detection and remediation**: The system constantly monitors the cluster's live state against the desired state in Git. It instantly detects any drift and reconciles it based on your specific automation policies.
- **Security-first pull model**: Unlike traditional CI/CD, Argo CD uses a pure pull-based architecture. Since the cluster initiates the connection, you don't need to open inbound firewall ports or grant Git repositories network access to your private clusters. This architecture makes Argo CD ideal for high-security environments, edge computing, and massive hybrid-cloud scales.  

## Argo CD cluster extension

To streamline management, Azure provides Argo CD as a managed [cluster extension](conceptual-extensions.md) (`Microsoft.KubernetesConfiguration/extensions/microsoft.argocd`). To manage applications via GitOps, [deploy the Argo CD extension](tutorial-use-gitops-argocd.md#create-gitops-argo-cd-extension-simple-installation) to your cluster.

## Managed components and controllers

The Argo CD extension packages and manages the core components required for enterprise-grade GitOps:

| **Component name** | **Kubernetes service name** | **Primary responsibility** |
|-----------|-----------------|-----------------|
| [Application controller](https://argo-cd.readthedocs.io/en/stable/operator-manual/server-commands/argocd-application-controller/) | `argocd-application-controller` |Continuously monitors running applications and compares the live state against the desired target state in Git. |
| API server | `argocd-server` | Acts as the gRPC/REST server that exposes the API used by the Web UI, CLI, and external CI/CD systems. |
| [Repository server](https://argo-cd.readthedocs.io/en/stable/operator-manual/server-commands/argocd-repo-server/) | `argocd-repo-server` | Maintains a local cache of Git repositories and is responsible for generating Kubernetes manifests from source (Helm, Kustomize, etc.). |
| Redis | `argocd-redis` | Provides an ephemeral, in-memory data store used for caching manifest generation results and application state.|
| [Dex](https://argo-cd.readthedocs.io/en/stable/operator-manual/user-management/#dex) and notifications | `argocd-dex-server` | An open-source OpenID Connect provider that brokers identity from external providers like [GitHub](https://github.com/), SAML, or LDAP.|
| Notifications controller | `argocd-notifications-controller` | Monitors application state changes and triggers alerts to external services like Slack, Email, or Webhooks. |
| ApplicationSet controller | `argocd-applicationset-controller` | Automates the creation of multiple applications across multiple clusters. |

## Argo CD constructs

After you deploy the Argo CD extension to your cluster, you can manage your workloads by using Argo CD constructs, such as:

- **Application and ApplicationSet**: Define individual applications for single-target deployments, or use ApplicationSets to automate the creation of multiple applications across different clusters and environments based on templates.

- **Management interfaces**: Manage and monitor resources through the Argo CD Web UI for a visual representation of cluster health, or by using the Argo CD CLI for terminal-based operations and automation.

For more information, see the [Argo CD documentation](https://argo-cd.readthedocs.io/en/stable/).

## Identity and access

To align with enterprise security requirements, the Argo CD extension integrates with Azure identity for both AKS and Azure Arc‑enabled Kubernetes clusters.

The extension supports workload identity federation, which enables Argo CD to securely access Azure resources such as Azure Container Registry (ACR) and Azure DevOps without relying on long‑lived secrets. The Argo CD extension also supports single sign‑on (SSO) by using Microsoft Entra ID, which allows users to authenticate to Argo CD by using their existing enterprise identities. By using this model, Azure centrally governs credentials and access policies, rather than embedding them directly in cluster configuration or Git repositories.

## Next steps

- Learn how to deploy the Argo CD extension and manage applications by using GitOps in the [Argo CD GitOps tutorial](tutorial-use-gitops-argocd.md).