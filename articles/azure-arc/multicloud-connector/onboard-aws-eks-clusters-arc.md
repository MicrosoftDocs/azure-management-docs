---
title: Onboard Amazon EKS clusters to Azure Arc through the multicloud connector (preview)
description: Learn how to enable the EKS Arc onboarding solution to discover and onboard Amazon EKS clusters to Azure Arc-enabled Kubernetes through the multicloud connector.
ms.topic: how-to
ms.date: 05/19/2026
# Customer intent: As a cloud administrator, I want to onboard Amazon EKS clusters to Azure Arc-enabled Kubernetes through the multicloud connector, so that I can manage AWS Kubernetes clusters alongside Azure resources by using Azure management, governance, and security services.
---

# Onboard Amazon EKS clusters to Azure Arc through the multicloud connector (preview)

The **EKS Arc onboarding** solution of the multicloud connector autodiscoversAmazon Elastic Kubernetes Service (Amazon EKS) clusters in a [connected AWS account](add-public-cloud.md), then helps onboard supported clusters to [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/overview). This simplified experience lets you view and manage AWS EKS clusters alongside Azure and Arc-enabled resources, and use supported Azure management, governance, and security services for Kubernetes workloads.

Currently, EKS Arc onboarding through the multicloud connector is supported for:

- Amazon Web Services (AWS)

You can enable the **EKS Arc onboarding** solution when you [connect your AWS account to Azure](add-public-cloud.md) through the multicloud connector.

> [!IMPORTANT]
> The EKS Arc onboarding solution of the multicloud connector is currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md#prerequisites), make sure you meet the requirements for each Amazon EKS cluster that you want to onboard to Azure Arc.

AWS prerequisites include the following:

- You must have the required AWS permissions to discover Amazon EKS clusters and to deploy or update the AWS resources used by the multicloud connector.
- Amazon EKS clusters must be in AWS regions supported by the multicloud connector.
- The Amazon EKS cluster must be reachable from the onboarding environment so the Azure Arc-enabled Kubernetes agents can be installed.
- The Kubernetes API server endpoint must be accessible based on the connectivity method you select.
- The cluster must meet the [general prerequisites for connecting Kubernetes clusters to Azure Arc](/azure/azure-arc/kubernetes/system-requirements).
- The cluster can't already be connected to Azure Arc. If the cluster is already Arc-enabled, onboarding through the multicloud connector might create duplicate Azure resource representations or fail. Use the existing Arc-enabled Kubernetes resource for clusters that are already connected.
- If you use private networking, a proxy, or Arc gateway, make sure the required URLs, proxy settings, and network paths are configured before onboarding.

## Resource representation in Azure

After you connect your AWS account and enable the **EKS Arc onboarding** solution, the multicloud connector discovers eligible Amazon EKS clusters and represents them in Azure.

Discovered clusters are associated with the connected AWS account and can be viewed with other multicloud resources in Azure. When a cluster is onboarded to Azure Arc, an Azure Arc-enabled Kubernetes resource is created so you can manage the cluster from Azure.

These resources are placed in Azure regions by using the [standard region-mapping scheme](resource-representation.md#region-mapping) of the multicloud connector. You can filter which AWS regions are scanned. By default, all supported AWS regions are scanned, but you can exclude regions when you [configure the solution](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal).

The resource group created for the connected AWS account follows the naming convention `<PublicCloud>_<AccountId>`. This resource group inherits permissions from its Azure subscription. You can grant additional access to user accounts in your tenant as needed.

## Connectivity method

When creating the **EKS Arc onboarding** solution, you configure how the onboarding process and Azure Arc-enabled Kubernetes agents connect to Azure.

Depending on your environment, you might need to provide:

- Kubernetes API server endpoint information.
- HTTP or HTTPS proxy settings.
- Proxy skip-range settings.
- An Arc gateway resource, if you want Arc traffic to use Azure Arc gateway.
- Network access from the EKS cluster to the required Azure Arc endpoints.

Arc gateway can simplify network configuration by reducing the number of endpoints that must be allowed in your environment to use Azure Arc. For more information, see [Simplify network configuration requirements with Azure Arc gateway](/azure/azure-arc/servers/arc-gateway).

If you use a proxy server, make sure the proxy is reachable from the cluster and that the required Azure Arc traffic is allowed.

## Periodic sync options

The periodic sync time that you select when configuring the **EKS Arc onboarding** solution determines how often your AWS account is scanned and synced to Azure.

When periodic sync is enabled, newly discovered Amazon EKS clusters that meet the prerequisites are evaluated for onboarding. If a supported cluster is discovered and selected for onboarding, the multicloud connector attempts to onboard it to Azure Arc-enabled Kubernetes.

If EKS onboarding fails for an eligible cluster, the cluster can be retried during a later sync, provided it still meets the onboarding prerequisites and remains in scope for the configured filters.

Clusters that are already connected to Azure Arc should be managed through their existing Arc-enabled Kubernetes resource. If an existing Arc-enabled Kubernetes agent is unhealthy or disconnected, fix the underlying issue on the cluster rather than attempting to onboard it again through the multicloud connector.

Periodic sync also helps keep Azure resource representations aligned with AWS. For example, if an EKS cluster is deleted from AWS, the corresponding discovered inventory representation in Azure can be removed during a later sync.

If you prefer, you can turn periodic sync off when configuring the solution. If periodic sync is off, new Amazon EKS clusters aren't automatically discovered and evaluated for onboarding after the initial configuration.

## Filter options

You can choose which Amazon EKS clusters are eligible for onboarding by filtering based on:

- AWS regions.
- AWS tags.

You can select specific AWS regions to scan for EKS clusters. You can also filter by AWS tags so that only clusters with matching tags are eligible for Arc onboarding. Tag matching is case-insensitive.

## Onboarding experience

After you enable the **EKS Arc onboarding** solution, the multicloud connector discovers EKS clusters in the selected AWS regions and shows their onboarding status.

Common statuses might include:

| Status | Meaning |
|----|----|
| Discovered | The EKS cluster was found by the multicloud connector. |
| Agent not installed | The cluster is eligible or selected for onboarding, but the Azure Arc-enabled Kubernetes agents aren't installed yet. |
| Onboarding in progress | The multicloud connector is attempting to onboard the cluster to Azure Arc. |
| Connected | The cluster is onboarded to Azure Arc-enabled Kubernetes. |
| Failed | The onboarding attempt didn't complete successfully. Review the error details, fix the issue, and retry onboarding. |

The exact status names in the Azure portal might vary.

## Limitations

The **EKS Arc onboarding** solution has the following limitations:

- Only Amazon EKS clusters in AWS are supported.
- GCP Kubernetes clusters aren't supported by this solution.
- Existing Amazon EKS clusters that are already connected to Azure Arc might require separate handling to avoid duplicate Azure resources.
- Brownfield onboarding for clusters with an existing Azure Arc-enabled Kubernetes agent might not be supported in the initial preview.
- Some Azure services for Arc-enabled Kubernetes might have their own regional, networking, or feature availability requirements.

## Next steps

- Learn more about [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/overview).
- Learn about the [multicloud connector **Inventory** solution](view-multicloud-inventory.md).
- Learn how to [connect AWS with the multicloud connector](add-public-cloud.md).
- Learn how to [simplify network configuration requirements with Azure Arc gateway](/azure/azure-arc/servers/arc-gateway).
