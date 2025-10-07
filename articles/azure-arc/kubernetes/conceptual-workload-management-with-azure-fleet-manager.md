---
title: "Multi-cluster workload management with Azure Kubernetes Fleet Manager"
description: "This article provides a conceptual overview of multi-cluster workload management using the Azure Kubernetes Fleet Manager extension for Azure Arc-enabled Kubernetes clusters."
ms.date: 10/01/2025
ms.topic: concept-article
author: ealianis
ms.author: sehobbs
# Customer intent: "As a DevOps engineer managing multiple Azure Arc-enabled Kubernetes clusters, I want to use Azure Kubernetes Fleet Manager to automate workload scheduling and deployment across clusters, so that I can efficiently manage applications at scale with centralized control and observability."
---

# Multi-cluster workload management with Azure Kubernetes Fleet Manager

Managing Kubernetes workloads across multiple Azure Arc-enabled clusters presents unique challenges for organizations operating in hybrid environments. The Azure Kubernetes Fleet Manager extension for Azure Arc-enabled Kubernetes (currently in public preview) provides a solution that extends Azure's native fleet management capabilities to clusters running outside of Azure, enabling centralized multi-cluster workload management regardless of where your clusters are located.

## Why use Azure Kubernetes Fleet Manager extension for Azure Arc-enabled Kubernetes?

The Azure Kubernetes Fleet Manager extension enables organizations to tackle multi-cluster orchestrations across their Kubernetes clusters:

### Consistent management across hybrid environments

Azure Arc-enabled Kubernetes clusters often span multiple environments - on-premises data centers, edge locations, and other cloud providers. The Azure Kubernetes Fleet Manager extension provides a unified control plane that maintains consistent management practices across all these diverse locations, eliminating the operational silos that typically emerge in hybrid scenarios.

### Centralized orchestration with local execution

While your clusters may be distributed across different geographic locations and network environments, the extension enables centralized orchestration through Azure while respecting local constraints and connectivity patterns. This approach is particularly valuable for organizations with edge computing requirements or regulatory constraints that require local data processing.

### Azure-native tooling and integration

By extending Azure Kubernetes Fleet Manager capabilities to Arc-enabled clusters, organizations can leverage familiar Azure tooling, security models, and operational practices across their entire Kubernetes infrastructure. This integration reduces learning curves and maintains consistency with existing Azure-based workflows.

## Preview limitations and capability restrictions

Azure Kubernetes Fleet Manager extension for Azure Arc-enabled Kubernetes is currently in public preview. During preview, certain capabilities are not available for Arc-enabled clusters. For a complete list of requirements and limitations, see [Member cluster types](https://learn.microsoft.com/azure/kubernetes-fleet/concepts-member-cluster-types).

## Key capabilities

The Azure Kubernetes Fleet Manager extension brings several powerful capabilities to Azure Arc-enabled Kubernetes clusters:

- **Declarative workload placement**: Define where workloads should run based on cluster characteristics.
- **Resource propagation**: Deploy Kubernetes resources consistently across multiple clusters in different environments.
- **Centralized observability**: Monitor workload health and deployment status across all clusters from a single Azure control plane.


## Architecture and comprehensive details

For complete information about Azure Kubernetes Fleet Manager's architecture, scheduling mechanisms, resource propagation, and detailed implementation guidance, see [Azure Kubernetes Fleet Manager resource management across multiple clusters using hub cluster as the control plane](https://learn.microsoft.com/azure/kubernetes-fleet/concepts-multi-cluster-workload-management).

This comprehensive guide covers:

- Overall architecture for multi-cluster resource management
- Hub-and-spoke control plane design
- Advanced scheduling and rollout strategies
- Kubernetes-native extension model
- Multi-cluster management challenges and solutions

## Next steps

- Get started with [Azure Kubernetes Fleet Manager](https://learn.microsoft.com/azure/kubernetes-fleet/overview)
