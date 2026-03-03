---
title: "Overview of Azure Arc-enabled Kubernetes"
ms.date: 10/08/2024
ms.topic: overview
description: "Azure Arc-enabled Kubernetes allows you to attach Kubernetes clusters running anywhere so that you can manage and configure them in Azure."
# Customer intent: As a DevOps engineer managing multiple Kubernetes clusters across environments, I want to connect and manage them through a unified control plane in Azure, so that I can streamline operations, enforcement of policies, and application deployment across diverse platforms.
---

# What is Azure Arc-enabled Kubernetes?

Azure Arc-enabled Kubernetes allows you to attach Kubernetes clusters running anywhere so that you can manage and configure them in Azure. When you manage all your Kubernetes resources in a single control plane, you get a more consistent development and operation experience. This approach helps you run cloud-native apps anywhere and on any Kubernetes platform.

When you [deploy Azure Arc agents to the cluster](quickstart-connect-cluster.md), the agents create a secure outbound connection to Azure.

Each Kubernetes cluster that you connect to Azure appears as its own resource in Azure Resource Manager. You can organize these clusters with resource groups and tagging, just like your other Azure resources.

## Supported Kubernetes distributions

Azure Arc-enabled Kubernetes works with any Cloud Native Computing Foundation (CNCF) certified Kubernetes clusters. This support includes clusters running on other public cloud providers, such as GCP or AWS, and clusters running in your on-premises data center, such as VMware vSphere or Azure Local.

The Azure Arc team works with key industry partners to [validate conformance of Kubernetes distributions with Azure Arc-enabled Kubernetes](./validation-program.md).

## Arc-enabled Kubernetes scenarios and enhanced functionality

After you connect your Kubernetes clusters to Azure, you can use a wide variety of Azure services and features to manage your clusters at scale, such as:

* View all connected Kubernetes clusters for inventory, grouping, and tagging, along with your Azure Kubernetes Service (AKS) clusters.

* Configure clusters and deploy applications using [GitOps-based configuration management](tutorial-use-gitops-flux2.md).

* View and monitor your clusters by using [Azure Monitor for containers](/azure/azure-monitor/containers/container-insights-enable-arc-enabled-clusters?toc=/azure/azure-arc/kubernetes/toc.json).

* Enable threat protection by using [Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-kubernetes-azure-arc?toc=/azure/azure-arc/kubernetes/toc.json).

* Manage and report on compliance by using [Azure Policy](/azure/governance/policy/concepts/policy-for-kubernetes?toc=/azure/azure-arc/kubernetes/toc.json).

* [Connect to your Kubernetes clusters from anywhere](cluster-connect.md), and manage access by using [Azure role-based access control (Azure RBAC)](azure-rbac.md).

* Deploy machine learning workloads by using [Azure Machine Learning for Kubernetes clusters](/azure/machine-learning/how-to-attach-kubernetes-anywhere?toc=/azure/azure-arc/kubernetes/toc.json).

* Deploy and manage [Kubernetes applications from Microsoft Marketplace](deploy-marketplace.md).

* Deploy services that allow you to take advantage of specific hardware, comply with data residency requirements, or enable new scenarios, such as [Azure Arc-enabled data services](../data/overview.md) or [Event Grid on Kubernetes](/azure/event-grid/kubernetes/overview).

* Use [Azure Kubernetes Fleet Manager](/azure/kubernetes-fleet/overview) and its Arc-enabled Kubernetes cluster extension to tackle hybrid and multicloud Kubernetes management challenges at scale.

[!INCLUDE [azure-lighthouse-supported-service](~/reusable-content/ce-skilling/azure/includes/azure-lighthouse-supported-service.md)]

## Next steps

* Learn about best practices and design patterns through the [Cloud Adoption Framework for hybrid and multicloud](/azure/cloud-adoption-framework/scenarios/hybrid/arc-enabled-kubernetes/eslz-arc-kubernetes-identity-access-management).
* Try out Arc-enabled Kubernetes without provisioning a full environment by using the [Azure Arc Jumpstart](https://jumpstart.azure.com/azure_arc_jumpstart/azure_arc_k8s).
* See [what's new with Azure Arc-enabled Kubernetes](release-notes.md).
* [Connect an existing Kubernetes cluster to Azure Arc](quickstart-connect-cluster.md).
* Help protect your cluster by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
