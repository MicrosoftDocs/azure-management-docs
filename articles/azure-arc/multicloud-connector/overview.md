---
title: "What is Multicloud connector enabled by Azure Arc?"
description: "The multicloud connector lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance."
ms.topic: overview
ms.date: 11/07/2025
ms.custom: references_regions
# Customer intent: As a cloud administrator, I want to connect AWS resources to Azure using a centralized management tool, so that I can efficiently manage and govern my multicloud environment.
---

# What is Multicloud connector enabled by Azure Arc?

Multicloud connector enabled by Azure Arc lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance. Currently, the multicloud connector provides support for connecting resources from these public clouds:

- Amazon Web Services (AWS)
- Google Cloud Platform (GCP) (preview)

The multicloud connector supports these solutions:

- **Inventory**: Allows you to see an up-to-date view of your resources from other public clouds in Azure, providing you with a single place to see all of your cloud resources. You can query all your cloud resources through Azure Resource Graph. When assets are represented in Azure, metadata from the source cloud is also included. For instance, if you need to query all of your Azure, AWS, and GCP resources with a certain tag, you can do so. The **Inventory** solution scans your source cloud on a periodic basis to ensure a complete, correct view is represented in Azure. You can also apply Azure tags or Azure policies on these resources.
- **Arc onboarding**: Auto-discovers EC2 instances running in your AWS environment, or VMs running in your GCP environment, and installs the [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview) on the VMs so that they're onboarded to Azure Arc. This simplified experience lets you use Azure management services such as Azure Monitor on these VMs, providing a centralized way to manage Azure, AWS, and GCP resources together.
- **Storage - Data management**: Reads data from Amazon Simple Storage Service (Amazon S3) in your AWS environment cloud. This solution is used to set up the [Azure Storage Mover data connection for cloud-to-cloud migration](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json).

For more information about how the multicloud connector works, including prerequisites, see [Add a public cloud with the multicloud connector in the Azure portal](add-public-cloud.md).

The multicloud connector can work side-by-side with Defender for Cloud support for [AWS accounts](/azure/defender-for-cloud/quickstart-onboard-aws) and [GCP projects](/azure/defender-for-cloud/quickstart-onboard-gcp). If you choose, you can use these connectors together.

## Supported regions

In Azure, the following regions are supported for the multicloud connector:

- East US, East US 2, West US 2, West US 3, West Central US, Canada Central, West Europe, North Europe, Sweden Central, UK South, Southeast Asia, AU East

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
- Learn how to [use the multicloud connector **Inventory** solution](view-multicloud-inventory.md).
- Learn how to [use the multicloud connector **Arc onboarding** solution](onboard-multicloud-vms-arc.md).
- Learn how to [use the multicloud connector **Storage - Data management** solution](/azure/storage-mover/cloud-to-cloud-migration?toc=/azure/azure-arc/multicloud-connector/toc.json).
