---
title: Azure Linux Container Host for AKS tutorial - Enable telemetry and monitoring for the Azure Linux Container Host
description: In this Azure Linux Container Host for AKS tutorial, you'll learn how to enable telemetry and monitoring for the Azure Linux Container Host.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content, innovation-engine
ms.topic: tutorial
ms.date: 04/06/2025
# Customer intent: "As a cloud administrator, I want to enable telemetry and monitoring for my Azure Linux Container Host cluster, so that I can ensure optimal performance and gain insights into the cluster's health."
---

# Tutorial: Enable telemetry and monitoring for your Azure Linux Container Host cluster

> [!div class="nextstepaction"]
> [Deploy and Explore](https://go.microsoft.com/fwlink/?linkid=2321847)

In this tutorial, part four of five, you'll set up Container Insights to monitor an Azure Linux Container Host cluster. You'll  learn how to: 

> [!div class="checklist"]
> * Enable monitoring for an existing cluster.
> * Verify that the agent is deployed successfully.
> * Verify that the solution is enabled.

In the next and last tutorial, you'll learn how to upgrade your Azure Linux nodes.

## Prerequisites

- In previous tutorials, you created and deployed an Azure Linux Container Host cluster. To complete this tutorial, you need an existing cluster. If you haven't done this step and would like to follow along, start with [Tutorial 1: Create a cluster with the Azure Linux Container Host for AKS](./tutorial-azure-linux-create-cluster.md).
- If you're connecting an existing AKS cluster to a Log Analytics workspace in another subscription, the Microsoft.ContainerService resource provider must be registered in the subscription with the Log Analytics workspace. For more information, see [Register resource provider](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider).
- You need the latest version of Azure CLI. Run `az --version` to find the version. If you need to install or upgrade, see [Install Azure CLI](/cli/azure/install-azure-cli).

## Enable monitoring

### Connect to your cluster

Before enabling monitoring, it's important to ensure you're connected to the correct cluster. The following command retrieves the credentials for your Azure Linux Container Host cluster and configures kubectl to use them:

```azurecli
az aks get-credentials --resource-group $RESOURCE_GROUP --name $CLUSTER_NAME
```

### Use a default Log Analytics workspace

The following step enables monitoring for your Azure Linux Container Host cluster using Azure CLI. In this example, you aren't required to precreate or specify an existing workspace. This command simplifies the process for you by creating a default workspace in the default resource group of the AKS cluster subscription. If one doesn't already exist in the region, the default workspace created will resemble the format *DefaultWorkspace-< GUID >-< Region >*. 

```azurecli
# Check if monitoring addon is already enabled
MONITORING_ENABLED=$(az aks show -g $RESOURCE_GROUP -n $CLUSTER_NAME --query "addonProfiles.omsagent.enabled" -o tsv)

if [ "$MONITORING_ENABLED" != "true" ]; then
  az aks enable-addons -a monitoring -n $CLUSTER_NAME -g $RESOURCE_GROUP
fi
```

### Option 2: Specify a Log Analytics workspace

In this example, you can specify a Log Analytics workspace to enable monitoring of your Azure Linux Container Host cluster. The resource ID of the workspace will be in the form `"/subscriptions/<SubscriptionId>/resourceGroups/<ResourceGroupName>/providers/Microsoft.OperationalInsights/workspaces/<WorkspaceName>"`. The command to enable monitoring with a specified workspace is as follows: ```az aks enable-addons -a monitoring -n $CLUSTER_NAME -g $RESOURCE_GROUP --workspace-resource-id <workspace-resource-id>```

## Verify agent and solution deployment

Run the following command to verify that the agent is deployed successfully.

```bash
kubectl get ds ama-logs --namespace=kube-system
```

The output should resemble the following example, which indicates that it was deployed properly:

<!-- expected_similarity=0.3 -->
```text
User@aksuser:~$ kubectl get ds ama-logs --namespace=kube-system
NAME       DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
ama-logs   3         3         3       3            3           <none>          3m22s
```

To verify deployment of the solution, run the following command:

```bash
kubectl get deployment ama-logs-rs -n=kube-system
```

The output should resemble the following example, which indicates that it was deployed properly:

<!-- expected_similarity=0.3 -->
```text
User@aksuser:~$ kubectl get deployment ama-logs-rs -n=kube-system
NAME           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE    AGE
ama-logs-rs    1         1         1            1            3h
```

## Verify solution configuration

Use the `aks show` command to find out whether the solution is enabled or not, what the Log Analytics workspace resource ID is, and summary information about the cluster.

```azurecli
az aks show -g $RESOURCE_GROUP -n $CLUSTER_NAME --query "addonProfiles.omsagent"
```

After a few minutes, the command completes and returns JSON-formatted information about the solution. The results of the command should show the monitoring add-on profile and resemble the following example output:

<!-- expected_similarity=0.3 -->
```JSON
{
  "config": {
    "logAnalyticsWorkspaceResourceID": "/subscriptions/xxxxx/resourceGroups/xxxxx/providers/Microsoft.OperationalInsights/workspaces/xxxxx"
  },
  "enabled": true
}
```

## Next steps

In this tutorial, you enabled telemetry and monitoring for your Azure Linux Container Host cluster. You learned how to: 

> [!div class="checklist"]
> * Enable monitoring for an existing cluster.
> * Verify that the agent is deployed successfully.
> * Verify that the solution is enabled.

In the next tutorial, you'll learn how to upgrade your Azure Linux nodes.

> [!div class="nextstepaction"]
> [Upgrade Azure Linux nodes](./tutorial-azure-linux-upgrade.md)