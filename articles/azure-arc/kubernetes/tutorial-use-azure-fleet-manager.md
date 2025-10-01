---
title: "Tutorial: Deploy applications using Azure Fleet Manager"
description: "This tutorial shows how to use Azure Fleet Manager to deploy applications across multiple Arc Clusters."
ms.date: 09/30/2025
ms.topic: tutorial
ms.custom:
  - template-tutorial
# Customer intent: As a DevOps engineer, I want to deploy applications .... todo
---

## Prerequisites

To deploy applications using Azure Fleet Manager, you need to have a Fleet, and join the Arc-Enabled Kubernetes cluster as a member.

### Azure Arc-enabled Kubernetes clusters

* An Azure Arc-enabled Kubernetes connected cluster that's up and running.

  [Learn how to connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md). If you need to connect through an outbound proxy, then assure you [install the Arc agents with proxy settings](./quickstart-connect-cluster.md?tabs=azure-cli#connect-using-an-outbound-proxy-server).

#### Azure Fleet Manager & Joined Member(s)

* An Azure Fleet Manager with Hub Cluster.
  
  - Read the [official Azure Fleet Manager guides](https://docs.azure.cn/en-us/kubernetes-fleet/overview) to create your Azure Fleet Manager.
  - Ensure your Fleet is Hubful, or [upgrade if needed](https://docs.azure.cn/en-us/kubernetes-fleet/upgrade-hub-cluster-type).
  - [Join the Arc-Enable Kubernetes cluster as a member](https://docs.azure.cn/en-us/kubernetes-fleet/quickstart-create-fleet-and-members?tabs=without-hub-cluster#join-member-clusters) to your Fleet.

#### Network requirements

The Arc-Enabled Kubernetes cluster must have a networking topology which allows for egress traffic from services within the cluster to its Fleet hub cluster.
Depending upon your enterprise networking configuration, e.g., network proxy, you may experience issues. Please review Fleet's support 

## Deploy applications using Azure Fleet Manager

Follow Azure Fleet Manager's [guide on creating a Multi-Cluster Resource Placement](https://docs.azure.cn/en-us/kubernetes-fleet/quickstart-resource-propagation?tabs=azure-cli) to deploy your application (workloads) to the member clusters.

## Delete the extension

Simply unjoin the cluster from the Fleet, and any applications (workloads) created and managed by the Fleet will be garbage collected automatically.

```azurecli
az  k8s-extension delete -g <resource-group> -c <cluster-name> -n argocd -t managedClusters --yes
```

---

## Next steps

* File issues and feature requests on the [Azure/AKS repository](https://github.com/Azure/AKS/labels/extension%2Fargocd) and be sure to include the word "ArgoCD" in the description or title.
* Explore [AKS-Platform engineering code sample](https://github.com/Azure-Samples/aks-platform-engineering) which deploys OSS ArgoCD with Backstage and Cluster API Provider for Azure (CAPZ) or Crossplane.
