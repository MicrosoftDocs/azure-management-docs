---
title: "Azure Arc-enabled Kubernetes connectivity modes"
ms.date: 09/14/2026
ms.topic: concept-article
description: "This article provides an overview of the connectivity modes supported by Azure Arc-enabled Kubernetes"
# Customer intent: "As a Kubernetes administrator, I want to understand the connectivity modes of Azure Arc-enabled Kubernetes, so that I can effectively manage agents and ensure consistent communication with Azure services for my clusters."
---

# Azure Arc-enabled Kubernetes connectivity modes

Azure Arc-enabled Kubernetes requires deployment of Azure Arc agents on your Kubernetes clusters so that capabilities such as [configurations (GitOps)](conceptual-gitops-flux2.md), extensions, [cluster connect](conceptual-cluster-connect.md), and [custom location](conceptual-custom-locations.md) are made available on the cluster. Because Kubernetes clusters deployed on the edge might not have constant network connectivity, the agents might not always be able to reach the Azure Arc services while in a semi-connected mode.

## Understand connectivity modes

When you work with Azure Arc-enabled Kubernetes clusters, it's important to understand how network connectivity modes impact your operations.

- **Fully connected**: With ongoing network connectivity, agents can consistently communicate with Azure. In this mode, there's typically little delay with tasks such as propagating GitOps configurations, enforcing Azure Policy and Gatekeeper policies, or collecting workload metrics and logs in Azure Monitor.

- **Semi-connected**:  Azure Arc agents can pull desired state specification from the Arc services, then later realize this state on the cluster.

  > [!IMPORTANT]
  > The managed identity certificate that the `clusteridentityoperator` pulls down is valid for up to 90 days before it expires. The agents try to renew the certificate during this time period; however, if there's no network connectivity, the certificate might expire, and the Azure Arc-enabled Kubernetes resource stops working. Because of this condition, ensure that the connected cluster has network connectivity at least once every 30 days. If the certificate expires, you need to delete and then recreate the Azure Arc-enabled Kubernetes resource and agents to reactivate Azure Arc features on the cluster.

- **Disconnected**: Kubernetes clusters in disconnected environments that can't access Azure aren't currently supported by Azure Arc-enabled Kubernetes.

## Connectivity status

The connectivity status of a cluster is determined by the time of the latest heartbeat received from the Arc agents deployed on the cluster:

| Status | Description |
| ------ | ----------- |
| Connecting | The Azure Arc-enabled Kubernetes resource was created in Azure, but the service didn't receive the agent heartbeat yet. |
| Connected | The Azure Arc-enabled Kubernetes service received an agent heartbeat within the previous 15 minutes. |
| Offline | The Azure Arc-enabled Kubernetes resource was previously connected, but the service didn't receive any agent heartbeat for at least 15 minutes. |
| Expired | The managed identity certificate of the cluster expired. In this state, Azure Arc features no longer work on the cluster. For more information about how to address expired Azure Arc-enabled Kubernetes resources, see the [FAQ](./faq.md#how-do-i-address-expired-azure-arc-enabled-kubernetes-resources). |

## Next steps

- Walk through our quickstart to [connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md).
- Learn more about creating connections between your cluster and a Git repository as a [configuration resource with Azure Arc-enabled Kubernetes](./conceptual-gitops-flux2.md).
- Review the [Azure Arc networking requirements](network-requirements.md).
