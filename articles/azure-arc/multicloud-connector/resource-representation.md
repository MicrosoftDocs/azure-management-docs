---
title: Multicloud connector resource representation in Azure
description: Understand how AWS and GCP resources are represented in Azure after they're added through the multicloud connector enabled by Azure Arc.
ms.topic: how-to
ms.date: 11/07/2025
ms.custom: references_regions
# Customer intent: As a cloud administrator, I want to understand how my connected AWS and GCP resources are represented in Azure, so that I can efficiently manage and govern all my cloud resources from a centralized platform.
---

# Multicloud connector resource representation in Azure

The multicloud connector enabled by Azure Arc lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance. Currently, the multicloud connector provides support for connecting resources from these public clouds:

- Amazon Web Services (AWS)
- Google Cloud Platform (GCP) (preview)

This article describes how AWS resources from a connected public cloud are represented in your Azure environment.

## Resource group name

After you [connect your public cloud to Azure](add-public-cloud.md), the multicloud connector creates a new resource group with the following naming convention:

`<PublicCloud>_<AccountId>`

> [!NOTE]
> Tags placed on the connector are extended and added to the resource group when it's created. If you need to adhere to policies requiring tags when creating new resource groups, be sure to add the required tags on the connector. Otherwise, the resource group creation process will fail due to the tags being missing.

For every AWS resource discovered through the **[Inventory](view-multicloud-inventory.md)** solution, an Azure representation is created in the `<PublicCloud>_<AccountId>` resource group. Each resource has the [namespace value associated with its AWS service](view-multicloud-inventory.md#supported-services).

AWS EC2 instances and GCP VMs that are connected to Azure Arc through the **[Arc onboarding](onboard-multicloud-vms-arc.md)** solution are represented as Arc-enabled server resources under `Microsoft.HybridCompute/machines` in the `<PublicCloud>_<AccountId>` resource group. If you previously onboarded an AWS EC2 instance or GCP VM to Azure Arc, you won't see that machine in this resource group, because it already has a representation in Azure.

## Region mapping

Resources that are discovered and projected in Azure are placed in Azure regions, using the following mapping scheme:

### [AWS](#tab/aws)

|AWS region |Mapped Azure region |
|--|--|
|us-east-1 | EastUS |
|us-east-2 | EastUS |
|us-west-1 | EastUS |
|us-west-2 | EastUS |
|ca-central-1 | EastUS |
|ap-southeast-1 | Southeast Asia |
|ap-northeast-1 | Southeast Asia |
|ap-northeast-3 | Southeast Asia |
|ap-south | Southeast Asia |
|ap-southeast-2 | Australia East |
|eu-west-1 | West Europe |
|eu-central-1 | West Europe |
|eu-north-1 | West Europe |
|eu-west-2 | UK South |
|sa-east-1 | Brazil South |

### [GCP](#tab/gcp)

|GCP region |Mapped Azure region |
|--|--|
|us-east1 | EastUS |
|us-east4 | EastUS |
|northamerica-northeast1 | EastUS |
|us-central1 | EastUS |
|us-west1 | EastUS |
|us-west2 | EastUS |
|europe-west1 | West Europe |
|europe-west2 | UK South |
|europe-west3 | West Europe |
|europe-north2 | West Europe |
|southamerica-east1 | Brazil South |
|australia-southeast1| Australia East|

---
## Removing resources

If you remove the connector, or disable a solution, periodic syncs will stop for that solution, and resources will no longer be updated in Azure. However, the resources will remain in your Azure account unless you delete them. To avoid confusion, we recommend removing these resource representations from Azure when you remove a  public cloud.

To remove all of the resource representations from Azure, navigate to the `<PublicCloud>_<AccountId>` resource group, then delete it.

If you delete the connector:

- In AWS, you should delete the CloudFormation Template. If you delete a solution, you'll also need to update your template to remove the required access for the deleted solution. You can find the updated template for the connector in the Azure portal under **Settings > Authentication template**.
- In GCP, you should delete the OSConfig policy for the Arc Onboarding solution. If you delete a solution, you'll also need to update your Terraform template to remove the required access for the deleted solution. You can find the updated template for the connector in the Azure portal under **Settings > Terraform template**.

To move the connector to a different subscription or resource group, you must delete it and recreate it in the desired location. Moving the connector resource and the resources directly in Azure isn't supported.

