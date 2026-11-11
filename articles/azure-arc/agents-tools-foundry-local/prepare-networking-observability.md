---
title: Install networking and observability components
description: "Learn how to install MetalLB and monitoring components for an Agents and Tools with Foundry Local deployment."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to prepare networking and observability for Agents and Tools with Foundry Local so that I can ensure secure connectivity and effective monitoring of my chat solution.
---

# Install networking and observability components

For your deployment, install networking and observability components by configuring MetalLB and setting up certificate and trust managers. This article is part of the deployment prerequisites checklist.

## Install the components
From the driver machine, install and configure MetalLB for the Azure Kubernetes Service (AKS) cluster enabled by Azure Arc and the observability dependency modules.

### Install MetalLB

Skip this step if MetalLB is installed and configured in the current AKS Arc cluster.

To install and [configure](/azure/aks/hybrid/deploy-load-balancer-cli) MetalLB, run the following commands from the driver machine:

```powershell
$lbName = "metallb"
$ipRange = ""   # <------ Provide the static IP address range that will be assigned to metalLB (format: CIDR format E.g. <IP address>-<IP address> or <IP address>/32)
$sub = "<Subscription GUID>"
$rg = "<Resource Group name>"
$k8scluster = "<AKS Arc cluster name>"
$resourceuri = "/subscriptions/$sub/resourceGroups/$rg/providers/Microsoft.Kubernetes/connectedClusters/$k8scluster"
az extension add -n k8s-runtime --upgrade
az k8s-runtime load-balancer enable --resource-uri $resourceuri
az k8s-runtime load-balancer create --load-balancer-name $lbName --resource-uri $resourceuri --addresses $ipRange --advertise-mode "ARP"
```

### Install certificate and trust manager extension

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
