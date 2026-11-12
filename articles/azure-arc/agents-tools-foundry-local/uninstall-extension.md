---
title: Uninstall Agents and Tools with Foundry Local Extension
description: "Learn how to uninstall the Agents and Tools with Foundry Local extension by using Azure PowerShell or the Azure portal and remove the associated namespace step by step."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a Kubernetes cluster administrator, I want to uninstall the Agents and Tools with Foundry Local extension so that I can remove the extension and clean up the associated namespace from my cluster.
---

# Uninstall extension for Agents and Tools with Foundry Local

This article explains how to remove the Agents and Tools with Foundry Local extension and delete the associated namespace from your cluster.

## Delete the Agents and Tools with Foundry Local extension

Delete the extension by using Azure PowerShell or the Azure portal.

#### [Azure PowerShell](#tab/azure-powershell)


1. For any location where `kubectl` is configured, such as your dev box or personal laptop, run the following code to sign in to Azure CLI and set it to the correct cluster:

    ```powershell
    az login --use-device-code --tenant <tenant ID>

    $sub = "<Subscription GUID>"
    $resourceGroup = "<Resource Group>"
    $clusterName = "<Azure Kubernetes Service (AKS) Arc cluster name>"
    $localextname = "<extension-name>"

    az account set -s $sub
    az aksarc get-credentials --name $clusterName --resource-group $resourceGroup --admin --only-show-errors
    ```

1. Run the following command to delete the extension:

    ```powershell
    az k8s-extension delete --cluster-name $clusterName --cluster-type connectedClusters --resource-group $resourceGroup --name $localextname --debug --yes
    ```
    
#### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com/), go to the Azure Kubernetes service on Azure Local cluster where the extension is deployed.
1. Under **Settings**, go to the **Extensions** tab.
1. In the list of installed extensions, find and select the extension with type **microsoft.arc.rag**.
1. Select the ellipses (**...**) and choose **Uninstall**.
1. Wait for the extension deletion notification.

----

Ensure the Agents and Tools with Foundry Local extension is successfully deleted and is no longer in the list of extensions on the cluster.

## Delete the namespace

Deleting the `arc-rag` namespace removes any retained PersistentVolumeClaims (PVCs) and Kubernetes Secrets in that namespace. For a full cleanup, also remove the `model-access-arc-rag` and `mcp-access-arc-rag` namespaces that the extension creates.

From the directory where you downloaded `kubectl`, run the following commands to delete the namespaces:

```powershell
kubectl delete namespace arc-rag
kubectl delete namespace model-access-arc-rag --ignore-not-found
kubectl delete namespace mcp-access-arc-rag --ignore-not-found
```

## Related content

- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
- [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
