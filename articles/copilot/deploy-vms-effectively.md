---
title: Deploy and manage virtual machines effectively using Microsoft Copilot in Azure
description: Learn how Microsoft Copilot in Azure can help you deploy cost-efficient VMs.
ms.date: 07/13/2025
ms.topic: how-to
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
  - ignite-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As a cloud administrator, I want to deploy and manage cost-efficient and scalable virtual machines using guided assistance, so that I can optimize resource allocation and ensure high availability for my workloads.
---

# Deploy and manage virtual machines effectively using Microsoft Copilot in Azure

Microsoft Copilot in Azure can help you deploy [virtual machines in Azure](/azure/virtual-machines/overview) that are efficient and effective. You can get suggestions for different options to save costs and choose the right size for your VMs.

When you ask Microsoft Copilot in Azure for help with a VM, it automatically pulls context when possible, based on the current conversation or on the page you're viewing in the Azure portal. For example, if you start from the **Create a virtual machine** page, Copilot in Azure will provide suggestions for the new VM you're creating. If you're starting from another page, and the context isn't clear, you'll be prompted to specify the VM for which you want assistance.

You can ask Copilot in Azure for help with troubleshooting VM deployment failures, network security issues, or other errors.

Microsoft Copilot in Azure is designed to help you regardless of your level of expertise on VM configuration options such as pricing, scalability, availability, and size. In all cases, we recommend that you closely review the suggestions to confirm that they meet your needs.

[!INCLUDE [scenario-note](includes/scenario-note.md)]

## Create cost-efficient VMs

Microsoft Copilot in Azure can guide you in suggesting different options to save costs as you deploy a virtual machine. If you're new to creating VMs, Microsoft Copilot in Azure can help you understand the best ways to reduce costs More experienced users can confirm the best ways to make sure VMs align with both use cases and budget needs, or find ways to make a specific VM size more cost-effective by enabling certain features that might help lower overall cost.

### Cost-efficient VM sample prompts

- "How do I reduce the cost of my virtual machine?"
- "Help me create a cost-efficient virtual machine"
- "Help me create a low cost VM"

### Cost-efficient VM examples

During the VM creation process, you can ask "**How do I reduce the cost of my virtual machine?**", or select the **Help me create a low-cost VM** button near the top of the pane. Microsoft Copilot in Azure guides you through options to make your VM more cost-effective. You can choose whether or not to enable each option.

:::image type="content" source="media/deploy-vms-effectively/vm-reduce-costs.png" lightbox="media/deploy-vms-effectively/vm-reduce-costs.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing ways to lower VM costs.":::

Once you complete the options that Microsoft Copilot in Azure suggests, you can review and create the VM with the provided recommendations, or continue to make other changes. The **Estimated monthly cost** pane shows updates based on your configuration.

:::image type="content" source="media/deploy-vms-effectively/vm-reduce-costs-complete.png" lightbox="media/deploy-vms-effectively/vm-reduce-costs-complete.png" alt-text="Screenshot showing Microsoft Copilot in Azure completing its recommendations to reduce VM costs.":::

## Create highly available and scalable VMs

Microsoft Copilot in Azure can provide additional context to help you create high-availability VMs. It can help you create VNs in availability zones, decide whether a Virtual Machine Scale Set is the right option for your needs, or assess which networking resources will help manage traffic effectively across your compute resources.

### High-availability VM sample prompts

- "How do I create a resilient virtual machine?"
- "Help me create a high availability virtual machine"

### High-availability VM examples

During the VM creation process, you can ask "**How do I create a resilient and high availability virtual machine?**", or select the **Help me create a VM optimized for high availability** button near the top of the pane. Microsoft Copilot in Azure guides you through options to configure your VM for high availability, providing options that you can enable.

:::image type="content" source="media/deploy-vms-effectively/vm-resilient-high-availability.png" lightbox="media/deploy-vms-effectively/vm-resilient-high-availability.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing ways to configure a VM for high availability.":::

## Choose the right compute offering

When you're not sure exactly which compute offering to choose, Copilot in Azure can guide you to successfully identify and deploy the right type of resource (such as [Virtual Machine Scale Sets](/azure/virtual-machine-scale-sets/overview) or [Azure Compute Fleet](/azure/azure-compute-fleet/overview)) for your workload needs. Copilot in Azure can also help direct you to the right service if you want to assess how to integrate your on-premises compute solutions into Azure or migrate from another cloud platform.

If you're migrating compute instances from other cloud platforms, Copilot in Azure can help determine the equivalent Azure size family for those resources.

While familiarity with the product options can be beneficial, Copilot in Azure is designed to assist you regardless of your expertise level. However, it's important to exercise due diligence before deploying the suggested options.

### Compute offering sample prompts

- "Help me choose the right Compute infrastructure service"
- "I'd like to use a compute resource for a batch processing job, which one should I pick?"

## Choose the right size for your VMs

Azure offers several size options based on your workload needs. Microsoft Copilot in Azure can help you identify the best VM size based on your scenario needs and assess it in the context of your other configuration requirements.  

