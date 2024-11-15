---
title: Deploy virtual machines effectively using Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure can help you deploy cost-efficient VMs.
ms.date: 11/14/2024
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
ms.author: jenhayes
author: JnHs
---

# Deploy virtual machines effectively using Microsoft Copilot in Azure

Microsoft Copilot in Azure (preview) can help you deploy [virtual machines in Azure](/azure/virtual-machines/overview) that are efficient and effective. You can get suggestions for different options to save costs and choose the right type and size for your VMs.

When you ask Microsoft Copilot in Azure for help with a VM, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. For example, if you start from the **Create a virtual machine** page, Copilot in Azure will provide suggestions for the new VM you're creating. If you're starting from another page, and the context isn't clear, you'll be prompted to specify the VM for which you want assistance.

You can also ask Copilot in Azure for help troubleshooting VM deployment failures related to virtual machine capacity, allocation, or other issues.

Microsoft Copilot in Azure is designed to help you regardless of your level of expertise on VM configuration options such as pricing, scalability, availability, and size. In all cases, we recommend that you closely review the suggestions to confirm that they meet your needs.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

[!INCLUDE [preview-note](includes/preview-note.md)]

## Create cost-efficient VMs

Microsoft Copilot in Azure can guide you in suggesting different options to save costs as you deploy a virtual machine. If you're new to creating VMs, Microsoft Copilot in Azure can help you understand the best ways to reduce costs More experienced users can confirm the best ways to make sure VMs align with both use cases and budget needs, or find ways to make a specific VM size more cost-effective by enabling certain features that might help lower overall cost.

### Cost-efficient VM sample prompts

- "How do I reduce the cost of my virtual machine?"
- "Help me create a cost-efficient virtual machine"
- "Help me create a low cost VM"

### Cost-efficient VM examples

During the VM creation process, you can ask "**How do I reduce the cost of my virtual machine?**", or select the **Help me create a low-cost VM** button near the top of the pane. Microsoft Copilot in Azure guides you through options to make your VM more cost-effective. You can choose whether or not to enable each option.

:::image type="content" source="media/deploy-vms-effectively/vm-reduce-costs.png" lightbox="media/deploy-vms-effectively/vm-reduce-costs.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing ways to lower VM costs.":::

Once you complete the options that Microsoft Copilot in Azure suggests, you can review and create the VM with the provided recommendations, or continue to make other changes.

:::image type="content" source="media/deploy-vms-effectively/vm-reduce-costs-complete.png" lightbox="media/deploy-vms-effectively/vm-reduce-costs-complete.png" alt-text="Screenshot showing Microsoft Copilot in Azure completing its recommendations to reduce VM costs.":::

## Create highly available and scalable VMs

Microsoft Copilot in Azure can provide additional context to help you create high-availability VMs. It can help you create VNs in availability zones, decide whether a Virtual Machine Scale Set is the right option for your needs, or assess which networking resources will help manage traffic effectively across your compute resources.

### High-availability VM sample prompts

- "How do I create a resilient virtual machine?"
- "Help me create a high availability virtual machine"

### High-availability VM examples

During the VM creation process, you can ask "**How do I create a resilient and high availability virtual machine?**", or select the **Help me create a VM optimized for high availability** button near the top of the pane. Microsoft Copilot in Azure guides you through options to configure your VM for high availability, providing options that you can enable.

:::image type="content" source="media/deploy-vms-effectively/vm-resilient-high-availability.png" lightbox="media/deploy-vms-effectively/vm-resilient-high-availability.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing ways to configure a VM for high availability.":::

## Choose the right size for your VMs

Azure offers several size options based on your workload needs. Microsoft Copilot in Azure can help you identify the best VM size based on your scenario needs and assess it in the context of your other configuration requirements.  

When you share your intent and scenario, Copilot helps you narrow down the right VM size from the vast size selection offered by Azure today. Copilot helps you identify requirements without requiring you to know the exact amount of RAM or CPU that you need. Size recommendations also take any quota and offer restrictions into account and surface the most intelligent recommendations pertinently. Copilot can also help you create a support request to increase a quota if desired.

