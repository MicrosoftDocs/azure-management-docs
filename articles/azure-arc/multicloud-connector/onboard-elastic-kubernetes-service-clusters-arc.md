---
title: Onboard Amazon EKS clusters to Azure Arc through the multicloud connector (preview)
description: Learn how to enable the EKS Arc onboarding solution to discover and onboard Amazon EKS clusters to Azure Arc-enabled Kubernetes through the multicloud connector.
ms.topic: how-to
ms.date: 05/19/2026
# Customer intent: As a cloud administrator, I want to onboard Amazon EKS clusters to Azure Arc-enabled Kubernetes through the multicloud connector, so that I can manage AWS Kubernetes clusters alongside Azure resources by using Azure management, governance, and security services.
---

# Onboard Kubernetes Clusters to Azure Arc through the Multicloud Connector (preview)

The Multicloud Connector Arc onboarding solutions for Kubernetes Clusters auto-discovers supported Kubernetes clusters in connected public cloud accounts and help onboard them to Azure Arc-enabled Kubernetes. This simplified experience lets you view and manage Kubernetes clusters alongside Azure and Arc-enabled resources, and use supported Azure management, governance, and security services for Kubernetes workloads.

Currently supported platforms include:

- Amazon Elastic Kubernetes Service (Amazon EKS)

- Google Kubernetes Engine (GKE)

> [!IMPORTANT]
> The EKS Arc onboarding and GKE Arc onboarding solutions for the Multicloud Connector are currently in preview. See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md#prerequisites), make sure you meet the requirements for each Kubernetes cluster that you want to onboard to Azure Arc.

### [AWS EKS](#tab/awseks)

- You must have the required AWS permissions to discover Amazon EKS clusters and to deploy or update the AWS resources used by the Multicloud Connector.

- Amazon EKS clusters must be in AWS regions supported by the Multicloud Connector.

- The Amazon EKS cluster must be reachable from the onboarding environment so the Azure Arc-enabled Kubernetes agents can be installed.
- The Kubernetes API server endpoint must be accessible based on the connectivity method you select.
- The cluster must meet the [general prerequisites for connecting Kubernetes clusters to Azure Arc](/azure/azure-arc/kubernetes/system-requirements).
- The cluster can't already be connected to Azure Arc. If the cluster is already Arc-enabled, onboarding through the Multicloud Connector might create duplicate Azure resource representations or fail. Use the existing Arc-enabled Kubernetes resource for clusters that are already connected.

- If you use private networking, a proxy, or Arc gateway, make sure the required URLs, proxy settings, and network paths are configured before onboarding.

### [GCP GKE](#tab/gcpgke)

- You must have the required GCP permissions to discover GKE clusters and to deploy or update the GCP resources used by the Multicloud Connector.

- GKE clusters must be in GCP regions supported by the Multicloud Connector.

- You must be able to reach the GKE cluster from the onboarding environment so you can install the Azure Arc-enabled Kubernetes agents.

- The Kubernetes API server endpoint must be accessible based on the connectivity method you select.
- The cluster must meet the [general prerequisites for connecting Kubernetes clusters to Azure Arc](/azure/azure-arc/kubernetes/system-requirements).
- The cluster can't already be connected to Azure Arc. If the cluster is already Arc-enabled, onboarding through the Multicloud Connector might create duplicate Azure resource representations or fail. Use the existing Arc-enabled Kubernetes resource for clusters that are already connected.

- If you use private networking, a proxy, or Arc gateway, make sure the required URLs, proxy settings, and network paths are configured before onboarding.

---

## Resource representation in Azure

After you connect your AWS or GCP account and enable the related **Arc onboarding** solution, the Multicloud Connector discovers eligible Kubernetes clusters and represents them in Azure.

Discovered clusters are associated with the connected AWS or GCP account and you can view them with other multicloud resources in Azure. When you onboard a cluster to Azure Arc, you create an Azure Arc-enabled Kubernetes resource so you can manage the cluster from Azure.

