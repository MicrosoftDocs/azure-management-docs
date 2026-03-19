---
title: "Application deployments with GitOps (ArgoCD)"
description: "This article provides a conceptual overview of GitOps with ArgoCD for use in Azure Arc-enabled Kubernetes and Azure Kubernetes Service (AKS) clusters."
ms.date: 03/23/2026
ms.topic: concept-article
# Customer intent: "As a DevOps engineer, I want to implement GitOps with Flux for managing application deployments on Azure Kubernetes clusters, so that I can ensure consistent configurations, streamline operations, and enhance visibility into the status of applications across multiple environments."
---

# Application deployments with GitOps (ArgoCD) for AKS and Azure Arc-enabled Kubernetes

When using [ArgoCD for GitOps on Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes clusters](tutorial-use-gitops-argocd.md), your Git repository becomes the source of truth for the desired state of your applications and cluster configurations. This approach offers several benefits.

- **Continuous synchronization**: ArgoCD runs natively within your cluster, pulling manifests, Helm charts, or Kustomize configurations directly from Git.
- **Drift detection and remediation**: The system constantly monitors the cluster's live state against the desired state in Git. Any "drift" is instantly detected and reconciled based on your specific automation policies.
- **Security-first pull model**: Unlike traditional CI/CD, ArgoCD uses a pure pull-based architecture. Since the cluster initiates the connection, you don't need to open inbound firewall ports or grant Git repositories network access to your private clusters. This makes ArgoCD ideal for high-security environments, edge computing, and massive hybrid-cloud scales.  

Argo CD Cluster Extension 

To streamline management, Azure provides Argo CD as a managed cluster extension (Microsoft.KubernetesConfiguration/extensions/microsoft.argocd).  

Prerequisite: This extension must be active before you can manage applications via GitOps. 

Deployment: The extension installs automatically the first time you enable Argo CD GitOps on a cluster. For custom setups, you can deploy it manually via the Azure Portal, Azure CLI, ARM templates, or REST API.  