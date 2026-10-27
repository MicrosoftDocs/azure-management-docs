---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: 'GitOps based application deployment for AKS and Azure Arc enabled Kubernetes '
description: Provides overview on GitOps managed offerings for AKS and Azure Arc enabled Kubernetes
author:      ponatara # GitHub alias
ms.author: davidsmatlak
ms.service: azure-arc
ms.topic: overview
ms.date: 09/17/2026
feedback_help_link_url: https://learn.microsoft.com/answers/tags/133/azure
feedback_product_url: https://feedback.azure.com/d365community/forum/79b1327d-d925-ec11-b6e6-000d3a4f06a4
searchScope:
- Azure
---

# GitOps-based application deployments for AKS and Azure Arc-enabled Kubernetes

GitOps provides a declarative approach to Kubernetes application deployment and configuration management. By using source control as the system of record, platform teams can consistently deploy and manage applications across Azure Kubernetes Service (AKS), Azure Arc-enabled Kubernetes, on-premises environments, edge locations, and multicloud deployments.

## Why GitOps?

As Kubernetes adoption grows, organizations often need a consistent way to manage application deployments across multiple environments and clusters.

GitOps helps platform teams:

- Standardize deployment workflows.
- Improve deployment consistency.
- Detect and remediate configuration drift.
- Audit and review infrastructure and application changes.
- Reduce manual operational tasks.
- Scale application delivery across large Kubernetes fleets.

By storing desired configuration in source control, teams can use existing development processes, reviews, and approval workflows to manage Kubernetes deployments.

## Managed GitOps solutions in Azure

Azure provides managed GitOps experiences for both Flux and Argo CD on Azure Kubernetes Service (AKS) and Azure Arc-enabled Kubernetes.

Organizations can use GitOps to manage:

- Application deployments
- Kubernetes configuration
- Infrastructure components
- Environment-specific settings
- Fleet-wide operational standards

By providing managed GitOps experiences, Azure helps platform teams reduce operational overhead while continuing to use familiar GitOps workflows and open-source tooling.

## Enterprise capabilities

Both Flux and Argo CD extensions integrate with Azure services and enterprise operational practices, enabling organizations to scale GitOps adoption across teams, environments, and clusters.

### Azure-managed lifecycle management

Flux and Argo CD extensions are available as Azure-managed experiences that reduce the operational overhead associated with deploying, upgrading, and maintaining GitOps infrastructure.

Benefits include:

- Simplified deployment and upgrades
- Consistent management across environments
- Azure Resource Manager integration
- Centralized governance and operations
- Reduced maintenance burden

### Hybrid and multicloud deployments

Apply consistent GitOps practices across:

- Azure Kubernetes Service (AKS)
- Azure Arc-enabled Kubernetes (On-premises Kubernetes environments, Multicloud Kubernetes deployments)

Platform teams can standardize application delivery and configuration management regardless of where clusters run.

### Enterprise identity and security

GitOps solutions integrate with Microsoft Entra ID and Azure identity services, helping organizations align deployment workflows with existing security and compliance requirements.

Benefits include:

- Centralized authentication
- Consistent access controls
- Reduced credential management
- Improved auditability
- Support for secure access to Azure services

### Flexible deployment sources

Flux and Argo CD extensions support common Kubernetes deployment sources including:

- Git repositories
- Helm charts
- OCI artifacts
- Kustomize configurations

This flexibility allows teams to adopt GitOps without changing existing packaging and deployment workflows.

### Governance and compliance

Platform teams can apply consistent governance controls across environments and clusters. Common scenarios include:


- Environment standardization
- Repository governance
- Policy enforcement
- Team-level isolation
- Controlled promotion across environments
- Configuration consistency across large cluster fleets

### Observability and operations

GitOps deployments integrate with Azure monitoring and operational tooling, enabling teams to monitor deployment health, synchronization status, and platform operations by using familiar Azure experiences. This integration helps platform teams maintain operational visibility while quickly identifying deployment issues and configuration drift.


### Fleet-scale Kubernetes management

Organizations that operate hundreds or thousands of Kubernetes clusters can use GitOps to standardize application delivery and operational practices across distributed environments.

Common examples include:

- Retail stores
- Manufacturing facilities
- Branch offices
- Healthcare environments
- Financial services organizations

By using GitOps, platform teams can reduce operational variation while maintaining governance, compliance, and security requirements across large Kubernetes fleets.



## Next steps

- Get started with GitOps by using Flux.
- Deploy applications by using GitOps with Argo CD.
- Manage GitOps deployments across AKS and Azure Arc-enabled Kubernetes.
- Configure enterprise capabilities such as identity integration, monitoring, governance, and fleet-scale operations.