The Multicloud Connector places these resources in Azure regions by using the [standard region-mapping scheme](resource-representation.md#region-mapping). You can filter which AWS or GCP regions are scanned. By default, all supported AWS or GCP regions are scanned, but you can exclude regions when you [configure the solution](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal).

The resource group created for the connected AWS or GCP account follows the naming convention `<PublicCloud>_<AccountId>`. This resource group inherits permissions from its Azure subscription. You can grant additional access to user accounts in your tenant as needed.

## Connectivity method

When you create the **EKS or GKE Arc onboarding** solution, you configure how the onboarding process and Azure Arc-enabled Kubernetes agents connect to Azure.

Depending on your environment, you might need to provide:

- Kubernetes API server endpoint information.
- HTTP or HTTPS proxy settings.
- Proxy skip-range settings.
- An Arc gateway resource, if you want Arc traffic to use Azure Arc gateway.
- Network access from the EKS or GKE cluster to the required Azure Arc endpoints.

Arc gateway can simplify network configuration by reducing the number of endpoints that must be allowed in your environment to use Azure Arc. For more information, see [Simplify network configuration requirements with Azure Arc gateway](/azure/azure-arc/servers/arc-gateway).

If you use a proxy server, make sure the proxy is reachable from the cluster and that the required Azure Arc traffic is allowed.

## Periodic sync options

The periodic sync time that you select when configuring the **EKS or GKE Arc onboarding** solution determines how often your source cloud is scanned and synced to Azure.

When you enable periodic sync, the solution evaluates newly discovered clusters that meet the prerequisites for onboarding. If the solution discovers a supported cluster and you select it for onboarding, the Multicloud Connector attempts to onboard the cluster to Azure Arc-enabled Kubernetes.

If onboarding fails for an eligible cluster, the solution can retry the cluster during a later sync, provided it still meets the onboarding prerequisites and remains in scope for the configured filters.

Clusters that are already connected to Azure Arc should be managed through their existing Arc-enabled Kubernetes resource. If an existing Arc-enabled Kubernetes agent is unhealthy or disconnected, fix the underlying issue on the cluster rather than attempting to onboard it again through the Multicloud Connector.

Periodic sync also helps keep Azure resource representations aligned with your source cloud. For example, if a cluster is deleted from AWS or GCP, the corresponding discovered inventory representation in Azure can be removed during a later sync.

If you prefer, you can turn periodic sync off when configuring the solution. If you turn periodic sync off, new clusters aren't automatically discovered and evaluated for onboarding after the initial configuration.

## Filter options

Choose which clusters are eligible for onboarding by filtering based on:

- Source cloud regions.

- Source cloud tags or labels.

Select specific source cloud regions to scan for Kubernetes clusters. You can also filter by tags or labels so that only clusters with matching tags or labels are eligible for Arc onboarding. Tag or label matching is case-insensitive.

## Onboarding experience

After you enable the **EKS or GKE Arc onboarding** solution, the Multicloud Connector discovers clusters in the selected regions and shows their onboarding status.

Common statuses might include:

| Status | Meaning |
|----|----|
| Connecting| The Multicloud Connector is attempting to onboard the cluster to Azure Arc. |
| Connected | The cluster is onboarded to Azure Arc-enabled Kubernetes. |
|Agent Not Installed| The cluster is eligible or selected for onboarding, but the Azure Arc-enabled Kubernetes agents aren't installed yet. |
| Offline|The cluster's is onboarded to Azure Arc-enabled Kubernetes, but the cluster is offline. |
| Expired|The cluster's is connection to Azure Arc-enabled Kubernetes has expired. |

The exact status names in the Azure portal might vary.

## Limitations

The **EKS or GKE Arc onboarding** solutions have the following limitations:

- Only Amazon EKS clusters in AWS, and GKE clusters in GCP are supported.

- Existing EKS or GKE clusters that are already connected to Azure Arc might require separate handling to avoid duplicate Azure resources.

- Brownfield onboarding for clusters with an existing Azure Arc-enabled Kubernetes agent might not be supported in the initial preview.
- Some Azure services for Arc-enabled Kubernetes might have their own regional, networking, or feature availability requirements.

## Next steps

- Learn more about [Azure Arc-enabled Kubernetes](/azure/azure-arc/kubernetes/overview).
- Learn about the [Multicloud Connector Inventory solution](view-multicloud-inventory.md).
- Learn how to [connect AWS with the Multicloud Connector](add-public-cloud.md).
- Learn how to [simplify network configuration requirements with Azure Arc gateway](/azure/azure-arc/servers/arc-gateway).
