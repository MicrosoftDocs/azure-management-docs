---
title: Troubleshoot multicloud connector enabled by Azure Arc
description: Troubleshoot the multicloud connector enabled by Azure Arc
ms.topic: how-to
ms.date: 11/18/2024
---

# Troubleshoot the multicloud connector enabled by Azure Arc

This document provides guidance on troubleshooting issues for your Multicloud connector and solutions.

## Issues with AWS Authentication

If you're having issues authenticating to AWS, and the status of your solutions are **Failed**, try the following steps:

1. Sign in to the AWS account you're trying to connect to. Browse to the **Identity and Access Management (IAM)** Dashboard. Select **Identity providers**. Check to see if Provider `sts.windows.net/975f013f-7f24-47e8-a7d3-abc4752bf346/` exists. If it does, delete this Provider.
2. Browse to the **CloudFormation** Dashboard in AWS. Delete your existing CloudFormation Stack (and StackSet for Organization Account).
3. In the Azure portal, browse to your Connector for this account. Check to make sure the **AWS account ID** listed in Azure matches the AWS account number.
4. Browse to the **Authentication template** tab under **Settings** and download the CloudFormation template.
5. In AWS, create a new **Stack** (and **StackSet** for Organization Account) with the newly downloaded CloudFormation Template.
6. After the AWS CloudFormation Stack is created, navigate to your connector in the Azure portal and select **Test Permissions**. The status should now show as **Success**.

## Issues with Connector creation

If you're trying to create your connector but hit issues, try the following steps:

- Ensure you have the necessary [prerequisites](connect-to-aws.md#prerequisites).
- The connector creates [resource groups](resource-representation.md#resource-group-name) to store your AWS resources for the Inventory solution. If you have a policy that blocks resource group creation, do one of the following actions:
  - Remove the policy until the connector creation succeeds.
  - Manually create the resource group prior to creating the connector.

## Issues with scanning for AWS resources

If you believe an AWS resource is missing in Azure, check the following steps:

1. Check to make sure your authentication is correct by browsing to your connector in the Azure portal and selecting **Test Permissions**.
2. Ensure that the [resource type](view-multicloud-inventory.md#supported-aws-services) is supported.
3. Ensure that the [AWS region](overview.md#supported-regions) where the resource is located is supported.
4. Ensure that you selected the AWS service and AWS region in your Inventory solution by browsing to your connector in the portal and selecting **Solutions** under the **Settings** tab.
5. Select **Scan now** to rescan your AWS account. Check to see if the resource appears in the **Resources** tab of your connector.

## Issues with Arc onboarding

If you selected the Arc onboarding solution, but your EC2 instance isn't showing the **Connected** state, check the following steps:

1. Check to make sure your authentication is correct by browsing to your connector in the Azure portal and selecting **Test Permissions**.
2. Ensure you have the necessary [prerequisites](onboard-multicloud-vms-arc.md#prerequisites).
3. In AWS, browse to your EC2 machine and ensure that it has the **ArcForServerSSMRole** IAM role assigned.
4. Connect to your EC2 instance and ensure the [SSM agent is running](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-status-and-restart.html).
5. Ensure your EC2 instance is in a supported [AWS region](overview.md#supported-regions).
6. Check to ensure your EC2 instance meets your filtering requirements by browsing to **Solutions** under the **Settings** tab. Select **Arc onboarding** and see if there are any EC2 Filter Tags or AWS regions. Ensure your EC2 instance meets these filter requirements.
7. Select **Scan now** to rescan your AWS account. Browse to the **Machines** view for Azure Arc in the Azure portal, then select your EC2 instance to check the connectivity status.
