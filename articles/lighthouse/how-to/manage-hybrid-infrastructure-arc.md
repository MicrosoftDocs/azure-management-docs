---
title: Manage hybrid infrastructure at scale with Azure Arc
description: Azure Lighthouse helps you use Azure Arc to effectively manage customers' machines and Kubernetes clusters outside of Azure.
ms.date: 01/09/2025
ms.topic: how-to
# Customer intent: As a service provider managing customer environments, I want to leverage Azure Arc and Azure Lighthouse to oversee hybrid infrastructure, so that I can simplify operations and maintain compliance across both cloud and on-premises resources effectively.
---

# Manage hybrid infrastructure at scale with Azure Arc

[Azure Lighthouse](../overview.md) can help service providers use Azure Arc to manage customers' hybrid environments, with visibility across all managed Microsoft Entra tenants.

[Azure Arc](../../azure-arc/overview.md) helps simplify complex and distributed environments across on-premises, edge and multicloud, enabling deployment of Azure services anywhere and extending Azure management to any infrastructure.

With [Azure Arc–enabled servers](../../azure-arc/servers/overview.md), customers can manage Windows and Linux machines hosted outside of Azure on their corporate network, in the same way they manage native Azure virtual machines. Through Azure Lighthouse, service providers can then manage these connected non-Azure machines along with their customers' Azure resources.

[Azure Arc–enabled Kubernetes](../../azure-arc/kubernetes/overview.md) lets customers attach and configure Kubernetes clusters outside of Azure. Through Azure Lighthouse, service providers can connect Kubernetes clusters and manage them along with their customer's Azure Kubernetes Service (AKS) clusters and other Azure resources.

> [!TIP]
> Though we refer to service providers and customers in this topic, this guidance also applies to [enterprises using Azure Lighthouse to manage multiple tenants](../concepts/enterprise.md).

## Manage hybrid servers at scale with Azure Arc–enabled servers

As a service provider, you can connect and disconnect on-premises Windows Server or Linux machines outside Azure to your customer's subscription. When you [generate a script to connect a server](/azure/azure-arc/servers/learn/quick-enable-hybrid-vm), use the `--user-tenant-id` parameter to specify your managing tenant, with the `--tenant-id` parameter indicating the customer's tenant.  

When viewing resources for a delegated subscription in the Azure portal, you'll see these connected machines labeled with **Azure Arc**. You can manage these connected machines using Azure constructs, such as Azure Policy and tagging, just as you would manage the customer's Azure resources. You can also work across customer tenants to manage all connected machines together.

For example, you can [ensure the same set of policies are applied across customers' hybrid machines](../../azure-arc/servers/learn/tutorial-assign-policy-portal.md). You can [use Microsoft Defender for Cloud to monitor compliance](/azure/defender-for-cloud/quickstart-onboard-machines) across all of your customers' hybrid environments, or [use Azure Monitor to collect data directly](../../azure-arc/servers/learn/tutorial-enable-vm-insights.md) into a Log Analytics workspace. [Virtual machine extensions](../../azure-arc/servers/manage-vm-extensions.md) can be deployed to non-Azure Windows and Linux VMs, simplifying management of your customers' hybrid machines.

## Manage hybrid Kubernetes clusters at scale with Azure Arc-enabled Kubernetes

You can manage Kubernetes clusters that are [connected to a customer's subscription with Azure Arc](../../azure-arc/kubernetes/quickstart-connect-cluster.md), just as if they were running in Azure.

If your customer uses a service principal account to onboard Kubernetes clusters to Azure Arc, you can access this account so that you can [onboard and manage clusters](../../azure-arc/kubernetes/quickstart-connect-cluster.md). To do so, a user in the managing tenant must have the [Kubernetes Cluster - Azure Arc Onboarding built-in role](/azure/role-based-access-control/built-in-roles#kubernetes-cluster---azure-arc-onboarding) when the subscription containing the service principal account was [onboarded to Azure Lighthouse](onboard-customer.md).

You can deploy [configurations and Helm charts](../../azure-arc/kubernetes/tutorial-use-gitops-flux2.md) using [GitOps for connected clusters](../../azure-arc/kubernetes/conceptual-gitops-flux2.md).

You can also [monitor connected clusters](/azure/azure-monitor/containers/container-insights-enable-arc-enabled-clusters) with Azure Monitor, use tagging to organize clusters, and [use Azure Policy for Kubernetes](/azure/governance/policy/concepts/policy-for-kubernetes?toc=%2Fazure%2Fazure-arc%2Fkubernetes%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fkubernetes%2Fbreadcrumb%2Ftoc.json) to manage and report on compliance state.

## Next steps

- Explore the [Azure Arc Jumpstart](https://azurearcjumpstart.com/).
- Learn about [supported cloud operations for Azure Arc-enabled servers](../../azure-arc/servers/overview.md#supported-cloud-operations).
- Learn about [accessing connected Kubernetes clusters through the Azure portal](../../azure-arc/kubernetes/kubernetes-resource-view.md).
