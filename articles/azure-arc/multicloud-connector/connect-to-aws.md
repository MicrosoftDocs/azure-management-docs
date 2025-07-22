---
title: "Connect to AWS with the multicloud connector in the Azure portal"
description: "Learn how to add an AWS cloud by using the multicloud connector enabled by Azure Arc."
ms.topic: how-to
ms.date: 07/03/2025
# Customer intent: "As a cloud administrator, I want to connect my AWS account to Azure using the multicloud connector, so that I can manage my AWS resources alongside Azure services efficiently."
---

# Connect to AWS with the multicloud connector in the Azure portal

The multicloud connector enabled by Azure Arc lets you connect non-Azure public cloud resources to Azure by using the Azure portal. Currently, AWS public cloud environments are supported.

As part of connecting an AWS account to Azure, you deploy a CloudFormation template to the AWS account. This template creates all of the required resources for the connection.

## Prerequisites

To use the multicloud connector, you need the appropriate permissions in both AWS and Azure.

### AWS prerequisites

To create the connector and to use multicloud inventory, you need the following permissions in AWS:

- **AmazonS3FullAccess**
- **AWSCloudFormationFullAccess**
- **IAMFullAccess**

For Arc onboarding, there are [more prerequisites that must be met](onboard-multicloud-vms-arc.md#prerequisites).

#### AWS solution permissions

When you upload your CloudFormation template, more permissions are requested, based on the solutions that you selected:

- For **Inventory**, you can choose your permission:

  1. **Global Read**: Provides read-only access to all resources in the AWS account. When new services are introduced, the connector can scan for those resources without requiring an updated CloudFormation template.

  1. **Least Privilege Access**: Provides read access to only the resources under the selected services. If you choose to scan for more resources in the future, a new CloudFormation template must be uploaded.

- For **Arc Onboarding**, our service requires **EC2 Write** access in order to install the [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview).

### Azure prerequisites

To use the multicloud connector in an Azure subscription, you need the **Contributor** built-in role.

If this is the first time you're using the service, you need to [register these resource providers](/azure/azure-resource-manager/management/resource-providers-and-types#register-resource-provider), which requires **Contributor** access on the subscription:

- Microsoft.HybridCompute
- Microsoft.HybridConnectivity
- Microsoft.AwsConnector
- Microsoft.Kubernetes

> [!NOTE]
> The multicloud connector can work side-by-side with the [AWS connector in Defender for Cloud](/azure/defender-for-cloud/quickstart-onboard-aws). If you choose, you can use both of these connectors.

## Add your public cloud in the Azure portal

To add your AWS public cloud to Azure, use the Azure portal to enter details and generate a CloudFormation template.

1. In the Azure portal, navigate to **Azure Arc**.
1. Under **Management**, select **Multicloud connectors (preview)**.
1. In the **Connectors** pane, select **Create**.
1. On the **Basics** page:

   1. Select the subscription and resource group in which to create your connector resource.
   1. Enter a unique name for the connector and select a [supported region](overview.md#supported-regions).
   1. Provide the ID for the AWS account that you want to connect, and indicate whether it's a single account or an organization account.
   1. Select **Next**.

1. On the **Solutions** page, select which solutions you'd like to use with this connector and configure them. Select **Add** to enable **[Inventory](view-multicloud-inventory.md)**, **[Arc onboarding](onboard-multicloud-vms-arc.md)**, or both.

      :::image type="content" source="media/add-aws-connector-solutions.png" alt-text="Screenshot showing the Solutions for the AWS connector in the Azure portal.":::

   - For **Inventory**, you can modify the following options:

      1. Choose whether or not to enable **Add all supported AWS services.** By default, this option is enabled, so that all services (available now and services added in the future) are scanned.

      1. Choose the **AWS Services** for which you want to scan and import resources. By default, all available services are selected.
      1. Choose your permissions. If **Add all supported AWS services** is checked, you must have **Global read** access.
      1. Choose whether or not to enable periodic sync. By default, this option is enabled so that the connector scans your AWS account regularly. If you uncheck the box, your AWS account is scanned only once.
      1. If **Enable periodic sync** is checked, confirm or change the **Recur every** selection to specify how often your AWS account are scanned.
      1. Choose whether or not to enable **Include all supported AWS regions**. By selecting this option, all current and future AWS regions are scanned.
      1. Choose which regions to scan for resources in your AWS account. By default, all available regions are selected. If you selected **Include all supported AWS regions**, all regions must be selected.
      1. When you have finished making selections, select **Save** to return to the **Solutions** page.

   - For **Arc onboarding**:

      1. Select a **Connectivity method** to determine whether the Connected Machine agent should connect to the internet via a public endpoint or by proxy server. If you select **Proxy server**, provide a **Proxy server URL** to which the EC2 instance can connect.
      1. Choose whether or not to enable periodic sync. By default, this option is enabled so that the connector scans your AWS account regularly. If you uncheck the box, your AWS account is scanned only once.
      1. If **Enable periodic sync** is checked, confirm or change the **Recur every** selection to specify how often your AWS account are scanned.
      1. Choose whether or not to enable **Include all supported AWS regions**. By selecting this option, all current and future AWS regions are scanned.
      1. Choose which regions to scan for EC2 instances in your AWS account. By default, all available regions are selected. If you selected **Include all supported AWS regions**, all regions must be selected.
      1. Choose to filter for EC2 instances by AWS tag. If you enter a tag value here, only EC2 instances that contain that tag are onboarded to Arc. By leaving this value empty, all EC2 Instances discovered are onboarded to Arc.

1. On the **Authentication template** page, download the CloudFormation template that you'll upload to AWS. This template is created based on the information you provided in **Basics** and the solutions you selected. You can [upload the template](#upload-cloudformation-template-to-aws) right away, or wait until you finish adding your public cloud.
1. On the **Tags** page, enter any tags you'd like to use.
1. On the **Review and create** page, confirm your information, then select **Create**.

If you didn't upload your template during this process, follow the steps in the next section to do so.

## Upload CloudFormation template to AWS

After you save the CloudFormation template generated in the previous section, you need to upload it to your AWS public cloud. If you upload the template before you finish connecting your AWS cloud in the Azure portal, your AWS resources are scanned immediately. If you complete the **Add public cloud** process in the Azure portal before uploading the template, it takes a bit longer to scan your AWS resources and make them available in Azure.

### Create stack

Follow these steps to create a stack and upload your template:

1. Open the AWS CloudFormation console and select **Create stack**.
1. Select **Template is ready**, then select **Upload a template file**. Select **Choose file** and browse to select your template. Then select **Next**.
1. In **Specify stack details**, enter a stack name.

   1. If you selected the **Arc Onboarding** solution, fill out the following details in the Stack parameters:

      1. **EC2SSMIAMRoleAutoAssignment**: Specifies whether IAM roles used for SSM tasks are automatically assigned to EC2 instances. By default this option is set to *true*, and all discovered EC2 machines will have the IAM role assigned. If you set this option to *false,* you must manually assign the IAM role to the EC2 instances that you want onboarded to Arc.

      1. **EC2SSMIAMRoleAutoAssignmentSchedule**: Specifies whether the EC2 IAM Role used for SSM tasks should be autoassigned periodically. By default, this option is set to *enable,* meaning that any EC2 machines discovered in the future will have the IAM role automatically assigned. If you set this option to *disable,* you must manually assign the IAM role to any newly deployed EC2 that you want onboarded to Azure Arc.

      1. **EC2SSMIAMRoleAutoAssignmentScheduleInterval**:  Specifies the periodic interval for autoassignment of the EC2 IAM Role used for SSM tasks (for example, 15 minutes, 6 hours, or 1 day). If you set the **EC2SSMIAMRoleAutoAssignment** to *true* and **EC2SSMIAMRoleAutoAssignmentSchedule** to *enable*, you can choose how often you want to scan for new EC2 instances to be assigned the IAM role. The default interval is *1 day*.

      1. **EC2SSMIAMRolePolicyUpdateAllowed**: Specifies whether existing EC2 IAM roles used for SSM tasks are allowed to update with required permission policies if they're missing. By default, this option is set to *true*. If you choose to set to *false*, you must manually add this IAM role permission to the EC2 instance.

   1. Otherwise, leave the other options set to their default settings and select **Next**.

1. In **Configure stack options**, leave the options set to their default settings and select **Next**.
1. In **Review and create**, confirm that the information is correct, select the acknowledgment checkbox, and then select **Submit**.

### Create StackSet

If your AWS account is an organization account, you also need to create a StackSet and upload your template again. To do so:

1. Open the AWS CloudFormation console and select **StackSets**, then select **Create StackSet**.
1. Select **Template is ready**, then select **Upload a template file**. Select **Choose file** and browse to select your template. Then select **Next**.
1. In **Specify stack details**, enter `AzureArcMultiCloudStackset` as the StackSet name

   1. If you selected the **Arc Onboarding** solution, fill out the following details in the Stack parameters:

      1. **EC2SSMIAMRoleAutoAssignment**: Specifies whether IAM roles used for SSM tasks are automatically assigned to EC2 instances. By default this  option is set to *true*, and all discovered EC2 machines will have the IAM role assigned. If you set this option to *false,* you must manually assign the IAM role to the EC2 instances that you want onboarded to Arc.

      1. **EC2SSMIAMRoleAutoAssignmentSchedule**: Specifies whether the EC2 IAM Role used for SSM tasks should be autoassigned periodically. By default, this option is set to *enable,* meaning that any EC2 machines discovered in the future will have the IAM role automatically assigned. If you set this option to *disable,* you must manually assign the IAM role to any newly deployed EC2 instance that you want onboarded to Arc.

      1. **EC2SSMIAMRoleAutoAssignmentScheduleInterval**:  Specifies the periodic interval for autoassignment of the EC2 IAM Role used for SSM tasks (such as, 15 minutes, 6 hours, or 1 day). If you set the **EC2SSMIAMRoleAutoAssignment** to *true* and **EC2SSMIAMRoleAutoAssignmentSchedule** to *enable*, you can choose how often you want to scan for new EC2 instances to be assigned the IAM role. The default interval is *1 day*.

      1. **EC2SSMIAMRolePolicyUpdateAllowed**: Specifies whether existing EC2 IAM roles used for SSM tasks are allowed to update with required permission policies if they're missing. By default, this option is set to *true*. If you choose to set to *false*, you must manually add this IAM role permission to the EC2 instance.

   1. Otherwise, leave the other options set to their default settings and select **Next**.
1. In **Configure stack options**, leave the options set to their default settings and select **Next**.
1. In **Set deployment options**, enter the ID for the AWS account where the StackSet will be deployed, and select any AWS region to deploy the stack. Leave the other options set to their default settings and select **Next**.
1. In **Review**, confirm that the information is correct, select the acknowledgment checkbox, and then select **Submit**.

## Confirm deployment

After you complete the **Add public cloud** option in Azure and upload your template to AWS, your connector and selected solutions are created. On average, it takes about one hour for your AWS resources to become available in Azure. If you upload the template after creating the public cloud in Azure, it may take a bit more time before you see the AWS resources.

AWS resources are stored in a resource group using the naming convention `aws_yourAwsAccountId`, with permissions inherited from its subscription. Scans run regularly to update these resources, based on your **Enable periodic sync** selections.

## Next steps

- Query your inventory with [the multicloud connector **Inventory** solution](view-multicloud-inventory.md).
- Learn how to [use the multicloud connector **Arc onboarding** solution](onboard-multicloud-vms-arc.md).
