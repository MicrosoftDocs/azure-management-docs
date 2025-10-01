---
title: Install Networking and Observability Components for Edge RAG Preview Enabled by Azure Arc
description: "Learn how to install and configure networking and observability components for Edge RAG deployment, including MetalLB and monitoring tools."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/20/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to prepare networking and observability for Edge RAG so that I can ensure secure connectivity and effective monitoring of my chat solution.
---

# Install networking and observability components for Edge RAG Preview enabled by Azure Arc

For your Edge RAG deployment, install networking and observability components by configuring MetalLB and setting up certificate and trust managers. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Install components for Edge RAG

From the driver machine, install and configure MetalLB for the Azure Arc Azure Kubernetes Service (AKS) cluster and the observability dependency modules.

1. **Install MetalLB**

   Skip this step if MetalLB is installed and configured in the current AKS Arc cluster.

   To install and [configure](/azure/aks/hybrid/deploy-load-balancer-cli) MetalLB, you can run the following commands on any of the cluster nodes in the Azure Local instance:

	```powershell
	$lbName = "metallb"
	$ipRange = ""   # <------ Provide the static IP address range that will be assigned to metalLB (format: CIDR format E.g. <IP address>-<IP address> or <IP address>/32)
	$sub = "<Subscription GUID>"
	$rg = "<Resource Group name>"
	$k8scluster = "<AKS Arc cluster name>"
	$resourceuri = "subscriptions/$sub/resourceGroups/$rg/providers/Microsoft.Kubernetes/connectedClusters/$k8scluster"
	az extension add -n k8s-runtime --upgrade
	az k8s-runtime load-balancer enable --resource-uri $resourceuri
	az k8s-runtime load-balancer create --load-balancer-name $lbName --resource-uri $resourceuri --addresses $ipRange --advertise-mode "ARP"
	```

1. **Install observability dependency modules**

   `Microsoft.iotoperations.platform` is a simple extension that installs certificate manager and trust manager modules. Run the following command to install the extension.

	```powershell
	$sub = "<Subscription GUID>"
	$rg = "<Resource Group name>"
	$k8scluster = "<AKS Arc cluster name>"
	az k8s-extension create -g $rg -c $k8scluster -t connectedClusters --scope cluster --name "cert-manager" --release-namespace "cert-manager" --release-train preview --extension-type "Microsoft.iotoperations.platform" --debug
	```

    if the `Microsoft.iotoperations.platform` extension isn't available in your region, use following steps to install the required certificate and trust manager.

	```powershell
    # Install Cert-Manager and Trust-Manager 

    kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.15.3/cert-manager.yaml --wait  
    helm repo add jetstack https://charts.jetstack.io --force-update  
    start-sleep -Seconds 20 
    helm upgrade trust-manager jetstack/trust-manager --install --namespace cert-manager --wait 
    ```

## Next step

> [!div class="nextstepaction"]
> [Configure DNS](prepare-dns.md)