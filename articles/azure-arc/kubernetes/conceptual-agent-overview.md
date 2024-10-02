---
title: "Azure Arc-enabled Kubernetes agent overview"
ms.date: 10/02/2024
ms.topic: conceptual
description: "Learn about the Azure Arc agents deployed on the Kubernetes clusters when connecting them to Azure Arc."
---

# Azure Arc-enabled Kubernetes agent overview

[Azure Arc-enabled Kubernetes](overview.md) provides a centralized, consistent control plane to manage policy, governance, and security across Kubernetes clusters in different environments.

Azure Arc agents are deployed on Kubernetes clusters when you [connect them to Azure Arc](quickstart-connect-cluster.md). This article provides an overview of these agents.

## Deploy agents to your cluster

Most on-premises datacenters enforce strict network rules that prevent inbound communication on the network boundary firewall. Azure Arc-enabled Kubernetes works with these restrictions by not requiring inbound ports on the firewall. Azure Arc agents require outbound communication to a [set list of network endpoints](network-requirements.md).

This diagram provides a high-level view of Azure Arc components. Kubernetes clusters in on-premises datacenters or different clouds are connected to Azure through the Azure Arc agents. This allows the clusters to be managed in Azure using management tools and Azure services. The clusters can also be accessed through offline management tools.

:::image type="content" source="media/architectural-overview.png" alt-text="Diagram showing an architectural overview of the Azure Arc-enabled Kubernetes agents." lightbox="media/architectural-overview.png":::

The following high-level steps are involved in [connecting a Kubernetes cluster to Azure Arc](quickstart-connect-cluster.md):

1. Create a Kubernetes cluster on your choice of infrastructure (VMware vSphere, Amazon Web Services, Google Cloud Platform, etc.). The cluster must already exist before you connect it to Azure Arc.

