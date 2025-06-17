---
title: "Cluster extensions in Azure Arc-enabled Kubernetes"
ms.date: 04/22/2025
ms.topic: concept-article
description: "Get a conceptual overview of the Azure Arc-enabled Kubernetes cluster extensions capability."
# Customer intent: As a cluster operator, I want to manage Kubernetes applications using cluster extensions, so that I can streamline installation and lifecycle management while ensuring compliance and efficient resource usage across my Azure Arc-enabled Kubernetes environment.
---

# Cluster extensions

This article describes how you can use Azure Arc-enabled Kubernetes cluster extensions via [Helm Charts](https://helm.sh/) to manage your Kubernetes applications. The cluster extensions feature in Azure Arc-enabled Kubernetes has all the building blocks you need to define, install, and upgrade even the most complex Kubernetes applications.

The cluster extension feature builds on the packaging components of Helm. With extensions, you use an Azure Resource Manager-driven experience for installation and lifecycle management of different capabilities on top of your Kubernetes cluster.

A cluster operator or admin can [use the cluster extensions feature](extensions.md) to:

- Install and manage key management, data, and application offerings on your Kubernetes cluster.
- Use Azure Policy to automate at-scale deployment of cluster extensions across all clusters in your environment.
- Subscribe to release trains (for example, `Preview` or `Stable`) for each extension.
- Set up autoupgrade for extensions, or you can pin a specific version and manually upgrade versions.
- Update extension properties or delete extension instances.

Extensions are available to support a wide range of Azure services and scenarios. For a list of currently supported extensions, see [Available extensions for Azure Arc-enabled Kubernetes clusters](extensions-release.md).

## Architecture

:::image type="content" source="media/conceptual-extensions.png" border="false" alt-text="Diagram showing the cluster extension installation workflow architecture." lightbox="media/conceptual-extensions.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

The cluster extension instance is created as an extension Azure Resource Manager resource (`Microsoft.KubernetesConfiguration/extensions`) on top of the Azure Arc-enabled Kubernetes resource (represented by `Microsoft.Kubernetes/connectedClusters`) in Azure Resource Manager.

This representation in Azure Resource Manager allows you to author policies that check for the presence or absence of a specific cluster extension in all Azure Arc-enabled Kubernetes resources. After you determine which clusters are missing cluster extensions that have specific property values, you can remediate noncompliant resources by using Azure Policy.

The `config-agent` component that runs on your cluster tracks new and updated extension resources on the Azure Arc-enabled Kubernetes resource. The `extensions-manager` agent running in your cluster reads the extension type that needs to be installed. Then, it pulls the associated Helm chart from Azure Container Registry or Microsoft Container Registry and installs it on the cluster.

Both the `config-agent` and `extensions-manager` components running in the cluster handle extension instance updates, version updates, and deleting extension instances. These agents use the system-assigned managed identity of the cluster to securely communicate with Azure services.

> [!NOTE]
> `config-agent` checks for new or updated extension instances on top of Azure Arc-enabled Kubernetes cluster. The agents require connectivity for the desired state of the extension to be pulled to the cluster. If agents can't connect to Azure, propagation of the desired state to the cluster is delayed.
>
> Protected configuration settings for an extension instance are stored for up to 48 hours in the Azure Arc-enabled Kubernetes services. As a result, if the cluster remains disconnected during the 48 hours after the extension resource is created in Azure, the extension changes from a `Pending` state to a `Failed` state. To prevent this, we recommend that you bring clusters online regularly.

> [!IMPORTANT]
> Currently, Azure Arc-enabled Kubernetes cluster extensions aren't supported on ARM64-based clusters, except for [Flux (GitOps)](conceptual-gitops-flux2.md) and [Microsoft Defender for Containers](/azure/defender-for-cloud/defender-for-containers-enable?pivots=defender-for-container-arc&toc=%2Fazure%2Fazure-arc%2Fkubernetes%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fkubernetes%2Fbreadcrumb%2Ftoc.json&tabs=aks-deploy-portal%2Ck8s-deploy-asc%2Ck8s-verify-asc%2Ck8s-remove-arc%2Caks-removeprofile-api#protect-arc-enabled-kubernetes-clusters). To [install and use other cluster extensions](extensions.md), the cluster must have at least one node of operating system and architecture type linux/amd64 .

## Extension scope

Each extension type defines the scope at which they operate on the cluster. Extension installations on Arc-enabled Kubernetes clusters are either *cluster-scoped* or *namespace-scoped*.

When you create an extension instance, you specify the namespace where it's installed as `release-namespace`. Typically, only one instance of a cluster-scoped extension and its components, including pods, operators, and custom resource definitions (CRDs), are installed in the release namespace on the cluster.

You can install a namespace-scoped extension in a specific namespace by using the `–namespace` property. Because extension can be deployed at a namespace scope, multiple instances of a namespace-scoped extension and its components can run on a cluster. Each instance of the extension has permissions for the namespace where it's deployed. All extensions that are described in this article are cluster-scoped except the Event Grid on Kubernetes extension.

All [currently available extensions](extensions-release.md) are cluster-scoped, except [Azure API Management on Azure Arc](/azure/api-management/how-to-deploy-self-hosted-gateway-azure-arc).

## Related content

- Use our quickstart to [connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md).
- [Deploy cluster extensions](./extensions.md) on your Azure Arc-enabled Kubernetes cluster.
