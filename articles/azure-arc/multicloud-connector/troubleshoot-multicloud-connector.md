---
title: Troubleshoot multicloud connector enabled by Azure Arc
description: Troubleshoot the multicloud connector enabled by Azure Arc
ms.topic: how-to
ms.date: 11/07/2025
# Customer intent: As a cloud administrator, I want to troubleshoot issues with the multicloud connector enabled by Azure Arc, so that I can ensure seamless integration and management of my AWS and GCP resources within the Azure environment.
---

# Troubleshoot multicloud connector enabled by Azure Arc

This document provides guidance on troubleshooting issues for your Multicloud connector and solutions.

## Issues with AWS authentication

If you're having issues authenticating to AWS, and the status of your solutions are **Failed**, try the following steps:

1. Browse to the **CloudFormation** Dashboard in AWS. Delete your existing CloudFormation Stack (and StackSet for Organization Account).
1. In the Azure portal, browse to your Connector for this account. Check to make sure the **AWS account ID** listed in Azure matches the AWS account number.
1. Browse to the **Authentication template** tab under **Settings** and download the CloudFormation template.
1. In AWS, create a new **Stack** (and **StackSet** for Organization Account) with the newly downloaded CloudFormation Template.
1. After the AWS CloudFormation Stack is created, navigate to your connector in the Azure portal and select **Test Permissions**. The status should now show as **Success**.

## Issues with GCP authentication

If you’re having issues running the Terraform commands, these steps may help resolve common errors.

**`Error creating WorkloadIdentityPool: googleapi: Error 409: Requested entity already exists`** indicates that the Workload Identity Pool already exists in your project. Terraform attempts to recreate it, resulting in a 409 error from Google. To resolve this, import the existing pool and providers into the Terraform state and repeat the necessary steps, using the following commands. Replace `PROJECT_NUMBER` and `PROVIDER_ID` with real values. To find `PROVIDER_ID`, go to **GCP Portal > IAM & Admin > Workload Identity Federation**. Locate the `AzureArcMulticloudConnector` pool; the `PROVIDER_ID` will be listed below, such as `inventory-xxxxxxxxxx` or `server-xxxxxxxxxx`. 

```terraform
terraform import google_iam_workload_identity_pool.main_pool projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/azurearcmulticloudconnector
terraform import google_iam_workload_identity_pool_provider.asset_management_provider projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/azurearcmulticloudconnector/providers/PROVIDER_ID
terraform import google_iam_workload_identity_pool_provider.arc_for_server_provider projects/PROJECT_NUMBER/locations/global/workloadIdentityPools/azurearcmulticloudconnector/providers/PROVIDER_ID
```

**`Error returning invalid authentication`** signifies that your gcloud authentication token has expired or is invalid. To fix this, reauthenticate your user account by running `gcloud auth login` or `gcloud auth login --update-adc`.

## Issues with Connector creation

If you're trying to create your connector but hit issues, try the following:

- Ensure you have the necessary [prerequisites](add-public-cloud.md#prerequisites).
- The connector creates [resource groups](resource-representation.md#resource-group-name) to store your AWS resources for the Inventory solution. If you have a policy that blocks resource group creation, do one of the following actions:
  - Remove the policy until the connector creation succeeds.
  - Manually create the resource group prior to creating the connector.

## Issues with scanning for resources

If you believe a source cloud resource is missing in Azure, check the following steps:

1. Check to make sure your authentication is correct by browsing to your connector in the Azure portal and selecting **Test Permissions**.
1. Ensure that the [resource type](view-multicloud-inventory.md#supported-services) is supported.
1. Ensure that the [AWS or GCP region](overview.md#supported-regions) where the resource is located is supported.
1. Ensure that you selected the AWS or GCP service and the correct region in your Inventory solution by browsing to your connector in the portal and selecting **Solutions** under the **Settings** tab.
1. Select **Scan now** to rescan your source cloud, then check to see if the resource appears in the **Resources** tab of your connector.

## Issues with Arc onboarding

If you selected the Arc onboarding solution, but your EC2 instance or GCP VM isn't showing the **Connected** state, check the following steps:

1. Check to make sure your authentication is correct by browsing to your connector in the Azure portal and selecting **Test Permissions**.
1. Ensure you have the necessary [prerequisites](onboard-multicloud-vms-arc.md#prerequisites).
1. In AWS, browse to your EC2 machine and ensure that it has the **ArcForServerSSMRole** IAM role assigned.
1. Connect to your EC2 instance and ensure the [SSM agent is running](https://docs.aws.amazon.com/systems-manager/latest/userguide/ssm-agent-status-and-restart.html).
1. Ensure your EC2 instance is in a supported [AWS region](overview.md#supported-regions).
1. Check to ensure your EC2 instance meets your filtering requirements by browsing to **Solutions** under the **Settings** tab. Select **Arc onboarding** and see if there are any EC2 Filter Tags or AWS regions. Ensure your EC2 instance meets these filter requirements.
1. Select **Scan now** to rescan your AWS account. Browse to the **Machines** view for Azure Arc in the Azure portal, then select your EC2 instance to check the connectivity status.