1. Start the Azure Arc registration for your cluster.

   * The agent Helm chart is deployed on the cluster.
   * The cluster nodes initiate an outbound communication to the [Microsoft Container Registry](https://github.com/microsoft/containerregistry), pulling the images needed to create the following agents in the `azure-arc` namespace:

     | Agent | Description |
     | ----- | ----------- |
     | `deployment.apps/clusteridentityoperator` | Azure Arc-enabled Kubernetes currently supports only [system assigned identities](/azure/active-directory/managed-identities-azure-resources/overview). `clusteridentityoperator` initiates the first outbound communication. This first communication fetches the Managed Service Identity (MSI) certificate used by other agents for communication with Azure. |
     | `deployment.apps/config-agent` | Watches the connected cluster for source control configuration resources applied on the cluster. Updates the compliance state. |
     | `deployment.apps/controller-manager` | An operator of operators that orchestrates interactions between Azure Arc components. |
     | `deployment.apps/metrics-agent` | Collects metrics of other Arc agents to verify optimal performance. |
     | `deployment.apps/cluster-metadata-operator` | Gathers cluster metadata, including cluster version, node count, and Azure Arc agent version. |
     | `deployment.apps/resource-sync-agent` | Syncs the above-mentioned cluster metadata to Azure. |
     | `deployment.apps/flux-logs-agent` | Collects logs from the Flux operators deployed as a part of [source control configuration](conceptual-gitops-flux2.md). |
     | `deployment.apps/extension-manager` | Installs and manages lifecycle of extension Helm charts. |
     | `deployment.apps/kube-aad-proxy` | Used for authentication of requests sent to the cluster using cluster connect. |
     | `deployment.apps/clusterconnect-agent` | Reverse proxy agent that enables the cluster connect feature to provide access to `apiserver` of the cluster. Optional component deployed only if the [cluster connect](conceptual-cluster-connect.md) feature is enabled.  |
     | `deployment.apps/guard` | Authentication and authorization webhook server used for Microsoft Entra RBAC. Optional component deployed only if [Azure RBAC](conceptual-azure-rbac.md) is enabled on the cluster.   |

1. Once all the Azure Arc-enabled Kubernetes agent pods are in `Running` state, verify that your cluster is connected to Azure Arc. You should see:

   * An Azure Arc-enabled Kubernetes resource in [Azure Resource Manager](/azure/azure-resource-manager/management/overview). Azure tracks this resource as a projection of the customer-managed Kubernetes cluster, not the actual Kubernetes cluster itself.
   * Cluster metadata (such as Kubernetes version, agent version, and number of nodes) appearing on the Azure Arc-enabled Kubernetes resource as metadata.

## Move Arc-enabled Kubernetes clusters across Azure regions

In some circumstances, you may want to move your [Arc-enabled Kubernetes clusters](overview.md) to another region. For example, you might want to deploy features or services that are only available in specific regions, or you need to change regions due to internal policy and governance requirements or capacity planning considerations.

Moving a connected cluster to a new region means deleting the `connectedClusters` Azure Resource Manager resource in the source region and then [connecting your cluster](quickstart-connect-cluster.md) again in the target region.

Source control configurations, [Flux configurations](conceptual-gitops-flux2.md) and [extensions](conceptual-extensions.md) within the cluster are child resources of the connected cluster resource. To move these resources, you'll need to save details about the resources, then move the parent `connectedClusters` resource. After that, you can recreate the child resources in the target cluster resource.

Before you begin, ensure that Azure Arc-enabled Kubernetes resources (`Microsoft.Kubernetes/connectedClusters`) and any needed Azure Arc-enabled Kubernetes configuration resources (`Microsoft.KubernetesConfiguration/SourceControlConfigurations`, `Microsoft.KubernetesConfiguration/Extensions`, `Microsoft.KubernetesConfiguration/FluxConfigurations`) are [supported in the target region](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/?products=azure-arc).

1. Do a LIST to get all configuration resources in the source cluster (the cluster to be moved) and save the response body:

   * [Microsoft.KubernetesConfiguration/SourceControlConfigurations](/cli/azure/k8s-configuration?view=azure-cli-latest&preserve-view=true#az-k8sconfiguration-list)
   * [Microsoft.KubernetesConfiguration/Extensions](/cli/azure/k8s-extension?view=azure-cli-latest&preserve-view=true#az-k8s-extension-list)
   * [Microsoft.KubernetesConfiguration/FluxConfigurations](/cli/azure/k8s-configuration/flux?view=azure-cli-latest&preserve-view=true#az-k8s-configuration-flux-list)

   > [!NOTE]
   > LIST/GET of configuration resources **do not** return `ConfigurationProtectedSettings`. For such cases, the only option is to save the original request body and reuse them while creating the resources in the new region.

1. [Delete](./quickstart-connect-cluster.md?tabs=azure-cli#clean-up-resources) the previous Arc deployment from the underlying Kubernetes cluster.
1. With network access to the underlying Kubernetes cluster, [connect the cluster](./quickstart-connect-cluster.md#connect-an-existing-kubernetes-cluster) in the new region.
1. Verify that the Arc connected cluster is successfully running in the new region:

   1. Run `az connectedk8s show -n <connected-cluster-name> -g <resource-group>` and ensure the `connectivityStatus` value is `Connected`.
   1. Run [this command](./quickstart-connect-cluster.md?tabs=azure-cli#view-azure-arc-agents-for-kubernetes) to verify all Arc agents are successfully deployed on the underlying cluster.
   1. Do a LIST of all configuration resources in the target cluster. This should match the original LIST response from the source cluster.

1. Using the response body you saved, recreate each of the configuration resources obtained in the LIST command from the source cluster on the target cluster.

If you don't need to move the cluster, but just want to move configuration resources to an Arc-enabled Kubernetes cluster in a different region, do the following:

1. Do a LIST to get all configuration resources in the source cluster as noted above, and save the response body.
1. Delete the resources from the source cluster.
1. In the target cluster, recreate each of the configuration resources obtained in the LIST command from the source cluster.

## Next steps

* Walk through our quickstart to [connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md).
* View release notes to see [details about the latest agent versions](release-notes.md).
* Learn about [upgrading Azure Arc-enabled Kubernetes agents](agent-upgrade.md).
* Learn more about the creating connections between your cluster and a Git repository as a [configuration resource with Azure Arc-enabled Kubernetes](./conceptual-configurations.md).
