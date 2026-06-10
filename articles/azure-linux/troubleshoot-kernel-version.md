---
title: Troubleshoot Outdated Kernel Versions in Azure Linux Container Host Node Images
description: Guidance on troubleshooting issues where Azure Linux Container Host node images in AKS are running outdated kernel versions, including identifying symptoms, understanding causes, and applying solutions.
author: kavyamsft
ms.author: schaffererin
ms.service: microsoft-linux
ms.custom: linux-related-content
ms.topic: troubleshooting-general
ms.date: 04/28/2026
---


# Troubleshoot outdated kernel versions in Azure Linux Container Host for Azure Kubernetes Service (AKS) node images

During migration or when adding new node pools to your Azure Linux Container Host, you might encounter issues with outdated kernel versions. [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes) releases a new Azure Linux node image every week, which is used for new node pools and as the starting image for scaling up. However, older node pools might not be updating their kernel versions as expected.

This article provides guidance on how to troubleshoot issues where Azure Linux Container Host node images in AKS are running outdated kernel versions, including identifying the symptoms, understanding the underlying causes, and applying recommended solutions to ensure that your nodes are running the latest kernel versions.

## Verify the kernel version of your node pools

1. Check the kernel version of your node pools using the following command:

    ```bash
    kubectl get nodes -o wide
    ```

1. Compare the kernel version of your node pools with the latest kernel published on [packages.microsoft.com](https://packages.microsoft.com/azurelinux/).

## Symptom

A common symptom of this issue is that the Azure Linux nodes aren't using the latest kernel version.

## Causes

You might encounter outdated kernel versions on Azure Linux Container Host node images in AKS due to one or both of the following reasons:

- Automatic node-image upgrades weren't enabled when the node pool was created.
- The base image that AKS uses to start clusters runs two weeks behind the latest kernel versions due to their rollout procedure.

## Solution

You can enable automatic upgrades using [GitHub Actions](/azure/aks/node-upgrade-github-actions) and reboot the nodes.

### Enable automatic AKS node image upgrades

#### [Azure CLI](#tab/azure-cli)

Enable automatic node image upgrades on a new AKS cluster using the [`az aks create`](/cli/azure/aks#az-aks-create) command with the parameter `--auto-upgrade-channel node-image`. Replace the placeholder values with your own values.

```azurecli-interactive
az aks create --name <cluster-name> --resource-group <resource-group-name> --os-sku AzureLinux --auto-upgrade-channel node-image
```

#### [ARM template](#tab/arm-template)

Enable automatic node image upgrades in an Azure Resource Manager (ARM) template by setting the [`upgradeChannel`](/azure/templates/microsoft.containerservice/managedclusters?tabs=bicep&pivots=deployment-language-bicep#managedclusterautoupgradeprofile) property in `autoUpgradeProfile` to `node-image`. For example:

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

---

### Reboot the nodes

When updating the kernel version, you need to reboot the node to use the new kernel version. We recommend that you set up the [kured daemonset](/azure/aks/node-updates-kured). You can use [Kured](https://github.com/kubereboot/kured) to monitor your nodes for the `/var/run/reboot-required` file, drain the workload, and reboot the nodes.

### Workaround: Manual upgrades

If you need a quick workaround, you can manually upgrade the node image on an AKS cluster using the [`az aks nodepool upgrade`](/cli/azure/aks#az-aks-nodepool-upgrade) command. For example:

```azurecli-interactive
az aks nodepool upgrade \
    --resource-group testAzureLinuxResourceGroup \
    --cluster-name testAzureLinuxCluster \
    --name myAzureLinuxNodePool \
    --node-image-only
```

## Next steps

If the preceding steps don't resolve the issue, open a [support ticket](https://azure.microsoft.com/support/).
