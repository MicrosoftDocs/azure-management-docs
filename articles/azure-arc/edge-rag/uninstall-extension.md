---
title: Uninstall Edge RAG Extension
description: "Learn how to uninstall the Edge RAG extension by using Azure PowerShell or the Azure portal and remove the associated namespace step by step."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a Kubernetes cluster administrator, I want to uninstall the Edge RAG extension so that I can remove the extension and clean up the associated namespace from my cluster.
---

# Uninstall extension for Edge RAG Preview, enabled by Azure Arc

This article covers how to remove the Edge RAG extension and delete the associated namespace from your cluster.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Delete the Edge RAG Extension

Delete the extension by using Azure PowerShell or from the Azure portal.

#### [Azure PowerShell](#tab/azure-powershell)


1. For anywhere `kubectl` is configured such as your devbox or personal laptop run the following code to sign in into Azure CLI and set it to the right cluster:

    ```powershell
    az login --use-device-code --tenant <tenant ID>
    
    $sub="<Subscription GUID>"
    $resourceGroup="<Resource Group>"
    $clusterName="<Azure Kubernetes Service (AKS) Arc cluster name>"
    
    az account set -s $sub
    az aksarc get-credentials --name $clusterName --resource-group $resourceGroup --admin --only-show-errors
    ```
1. Run the following command to delete the extension:

    ```powershell
    az k8s-extension delete --cluster-name $clusterName  --cluster-type connectedClusters--resource-group $resourceGroup --name $localextname --debug --yes
    ```
    
#### [Azure portal](#tab/azure-portal)

1. In the [Azure portal](https://portal.azure.com/), go to the Azure Kubernetes service on Azure Local cluster where the extension is deployed.
1. Under **Settings**, go to the **Extensions** tab.
1. In the list of installed extensions, find and select the extension with type **microsoft.arc.rag**.
1. Select the ellipses (**...**) and choose **Uninstall**.
1. Wait for the extension deletion complete notification.

----

Make sure the Edge RAG extension is successfully deleted and is no longer in the list of extensions on the cluster.

## Delete the namespace

From the directory where `kubectl` is downloaded, run the following command to delete the namespace:

```powershell
kubectl delete namespace arc-rag 
```

## Related content

- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
- [Deploy the Edge RAG extension](deploy.md)