When you share your intent and scenario, Copilot helps you narrow down the right VM size from the vast size selection offered by Azure today. Copilot helps you identify requirements without requiring you to know the exact amount of RAM or CPU that you need. Size recommendations also take any quota and offer restrictions into account and surface the most intelligent recommendations pertinently. Copilot can also help you create a support request to increase a quota if desired.

While familiarity with the size options can be beneficial, Copilot in Azure is designed to assist you regardless of your expertise level. However, it's crucial that you exercise due diligence with the suggested options.

### VM size sample prompts

- "Help me choose a size for my virtual machine"
- "Which VM size will best suit my requirements?"

### VM size examples

Ask "**Help me choose a size for my VM,**", or select the **Help me choose the right VM size for my workload** button near the top of the pane. Copilot in Azure asks for some more information to help it determine the best options.

:::image type="content" source="media/deploy-vms-effectively/vm-choose-size.png" alt-text="Screenshot showing Microsoft Copilot in Azure asking for details to help determine the appropriate VM size.":::

After that, Copilot in Azure presents some options and lets you choose which of the recommended sizes to use for your VM.

:::image type="content" source="media/deploy-vms-effectively/vm-choose-size-options.png" alt-text="Screenshot showing Microsoft Copilot in Azure providing size recommendations for a VM.":::

## Copy VMs into any region

Microsoft Copilot in Azure can help you replicate existing VMs in the region of your choice. If the selected region has a low probability of deployment success, Copilot in Azure suggests alternate recommendations for the region, VM size, or availability zone. Additionally, Copilot showcases existing properties that are copied to the new VM from the existing VM, lets you know if any properties aren't supported and can't be copied over, and guides you to a prefilled creation experience where you can deploy VMs quickly and enable microproductivity gains.

Copilot can also advise you of scenarios where existing related services may not be easily attached to the new VM copy. For instance, if address space gets exhausted by reusing the existing VM's virtual network or subnet for new Virtual Machine copy, you'll be prompted to adjust network settings appropriately. Similarly, if the backend pool of an existing load balancer linked with the existing VM is exhausted, you're notified and will be able to update your load balancer/application gateway configuration settings.

When using Copilot in Azure to help you copy VMs, keep in mind the following:

- The cost of the new VM may differ from the existing VM, depending on licensing and usage implications.
- Licensing and password credentials aren't copied automatically. You can configure these during the creation experience.
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

### Troubleshoot deployment sample prompts

- "Help me troubleshoot my VM deployment failure"
- "Troubleshoot my VM allocation failure issue for this deployment"
- "My VM failed to provision with an error, can you help troubleshoot?"

### Troubleshoot deployment example

If a VM deployment fails, you can say "**troubleshoot my VM deployment failure**". After confirming the correct resource, Microsoft Copilot in Azure analyzes recent operations and suggests items to review or new selections you can make to better support your deployment.

:::image type="content" source="media/deploy-vms-effectively/vm-troubleshoot-deployment.png" alt-text="Screenshot of Microsoft Copilot for Azure responding to a request to troubleshoot VM deployment failure.":::

## Troubleshoot VM errors

Copilot in Azure offers analysis and guidance for troubleshooting VM errors caused by issues with configuration, capacity, quota, or other constraints. When you ask for help troubleshooting errors, Copilot in Azure collects details about recent operations, status, and configuration, and determines the appropriate troubleshooting steps.

### Troubleshoot VM error sample prompts

- "Explain the most recent operation failure on my VM."
- "Can you explain the OSProvisioningTimedOut error on my VM?"
- "Why did my virtual machine fail with VMExtensionProvisioningError?"

### Troubleshoot VM error example

If you see an error message in the Azure portal, you can ask "**Can you explain the OSProvisioningTimedOut error on my VM?**" Copilot in Azure analyzes the error and explains the problem, along with suggesting possible resolutions.

:::image type="content" source="media/deploy-vms-effectively/vm-troubleshoot-error.png" lightbox="media/deploy-vms-effectively/vm-troubleshoot-error.png" alt-text="Screenshot of Copilot in Azure explaining a VM error and how to resolve it.":::

## Troubleshoot VM network security issues

If your VM has connectivity problems due to network security groups blocking traffic, Copilot in Azure can help you understand the issue. When applicable, Copilot in Azure can guide you through the process of creating a new security rule in your network security group to open the necessary ports.

To get help with security rules, start from the virtual machine you want to troubleshoot and select **Connect**. Under **Troubleshoot**, select **Test network security groups**. After running a test, you may see that connections are blocked due to security rules. If so, open Copilot in Azure and use one of the provided prompts to get assistance.

:::image type="content" source="media/deploy-vms-effectively/vm-network-security-rule.png" lightbox="media/deploy-vms-effectively/vm-network-security-rule.png" alt-text="Screenshot of Copilot in Azure providing assistance with network security issues.":::

## Next steps

- Explore [capabilities](capabilities.md) of Microsoft Copilot in Azure.
- Learn more about [virtual machines in Azure](/azure/virtual-machines/overview).
