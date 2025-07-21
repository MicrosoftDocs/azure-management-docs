---
title:  Apply Azure tags to SCVMM resources through Azure Arc-enabled System Center Virtual Machine Manager
description: In this article, you learn how to apply Azure tags to Azure Arc-enabled SCVMM resources. 
ms.topic: how-to 
ms.date: 05/28/2025
ms.service: azure-arc
ms.subservice: azure-arc-scvmm
ms.author: jsuri
author: jyothisuri
---

# Apply Azure tags to SCVMM resources through Azure Arc-enabled SCVMM

Tags are metadata elements that you can apply to your Azure resources. They are key-value pairs that help you identify resources based on settings that are relevant to your organization. You can use Azure tags to organize and manage your resources effectively, to set up automation and to govern your resources with tag-based policies. For recommendations on how to implement a tagging strategy, see [Resource naming and tagging decision guide](/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming-and-tagging-decision-guide?toc=%2Fazure%2Fazure-resource-manager%2Fmanagement%2Ftoc.json).

In this article, you learn how to apply Azure tags to Azure Arc-enabled SCVMM resources. You can apply Azure tags to all Azure Arc-enabled SCVMM resources such as Virtual machines, VMM Clouds, VM templates, VM networks, SCVMM management server and its associated Azure Arc resource bridge.

## Prerequisites

Before you apply Azure tags to an Azure Arc-enabled SCVMM resource, ensure you meet the following prerequisites:
- The SCVMM management server is in a *Connected* state and its associated Azure Arc resource bridge is in a *Running* state.
- If you plan to apply Azure tags to a Virtual machine or a VMM Cloud or a VM template or a VM network, ensure the resource is [enabled for management in Azure](enable-scvmm-inventory-resources.md).
- *Azure Arc SCVMM VM Contributor* role or a custom Azure role with permissions to apply tags on the resources on which you plan to apply Azure tags.

## Apply Azure tags to SCVMM resources

>[!Note]
>You can also apply Azure tags while you [create a new SCVMM managed Virtual Machine from Azure](create-virtual-machine.md).

To apply Azure tags to the SCVMM resources, follow these steps:

1. Sign in to the [Azure portal](https://portal.azure.com/), go to **Azure Arc** > **SCVMM management server** and then select the SCVMM server which manages the resource you plan to apply Azure tags.
       To apply Azure tags to the SCVMM server, navigate to the **Azure tags** blade in the SCVMM server Azure resource. 
       To apply Azure tags to the associated Azure Arc resource bridge, navigate to its Azure resource from the **Overview** blade.
 
2. To apply tags to Virtual machines or VMM Clouds or VM templates or VM networks, navigate to the specific resource through the dedicated inventory views under the **SCVMM inventory**.

    :::image type="content" source="media/apply-azure-tags/management-server.png" alt-text="Screenshot showing SCVMM Management server.":::
 
3. Select the resource to which you want to apply Azure tags and then select **Tags**.
4. Enter the name and value in the fields respectively and then select **Apply**.
 
   :::image type="content" source="media/apply-azure-tags/tags.png" alt-text="Screenshot showing Tags screen.":::
        
For an Azure resource, you can view the applied tags under the **Tags** blade and also under the **Overview** blade. Use the **Box** icon next to a tag to list all the resources with the same tag. 

To delete the tags, use the **Bin** icon next to the tag key-value pair.

>[!Important]
>Azure tags applied on a SCVMM resource are accessible only from Azure and will not propagate to the SCVMM console resource tags or custom properties and vice versa.

## Next step

Review [Resource naming and tagging decision guide](/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming-and-tagging-decision-guide).

