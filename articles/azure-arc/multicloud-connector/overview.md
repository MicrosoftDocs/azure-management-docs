---
title: "What is Multicloud connector enabled by Azure Arc?"
description: "The multicloud connector lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance."
ms.topic: overview
ms.date: 04/13/2026
ms.custom: references_regions
# Customer intent: As a cloud administrator, I want to connect AWS resources to Azure using a centralized management tool, so that I can efficiently manage and govern my multicloud environment.
---

# What is Multicloud connector enabled by Azure Arc?

Multicloud connector enabled by Azure Arc lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance. Currently, the multicloud connector provides support for connecting resources from these public clouds:

- Amazon Web Services (AWS)
- Google Cloud Platform (GCP) (preview)

The multicloud connector supports these solutions:

- **Inventory:** Provides an up-to-date view of resources from connected public clouds in Azure, giving you a single place to view Azure, AWS, and GCP resources. You can query these resources through Azure Resource Graph, including metadata imported from the source cloud. For example, you can query Azure, AWS, and GCP resources by tag from one place. The Inventory solution scans the source cloud periodically to keep the Azure representation complete and current. You can also apply Azure tags and Azure policies to these represented resources.

- **Arc onboarding for Servers:** Auto-discovers Amazon EC2 instances running in AWS and virtual machines running in GCP, then installs the [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview) so those machines are onboarded to Azure Arc. This simplified experience lets you use Azure management services, such as Azure Monitor, across Azure, AWS, and GCP server resources from a centralized location.

- **Arc onboarding for Amazon EKS clusters (preview):** Auto-discovers Amazon Elastic Kubernetes Service (Amazon EKS) clusters in your AWS environment then installs the Azure Arc-enabled Kubernetes Agent so those machines are onboarded to Azure Arc. This lets you view and manage AWS EKS clusters alongside other Azure and Arc-enabled resources, and use supported Azure management, governance, and security services for Kubernetes workloads.

- **Storage - Data management:** Reads data from Amazon Simple Storage Service (Amazon S3) in your AWS environment. This solution is used to set up the [Azure Storage Mover data connection for cloud-to-cloud migration](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json)

For more information about how the multicloud connector works, including prerequisites, see [Add a public cloud with the multicloud connector in the Azure portal](add-public-cloud.md).

The multicloud connector can work side-by-side with Defender for Cloud support for [AWS accounts](/azure/defender-for-cloud/quickstart-onboard-aws) and [GCP projects](/azure/defender-for-cloud/quickstart-onboard-gcp). If you choose, you can use these connectors together.

## Supported regions

In Azure, the following regions are supported for the multicloud connector:

- East US, East US 2, West US 2, West US 3, West Central US, South Central US, Canada Central, West Europe, North Europe, Sweden Central, UK South, Southeast Asia, AU East

The multicloud connector isn't available in national clouds (Azure Government, Microsoft Azure operated by 21Vianet). The multicloud connector doesn't store customer data outside the region the customer deploys the service instance in.

In AWS, we scan for resources in the following regions:

- us-east-1, us-east-2, us-west-1, us-west-2, ca-central-1, ap-southeast-1, ap-southeast-2, ap-northeast-1, ap-northeast-3, eu-west-1, eu-west-2, eu-central-1, eu-north-1, sa-east-1, ap-south1

In GCP, we scan for resources in the following regions:

- us-east1, us-east4, us-central1, us-west1, us-west2, europe-west1, europe-west2, europe-west3, europe-north2, northamerica-northeast1, southamerica-east1, australia-southeast1

Scanned resources are automatically [mapped to corresponding Azure regions](resource-representation.md#region-mapping).

## Pricing

The multicloud connector is free to use, but it integrates with other Azure services that have their own pricing models. Any Azure service that is used with the Multicloud Connector, such as Azure Monitor, will be charged as per the pricing for that service. For more information, see the [Azure pricing page](https://azure.microsoft.com/pricing/).

After you connect your AWS or GCP cloud, the multicloud connector queries the relevant resource APIs several times a day. These read-only API calls incur no charges in AWS or GCP. However, for AWS, these calls *are* registered in CloudTrail if you've enabled a trail for read events.

Additionally, for GCP, the **Arc onboarding** solution requires Google Cloud [VM Manager](https://cloud.google.com/compute/docs/vm-manager) to be enabled. VM Manager [incurs costs](https://cloud.google.com/compute/vm-manager/pricing) based on usage.

## Next steps

- Learn how to [connect a public cloud in the Azure portal](add-public-cloud.md).
- Learn how to [use the multicloud connector ](view-multicloud-inventory.md)**[Inventory](view-multicloud-inventory.md)**[ solution](view-multicloud-inventory.md).
- Learn how to [use the multicloud connector ](onboard-multicloud-vms-arc.md)**[Arc onboarding](onboard-multicloud-vms-arc.md)**[ solution](onboard-multicloud-vms-arc.md).
- Learn how to [use the multicloud connector ](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json)**[Storage - Data management](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json)**[ solution](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json).
