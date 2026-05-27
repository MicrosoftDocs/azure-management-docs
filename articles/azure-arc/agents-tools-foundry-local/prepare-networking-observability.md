---
title: Install Networking and Observability Components for Agents and Tools with Foundry Local
description: "Learn how to install and configure networking and observability components for Agents and Tools with Foundry Local deployment, including MetalLB and monitoring tools."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/20/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to prepare networking and observability for Agents and Tools with Foundry Local so that I can ensure secure connectivity and effective monitoring of my chat solution.
---

# Install networking and observability components for Agents and Tools with Foundry Local

For your Agents and Tools with Foundry Local deployment, install networking and observability components by configuring MetalLB and setting up certificate and trust managers. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Install components for Agents and Tools with Foundry Local

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

   Install the certificate manager and trust manager as an Azure Arc extension. This installation includes the trust-manager parameters required for Foundry Local certificate handling.

	```powershell
	$sub = "<Subscription GUID>"
	$rg = "<Resource Group name>"
	$k8scluster = "<AKS Arc cluster name>"
	az k8s-extension create `
	  --cluster-name $k8scluster `
	  --name "azure-cert-manager" `
	  --resource-group $rg `
	  --cluster-type connectedClusters `
	  --extension-type Microsoft.CertManagement `
	  --scope cluster `
	  --release-train stable `
	  --config config.enableGatewayAPI=true `
	  --config cert-manager.crds.keep=true `
	  --config trust-manager.defaultPackage.enabled=false `
	  --config trust-manager.secretTargets.enabled=true `
	  --config trust-manager.secretTargets.authorizedSecretsAll=true
	```

   > [!IMPORTANT]
   > The `trust-manager.secretTargets.enabled` and `trust-manager.secretTargets.authorizedSecretsAll` parameters are required for Foundry Local to properly manage TLS certificates. Installing cert-manager without these parameters causes SSL certificate verification failures.

## Next step

> [!div class="nextstepaction"]
> [Configure DNS](prepare-dns.md)