While familiarity with the size options can be beneficial, Copilot in Azure is designed to assist users ranging across different expertise levels in achieving their deployment goals. However, it is crucial that you exercise due diligence with the suggested options.

### VM size sample prompts

- "Help me choose a size for my virtual machine"
- "Which VM size will best suit my requirements?"

### VM size examples

Ask "**Help me choose a size for my VM,**", or select the **Help me choose the right VM size for my workload** button near the top of the pane. Microsoft Copilot in Azure asks for some more information to help it determine the best options.

:::image type="content" source="media/deploy-vms-effectively/vm-choose-size.png" alt-text="Screenshot showing Microsoft Copilot in Azure asking for details to help determine the appropriate VM size.":::

After that, Copilot in Azure presents some options and lets you choose which of the recommended sizes to use for your VM.

:::image type="content" source="media/deploy-vms-effectively/vm-choose-size-options.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing size recommendations for a VM.":::

## Copy VMs into any region

Microsoft Copilot in Azure can help you replicate existing VMs in the region of your choice. If the selected region has a low probability of deployment success, Copilot in Azure suggests alternate recommendations for the region, VM size, or availability zone. Additionally, Copilot showcases existing properties that are copied to the new VM from the existing VM, lets you know if any properties aren't supported and can't be copied over, and guides you to a prefilled create experience where you can deploy VMs quickly and enable microproductivity gains.

Copilot can also advise you of scenarios where existing related services may not be easily attached to the new VM copy. For instance, if address space gets exhausted by reusing the existing VM's virtual network or subnet for new Virtual Machine copy, you'll be prompted to adjust network settings appropriately. Similarly, if the backend pool of an existing load balancer linked with the existing VM is exhausted, you're notified and will be able to update your load balancer/application gateway configuration settings.

When using Copilot in Azure to help you copy VMs, keep in mind the following:

- The cost of the new VM may differ from the existing VM, depending on licensing and usage implications.
- Licensing and password credentials aren't copied automatically. You can configure these during the create experience.
- You can't use Copilot to copy an existing VM if it has errors or is in a failed state.
- Data disks and OS disks can't be reused or copied, but the configuration of the OS disks and data disks will be copied over (if present in the existing VM).

While familiarity with the pricing information of different VM configurations can be beneficial, Copilot in Azure is designed to assist users ranging across different expertise levels in achieving their deployment goals. However, it is crucial that you exercise due diligence with the suggested options.

### Copy VM sample prompts

- "Help me copy this VM"
- "Help me copy this VM in the same region"
- "Help me copy this VM to East US"
- "Help me copy this VM to any region"

### Copy VM example

You can say **"Help me copy VMs in any region."** Microsoft Copilot in Azure prompts you to select a region and a name for the new VM, then begins the create experience.

:::image type="content" source="media/deploy-vms-effectively/vm-copy-region.png" lightbox="media/deploy-vms-effectively/vm-copy-region.png"alt-text="Screenshot of Microsoft Copilot for Azure responding to a request to copy a VM.":::

## Troubleshoot VM deployment failures

If you experience failures when trying to deploy a VM, Microsoft Copilot in Azure can help diagnose and resolve issues related to virtual machine capacity, allocation, or other deployment failures that may arise due to configuration issues or limited availability. Copilot in Azure provides real-time health diagnostics on resources and determines potential successful paths for capacity availability and overall deployment success.

### Troubleshooting sample prompts

- "Help me troubleshoot my VM deployment failure"
- "Troubleshoot my VM allocation failure issue for this deployment"
- "My VM failed to provision with an error, can you help troubleshoot?"

### Troubleshooting example

If a VM deployment fails, you can say "**troubleshoot my VM deployment failure**". After confirming the correct resource, Microsoft Copilot in Azure analyzes recent operations and suggest changes (such as selecting a new size, zone, or region) that have available capacity based on the prior deployment.

:::image type="content" source="media/deploy-vms-effectively/vm-troubleshoot-deployment.png" lightbox="media/deploy-vms-effectively/vm-troubleshoot-deployment.png" alt-text="Screenshot of Microsoft Copilot for Azure responding to a request to troubleshoot VM deployment failure.":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [virtual machines in Azure](/azure/virtual-machines/overview).
