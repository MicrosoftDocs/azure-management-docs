---
title: Troubleshooting Azure Linux Container Host for AKS kernel version issues
description: How to troubleshoot Azure Linux Container Host for AKS kernel version issues.
author: suhuruli
ms.author: suhuruli
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: troubleshooting
ms.date: 04/18/2023
# Customer intent: "As a DevOps engineer managing Azure Kubernetes Service, I want to troubleshoot and update outdated kernel versions on Azure Linux Container Host nodes, so that my clusters can benefit from the latest features and security improvements."
---

# Troubleshoot outdated kernel versions in Azure Linux Container Host node images

> [!IMPORTANT]
> Starting on **30 November 2025**, AKS will no longer support or provide security updates for Azure Linux 2.0. Starting on **31 March 2026**, node images will be removed, and you'll be unable to scale your node pools. Migrate to a supported Azure Linux version by [**upgrading your node pools**](/azure/aks/upgrade-aks-cluster) to a supported Kubernetes version or migrating to [`osSku AzureLinux3`](/azure/aks/upgrade-os-version). For more information, see [[Retirement] Azure Linux 2.0 node pools on AKS](https://github.com/Azure/AKS/issues/4988).

During migration or when adding new node pools to your Azure Linux Container Host, you may encounter issues with outdated kernel versions. [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) releases a new Azure Linux node-image every week, which is used for new node pools and as the starting image for scaling up. However, older node pools may not be updating their kernel versions as expected.

To check the KERNEL-VERSION of your node pools run:

```bash
kubectl get nodes -o wide
```

Then, compare the kernel version of your node pools with the latest kernel published on [packages.microsoft.com](https://packages.microsoft.com/cbl-mariner/).

## Symptom

A common symptom of this issue includes:
- Azure Linux nodes aren't using the latest kernel version.

## Causes

There are two primary causes for this issue: 
1. Automatic node-image upgrades weren't enabled when the node pool was created.
1. The base image that AKS uses to start clusters runs two weeks behind the latest kernel versions due to their rollout procedure.

## Solution

You can enable automatic upgrades using [GitHub Actions](/azure/aks/node-upgrade-github-actions) and reboot the nodes to resolve this issue.

### Enable automatic node-image upgrades by using Azure CLI

To enable automatic node-image upgrades when deploying a cluster from az-cli, add the parameter `--auto-upgrade-channel node-image`. 

```azurecli-interactive
az aks create --name testAzureLinuxCluster --resource-group testAzureLinuxResourceGroup --os-sku AzureLinux --auto-upgrade-channel node-image
```

### Enable automatic node-image upgrades by using ARM templates

To enable automatic node-image upgrades when using an ARM template you can set the [upgradeChannel](/azure/templates/microsoft.containerservice/managedclusters?tabs=bicep&pivots=deployment-language-bicep#managedclusterautoupgradeprofile) property in `autoUpgradeProfile` to `node-image`.

```json
    autoUpgradeProfile: {
      upgradeChannel: 'node-image'
    }
```

<!--### Enable automatic node-image upgrades by using Terraform

To enable automatic node-image upgrades when using a Terraform template, you can set the [automatic_channel_upgrade](https://registry.terraform.io/providers/hashicorp/azurerm/latest/docs/resources/kubernetes_cluster#automatic_channel_upgrade) property in `azurerm_kubernetes_cluster` to `node-image`.

```json
    resource "azurerm_kubernetes_cluster" "example" {
        name                = "example-azurelinuxaks1"
        [...]
        automatic_channel_upgrade = "node-image"
        [...]
    }
```
-->
### Reboot the nodes

When updating the kernel version, you need to reboot the node to use the new kernel version. We recommend that you set up the [kured daemonset](/azure/aks/node-updates-kured). [Kured](https://github.com/kubereboot/kured) to monitor your nodes for the `/var/run/reboot-required` file, drain the workload, and reboot the nodes.

## Workaround: Manual upgrades
If you need a quick workaround, you can manually upgrade the node-image on a cluster using [az aks nodepool upgrade](/azure/aks/node-image-upgrade#upgrade-a-specific-node-pool). This can be done by running 

```azurecli
az aks nodepool upgrade \
    --resource-group testAzureLinuxResourceGroup \
    --cluster-name testAzureLinuxCluster \
    --name myAzureLinuxNodepool \
    --node-image-only
```

## Next steps

If the preceding steps don't resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
