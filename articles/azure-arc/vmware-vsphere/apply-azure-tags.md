---
title: Apply Azure tags to VMware vCenter resources through Azure Arc-enabled VMware vSphere 
description: In this article, you learn how to apply Azure tags to Azure Arc-enabled VMware vSphere resources. 
ms.topic: how-to 
ms.date: 07/18/2025
ms.service: azure-arc
ms.subservice: azure-arc-vmware-vsphere
ms.author: v-gajeronika
author: Jeronika-MS
---

# Apply Azure tags to VMware vCenter resources through Azure Arc-enabled VMware vSphere

Tags are metadata elements that you can apply to your Azure resources. They are key-value pairs that help you identify resources based on settings that are relevant to your organization. You can use Azure tags to organize and manage your resources effectively, to set up automation and to govern your resources with tag-based policies. For recommendations on how to implement a tagging strategy, see [Resource naming and tagging decision guide](/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming-and-tagging-decision-guide?toc=%2Fazure%2Fazure-resource-manager%2Fmanagement%2Ftoc.json).

In this article, you learn how to apply Azure tags to Azure Arc-enabled VMware vSphere resources. You can apply Azure tags to all Azure Arc-enabled VMware vSphere resources such as Virtual machines, Resource Pools, Clusters, Hosts, VM templates, VM networks, vCenters, and its associated Azure Arc resource bridge. 

## Prerequisites

Before you apply Azure tags to an Azure Arc-enabled VMware vSphere resource, ensure you meet the following prerequisites:
- The VMware vCenter is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
- If you plan to apply Azure tags to a Virtual machine or a Resource Pool or a Cluster or a Host or a VM template or a VM network, ensure the resource is [enabled for management in Azure](browse-and-enable-vcenter-resources-in-azure.md).
- *Azure Arc VMware  VM Contributor* role or a custom Azure role with permissions to apply tags on the resources on which you plan to apply Azure tags.

## Apply Azure tags to VMware vSphere resources 

>[!Note]
>You can also apply Azure tags while you [create a new VMware vSphere managed Virtual Machine from Azure](create-virtual-machine.md).

To apply Azure tags to the VMware vSphere resources, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **VMware vCenters** and then select the VMware vCenter, which manages the resource you plan to apply Azure tags.
       To apply Azure tags to the VMware vCenter, navigate to the **Tags** blade in the VMware vCenter Azure resource. 
       To apply Azure tags to the associated Azure Arc resource bridge, navigate to its Azure resource from the **Overview** blade.
 
2. To apply tags to Virtual machines or a Resource Pool or a Cluster or a Host or VM templates or VM networks or Datastores, navigate to the specific resource through the dedicated inventory views under the **vCenter inventory**.

    :::image type="content" source="media/apply-azure-tags/vmware-vcenter.png" alt-text="Screenshot showing VMware vCenter." lightbox="media/apply-azure-tags/vmware-vcenter.png":::
 
3. Select the resource to which you want to apply Azure tags and then select **Tags**.
4. Enter the name and value in the fields respectively and then select **Apply**.
 
   :::image type="content" source="media/apply-azure-tags/tags.png" alt-text="Screenshot showing Tags screen." lightbox="media/apply-azure-tags/tags.png":::
        
For an Azure resource, you can view the applied tags under the **Tags** blade and also under the **Overview** blade. Use the **Box** icon next to a tag to list all the resources with the same tag. 

To delete the tags, use the **Bin** icon next to the tag key-value pair.

>[!Important]
>Azure tags applied on a VMware resource are accessible only from Azure and will not propagate to the VMware vSphere resource tags or custom attributes and vice versa.

## Next step

Review [Resource naming and tagging decision guide](/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming-and-tagging-decision-guide).

