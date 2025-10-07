---
title: "Tutorial: Deploy applications using Azure Kubernetes Fleet Manager"
description: "This tutorial shows how to use Azure Kubernetes Fleet Manager to deploy applications across multiple Arc Clusters."
author: ealianis
ms.author: sehobbs
ms.date: 09/30/2025
ms.topic: tutorial
ms.custom:
  - template-tutorial
  - ignite-2025
# Customer intent: As a DevOps engineer, I want to deploy applications to multiple kubernetes clusters at scale and across hybrid environments.
---

# Tutorial: Deploy applications using Azure Kubernetes Fleet Manager

This tutorial shows how to use [Azure Kubernetes Fleet Manager](/azure/kubernetes-fleet/overview) to deploy applications across multiple clusters including hybrid / multi-cloud clusters, including Azure Arc-enabled Kubernetes clusters.

> [!NOTE]
> Azure Arc-Enabled Kubernetes clusters for Azure Kubernetes Fleet Manager is in public preview. For a complete list of requirements and limitations, see [Member cluster types](/azure/kubernetes-fleet/concepts-member-cluster-types). 

## Prerequisites

To deploy applications using Azure Kubernetes Fleet Manager, you need to have a Fleet, and join the Arc-Enabled Kubernetes cluster as a member.

### Azure Arc-enabled Kubernetes clusters

* An Azure Arc-enabled Kubernetes connected cluster that's up and running.

  [Learn how to connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md). If you need to connect through an outbound proxy, then assure you [install the Arc agents with proxy settings](./quickstart-connect-cluster.md?tabs=azure-cli#connect-using-an-outbound-proxy-server).


### Azure Kubernetes Fleet Manager & Member(s)

* An Azure Kubernetes Fleet Manager with Hub Cluster.
  
  - Read the [official Azure Kubernetes Fleet Manager guides](/azure/kubernetes-fleet/overview) to create your Azure Kubernetes Fleet Manager.
  - Ensure your Fleet is Hubful, or [upgrade if needed](/azure/kubernetes-fleet/upgrade-hub-cluster-type).
  - [Join the Arc-Enable Kubernetes cluster as a member](/azure/kubernetes-fleet/quickstart-create-fleet-and-members?tabs=without-hub-cluster#join-member-clusters) to your Fleet.


### Deploy applications using Azure Kubernetes Fleet Manager

Follow Azure Kubernetes Fleet Manager's [guide on creating a Multi-Cluster Resource Placement](/azure/kubernetes-fleet/quickstart-resource-propagation) to deploy your workloads (applications) to the member clusters.

## Delete the extension

To remove the extension from your Arc-Enabled Kubernetes cluster simply unjoin (delete) the member from the Fleet, and any workloads (applications) created and managed by the Fleet Manager will be garbage collected automatically.

```azurecli
az fleet member delete -f <fleet-name> -g <member-resource-group> -n <member-name>
```