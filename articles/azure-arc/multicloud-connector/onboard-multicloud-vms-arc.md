---
title: Onboard VMs to Azure Arc through the multicloud connector
description: Learn how to enable the Arc onboarding solution with the multicloud connector enabled by Azure Arc.
ms.topic: how-to
ms.date: 01/08/2025
---

# Onboard VMs to Azure Arc through the multicloud connector

The **Arc onboarding** solution of the multicloud connector autodiscovers VMs in a [connected public cloud](connect-to-aws.md), then installs the [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview) to onboard the VMs to Azure Arc. Currently, EC2 instances in AWS public cloud environments are supported.

This simplified experience lets you use Azure management services, such as Azure Monitor, providing a centralized way to manage Azure and AWS VMs together.

You can enable the **Arc onboarding** solution when you [connect your public cloud to Azure](connect-to-aws.md).

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](connect-to-aws.md#prerequisites), be sure to meet the requirements for the **Arc onboarding** solution. This includes requirements for each EC2 instance to be onboarded to Azure Arc.

- You must have **AmazonEC2FullAccess** permissions in your public cloud.
- EC2 instances must meet the [general prerequisites for installing the Connected Machine agent](../servers/prerequisites.md).
- EC2 instances must have the AWS Systems Manager (SSM) agent  installed. Most EC2 instances have this preconfigured if you use a [supported OS](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html).

EC2 instances must also include the permissions described in the following section.

## Required AWS permissions

The **ArcForServerSSMRole** IAM role must be [attached on each EC2 instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/iam-roles-for-amazon-ec2.html#attach-iam-role) via the **ArcForServerSSMInstanceProfile** after the CloudFormation Template is deployed. This role includes the **AmazonSSMManagedInstanceCore** policy, which enables the SSM agent running on the EC2 instance to perform the actions required by Azure Arc for discovery, management, and onboarding into Azure Arc.

The **ArcForServerRole** IAM role is assumed by Azure Arc using OIDC federated identity. This role grants Azure Arc the ability to remotely trigger and monitor SSM commands, as well as retrieve EC2 and CloudFormation metadata—allowing it to manage the instance from outside AWS. This role includes the following permissions:

| Permission | Description |
|---|---|
| **ssm:SendCommand** | Allows sending commands to one or more managed instances. Used to execute onboarding scripts on the target machine. |
| **ssm:CancelCommand** | Allows canceling a command currently running on a managed instance. Used to stop onboarding commands if needed. |
| **ssm:DescribeInstanceInformation** | Allows viewing information about your managed instances. Used to identify EC2 instances that have the SSM agent installed and are managed by SSM, which is required for onboarding to Azure Arc. |
| **ssm:GetCommandInvocation** | Allows viewing the details and results of a command that was sent. Used to monitor the status and output of onboarding scripts. |
| **ec2:DescribeInstances** | Allows retrieving information about your EC2 instances. Used to check the instance's current state to determine eligibility for onboarding to Azure Arc. |
| **ec2:DescribeImages** | Allows retrieving information about your Amazon Machine Images (AMIs). Used to validate the operating system details of the instance. |
| **cloudformation:ListStackInstances** | Allows listing all stack instances in your AWS CloudFormation stacks. Used to identify resources created through CloudFormation that are relevant for onboarding. This permission is only required for accounts in an AWS Organization, but it's available by default to both management and member accounts. |

> [!NOTE]
> These permissions are included in the **ArcForServerRole** IAM role. While these permissions are provisioned through the CloudFormation Template, you should ensure there are no explicit deny policies in place, as they can override allowed permissions and prevent successful onboarding.

## Optional AWS permissions

In addition to the **ArcForServerRole** permissions, you can optionally grant permissions to automatically attach an SSM instance profile to EC2 machines. This simplifies Azure Arc onboarding by properly configuring the SSM agent without requiring manual intervention. If you don't enable these permissions, you must manually the SSM instance profile to your EC2 machines.

Configure the following parameters in your CloudFormation Template to control this behavior:

- `EC2SSMIAMRoleAutoAssignment` (default: `true`): Enables automatic assignment of the IAM role used for SSM tasks to EC2 instances. Set to false to disable this feature and manage instance profiles manually.
- `EC2SSMIAMRoleAutoAssignmentSchedule` (default: `Enable`): Controls whether the auto-assignment process runs periodically. Set to `Disable` to turn off scheduled checks.
- `EC2SSMIAMRoleAutoAssignmentScheduleInterval` (default: 1 day): Defines how frequently the auto-assignment Lambda function runs (e.g., 15 minutes, 6 hours, 1 day).
-  C2SSMIAMRolePolicyUpdateAllowed` (default: `true`): Allows the system to update existing IAM roles with required SSM permissions if they are missing.

To support automatic instance profile assignment, the following permissions must be allowed.

> [!TIP]
> These permissions are provisioned automatically if the Lambda-based auto-assignment (**EC2SSMIAMRoleAutoAssignment**) is enabled. Otherwise, you must manually ensure that the EC2 instances have the correct instance profile attached.

| Permission | Description |
|---|---|
| **ec2:DescribeInstances** | Allows viewing EC2 instances. |
| **ec2:DescribeRegions** | Allows retrieving available AWS regions. |
| **ec2:DescribeIamInstanceProfileAssociations** | Allows viewing existing IAM instance profile associations. |
| **ec2:AssociateIamInstanceProfile** | Allows attaching an instance profile to an EC2 instance. |
| **ec2:DisassociateIamInstanceProfile** | Allows detaching an instance profile from an EC2 instance. |
| **iam:GetInstanceProfile** | Allows retrieving instance profile information. |
| **iam:ListAttachedRolePolicies** | Allows viewing role policies attached to IAM roles. |
| **iam:AttachRolePolicy** | Allows attaching a policy to an IAM role. |
| **iam:PassRole** | Allows passing a role to a service like EC2. |
| **iam:AddRoleToInstanceProfile** | Allows adding a role to an instance profile. |
| **logs:CreateLogGroup**, **logs:CreateLogStream**, **logs:PutLogEvents** | Allows writing logs from Lambda to CloudWatch. |
| **ssm:GetServiceSetting** | Allows getting settings related to SSM services. |

## AWS resource representation in Azure

After you connect your AWS cloud and enable the **Arc onboarding** solution, the Multicloud Connector creates a new resource group with the naming convention `aws_yourAwsAccountId`.

When EC2 instances are connected to Azure Arc, representations of these machines appear in this resource group. These resources are placed in Azure regions, using a [standard mapping scheme](resource-representation.md#region-mapping). You can filter for which Azure regions you would like to scan for. By default, all regions are scanned, but you can choose to exclude certain regions when you [configure the solution](connect-to-aws.md#add-your-public-cloud-in-the-azure-portal).

The `aws_yourAwsAccountId` resource group inherits permissions from its subscription. You can grant additional access to user accounts in your tenant as needed to enable specific scenarios.  

## Connectivity method

When creating the [**Arc onboarding** solution](connect-to-aws.md#add-your-public-cloud-in-the-azure-portal), you select whether the Connected Machine agent should connect to the internet via a public endpoint or by proxy server. If you select **Proxy server**, you must provide a **Proxy server URL** to which the EC2 instance can connect.

For more information, see [Connected machine agent network requirements](../servers/network-requirements.md?tabs=azure-cloud).

## Periodic sync options

The periodic sync time that you select when configuring the **Arc onboarding** solution determines how often your AWS account is scanned and synced to Azure. By enabling periodic sync, whenever a new EC2 instance that meets the prerequisites is discovered, the Arc agent is automatically installed. The periodic sync option will also help clean up your resources in Azure. For instance, if the EC2 instance is removed from AWS, the Arc server in Azure will also be deleted. This is only applicable to Arc servers created in the aws_accountId resource group.

If you prefer, you can turn periodic sync off when configuring this solution. If you do so, new EC2 instances aren't automatically onboarded to Azure Arc, because Azure doesn't scan for new instances.

## EC2 Filter Options

You can choose to filter to scan for EC2 based on AWS regions or AWS tags. You can select which regions you would like to scan for EC2 resources. You can also filter by AWS tag to only onboard EC2 machines that have the matching tag (case-insensitive) to be eligible for EC2 onboarding.

## Next steps

- Learn more about [managing connected servers through Azure Arc](../servers/overview.md).
- Learn about the [Multicloud Connector **Inventory** solution](view-multicloud-inventory.md).
