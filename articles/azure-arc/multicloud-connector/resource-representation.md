---
title: Multicloud connector resource representation in Azure
description: Understand how AWS resources are represented in Azure after they're added through the multicloud connector enabled by Azure Arc.
ms.topic: how-to
ms.date: 06/11/2024
# Customer intent: As a cloud administrator, I want to understand how my connected AWS resources are represented in Azure, so that I can efficiently manage and govern all my cloud resources from a centralized platform.
---

# Multicloud connector resource representation in Azure

The multicloud connector enabled by Azure Arc lets you connect non-Azure public cloud resources to Azure, providing a centralized source for management and governance. Currently, AWS public cloud environments are supported.

This article describes how AWS resources from a connected public cloud are represented in your Azure environment.

## Resource group name

After you [connect your AWS public cloud to Azure](connect-to-aws.md), the multicloud connector creates a new resource group with the following naming convention:

`aws_yourAwsAccountId`

> [!NOTE]
> Tags placed on the connector will be extended and added to the resource group when it is created. Be sure to add any tags on the connector to adhere to any policies requiring tags when creating a resource group, otherwise the resource group creation process will fail due to the tags being missing.

For every AWS resource discovered through the **[Inventory](view-multicloud-inventory.md)** solution, an Azure representation is created in the `aws_yourAwsAccountId` resource group. Each resource has the [`AwsConnector` namespace value associated with its AWS service](view-multicloud-inventory.md#supported-aws-services).

EC2 instances connected to Azure Arc through the **[Arc onboarding](onboard-multicloud-vms-arc.md)** solution are also represented as Arc-enabled server resources under `Microsoft.HybridCompute/machines` in the `aws_yourAwsAccountId` resource group. If you previously onboarded an EC2 machine to Azure Arc, you won't see that machine in this resource group, because it already has a representation in Azure.

## Region mapping

Resources that are discovered in AWS and projected in Azure are placed in Azure regions, using the following mapping scheme:

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
|ap-southeast-2 | AU East |
|eu-west-1 | West Europe |
|eu-central-1 | West Europe |
|eu-north-1 | West Europe |
|eu-west-2 | UK South |
|sa-east-1 | Brazil South |

## Removing resources

If you remove the connector, or disable a solution, periodic syncs will stop for that solution, and resources will no longer be updated in Azure. However, the resources will remain in your Azure account unless you delete them. To avoid confusion, we recommend removing these AWS resource representations from Azure when you remove an AWS public cloud.

To remove all of the AWS resource representations from Azure, navigate to the `aws_yourAwsAccountId` resource group, then delete it.

If you delete the connector, you should delete the CloudFormation Template on AWS. If you delete a solution, you'll also need to update your CloudFormation Template to remove the required access for the deleted solution. You can find the updated template for the connector in the Azure portal under **Settings > Authentication template**.

Moving the connector resource and the AWS resources represented in Azure is not supported. If you want to move the connector to a different subscription or resource group, please delete it and recreate it in the desired location. 

