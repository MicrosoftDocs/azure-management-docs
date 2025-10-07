---
title: "Tutorial: Deploy applications using Azure Fleet Manager"
description: "This tutorial shows how to use Azure Fleet Manager to deploy applications across multiple Arc Clusters."
author: ealianis
ms.author: sehobbs
ms.date: 09/30/2025
ms.topic: tutorial
ms.custom:
  - template-tutorial
  - ignite-2025
# Customer intent: As a DevOps engineer, I want to deploy applications to multiple kubernetes clusters at scale and across hybrid environments.
---

# Tutorial: Deploy applications using Azure Fleet Manager

This tutorial shows how to use [Azure Fleet Manager](https://learn.microsoft.com/azure/kubernetes-fleet/overview) to deploy applications across multiple clusters including hybrid / multi-cloud clusters, including Azure Arc-enabled Kubernetes clusters.

## Prerequisites

To deploy applications using Azure Fleet Manager, you need to have a Fleet, and join the Arc-Enabled Kubernetes cluster as a member.

### Azure Arc-enabled Kubernetes clusters

* An Azure Arc-enabled Kubernetes connected cluster that's up and running.

  [Learn how to connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md). If you need to connect through an outbound proxy, then assure you [install the Arc agents with proxy settings](./quickstart-connect-cluster.md?tabs=azure-cli#connect-using-an-outbound-proxy-server). Note that Azure Fleet Manager has specific [capability restrictions](#preview-limitations-and-capability-restrictions) when Arc-Enabled clusters are utilizing a proxy.

### Azure Fleet Manager & Joined Member(s)

* An Azure Fleet Manager with Hub Cluster.
  
  - Read the [official Azure Fleet Manager guides](https://docs.azure.cn/en-us/kubernetes-fleet/overview) to create your Azure Fleet Manager.
  - Ensure your Fleet is Hubful, or [upgrade if needed](https://docs.azure.cn/en-us/kubernetes-fleet/upgrade-hub-cluster-type).
  - [Join the Arc-Enable Kubernetes cluster as a member](https://docs.azure.cn/en-us/kubernetes-fleet/quickstart-create-fleet-and-members?tabs=without-hub-cluster#join-member-clusters) to your Fleet.

### Preview limitations and capability restrictions

> [!IMPORTANT]
> Azure Kubernetes Fleet Manager extension for Azure Arc-enabled Kubernetes is currently in public preview. During preview, certain capabilities are not available for Arc-enabled clusters. Arc-enabled Kubernetes clusters have specific capability restrictions when used with Azure Fleet Manager. For a complete list of supported and unsupported features, see [Member cluster types](https://learn.microsoft.com/azure/kubernetes-fleet/concepts-member-cluster-types).

### Deploy applications using Azure Fleet Manager

Follow Azure Fleet Manager's [guide on creating a Multi-Cluster Resource Placement](https://docs.azure.cn/en-us/kubernetes-fleet/quickstart-resource-propagation?tabs=azure-cli) to deploy your workloads (applications) to the member clusters.

## Delete the extension

To remove the extension from your Arc-Enabled Kubernetes cluster simply unjoin the cluster from the Fleet, and any workloads (applications) created and managed by the Fleet Manager will be garbage collected automatically.

```azurecli
az fleet member delete -f <fleet-name> -g <member-resource-group> -n <member-name>
```