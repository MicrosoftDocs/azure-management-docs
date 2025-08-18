---
title: Next steps for cloud-native server management with Azure Arc-enabled servers
description: Learn how to transition your server management into a cloud-native model using Azure Arc-enabled servers. 
ms.date: 08/01/2025
ms.topic: overview
# Customer intent: "As a system architect managing a hybrid cloud environment, I want to understand how start moving from traditional on-premises management to a cloud-native model by using the management capabilities of Azure Arc-enabled servers."
---

# Next steps for cloud-native server management with Azure Arc-enabled servers

Transitioning from traditional on-premises management to a cloud-native approach is often a gradual process. It’s a journey, but one that leads to more consistent and scalable management. [Azure Arc-enabled servers](../overview.md) has the flexibility to help you along the way.

Evolve your server management approach by following the approach described in this article. You'll gradually migrate from being an "on-premises system administrator" to a "cloud-era system administrator." The core skills and knowledge that you use today remain as valuable as ever, but now Azure's powerful tools will help you do the job more efficiently.

## Hybrid enablement

Begin by [onboarding a subset of your servers to Azure Arc](../plan-at-scale-deployment.md). There are [different ways you can deploy the Connected Machine agent](../deployment-options.md), depending on your requirements and the tools you prefer to use. For example, you can [connect machines from Windows Admin Center](../onboard-windows-admin-center.md) or through a [Configuration Manager script](../onboard-configuration-manager-powershell.md) or [custom task sequence](../onboard-configuration-manager-custom-task.md). Alternately, you can [generate a script in the Azure portal and deploy it to individual machines](../onboard-portal.md).

Consider how to make use of [Azure's resource organization](inventory-resource.md) and figure out what approach makes sense for you. For example, you can group servers that have similar needs together in the same resource groups, and use consistent tagging conventions to designate, environment, location, or other considerations. A well-planned resource hierarchy will pay off in clarity and policy application.

## Iterative adoption of services

When starting your cloud-native journey, you don't have to change everything at once. You have the flexibility to make changes that work for you on your own timeframe and refine new processes to ensure they improve productivity and meet your organization's needs.

For example, you might start using [Azure Update Manager](/azure/update-manager/overview) for patching, even if you keep using Active Directory Group Policy Objects (GPOs) for configurations. Later, you might introduce [Azure machine configuration](/azure/governance/machine-configuration/) for a few critical settings, to help get familiar with compliance reporting in Azure. After that, you can explore using [Azure Run Command](/azure/virtual-machines/run-command-overview) for routine tasks on a designated set of servers, then expand more broadly when you're ready. By designing a phased approach, your team can learn and build confidence.

You might even decide that it makes sense to maintain overlapping solutions during your transition period. For example, you might keep Windows Server Update Services (WSUS) running as a backup while you confirm that that Azure Update Manager covers all patches. Another example is continuing to use GPO enforcement, but start using Azure Policy in audit mode to double-check compliance. This approach lets you compare results with your existing tools and helps you feel confident that nothing will fall through the cracks while you shift management planes. After you feel confident that your new cloud-based processes meet your needs, you can phase out redundant systems to reduce cost and complexity.

## Automation and environment management

Look for ways to automate common tasks. For example, you can use Azure Policy to automatically apply tags when your servers meet certain requirements. You can create scripts that automatically onboard new servers to Azure Arc by installing the Connected Machine agent as part of server provisioning. These types of automations reduce manual work and ensure consistency.

Leverage Azure's strengths by treating your Arc-managed servers with the same security rigor available for Azure VMs. By using [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/built-in-roles), you can ensure that only the right people can perform actions on your Arc-enabled servers, following the principle of least privilege. Microsoft Defender for Cloud provides recommendations for your Arc-enabled servers to improve your security posture, helping you address issues such as missing updates or vulnerable configurations. Many of these recommendations can be addressed immediatly by applying an Azure Policy or script. Azure Activity logs record actions and provide an audit trail.

To ensure the Azure Connected Machine agent (and dependent agents like Azure Monitor agent) remain healthy on each server, you can create alerts that notify you when an Arc-enabled server is unreachable, or if an extension has failed.

## Team training

As you refine your processes, document your approach and any necessary steps for your team. For example, when you establish a regular patching routine by using Azure Update Manager, you can create custom guidance letting team members understand how to schedule an on-demand patch run when needed.

Your team may also need to get familiar with using the Azure portal and Azure CLI, as well as new concepts such as Azure role-based access control (Azure RBAC) and Azure Resource Graph queries. Invest time in training or hands-on labs for your team. Microsoft offers an extensive catalog of [free training for Azure services](/training/azure/), including specific learning paths for Azure Arc, such as [Bring innovation to your hybrid environments with Azure Arc](/training/paths/manage-hybrid-infrastructure-with-azure-arc/) and [Deploy and manage Azure Arc-enabled servers](/training/paths/deploy-manage-azure-arc-enabled-servers/).

## Community resources

Azure Arc is still evolving, with new capabilities rolled out regularly. Staying informed helps you explore new approaches and try out preview features to optimise your cloud management road map.

Follow the [Azure Arc blog](https://techcommunity.microsoft.com/category/azure/blog/azurearcblog) to stay updated on the latest features and news related to Azure Arc. You can also view [Azure Updates](https://azure.microsoft.com/updates/) to get the latest updates on all Azure products and features.

When you have questions about using Azure, you can get support from the community through [Microsoft Q&A](/answers/tags/146/azure-arc). As you build your expertise, you can also share your knowledge here to help others.

The [Azure Arc Jumpstart](https://azurearcjumpstart.io/) provides a virtual, hybrid sandbox environment, letting you explore different capabilities and scenarios. New features and updates are regularly added, so it's a great resource for hands-on learning.

## Continuous improvement

Finally, prioritize learning and improvement, even after you've established your Azure-based processes. Review how each area is performing, and make adjustments to further optimize your approach. Explore Azure services that you hadn't previously considered to see how they can be extended to your Arc-enabled servers. Over a few cycles, your confidence in a cloud-native approach to server management will grow.
