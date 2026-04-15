---
title: Onboard VMs to Azure Arc through the multicloud connector
description: Learn how to enable the Arc onboarding solution with the multicloud connector enabled by Azure Arc.
ms.topic: how-to
ms.date: 04/13/2026
# Customer intent: As a cloud administrator, I want to onboard EC2 instances and GCP VMs to Azure Arc using the multicloud connector, so that I can manage AWS, GCP, and Azure VMs centrally and leverage Azure management services.
---

# Onboard VMs to Azure Arc through the multicloud connector

The **Arc onboarding** solution of the multicloud connector autodiscovers VMs in a [connected public cloud](add-public-cloud.md), then installs the [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview) to onboard the VMs to Azure Arc. This simplified experience lets you use Azure management services, such as Azure Monitor, providing a centralized way to manage Azure, AWS, and GCP VMs together.

Currently, the multicloud connector provides support for connecting resources from these public clouds:

- Amazon Web Services (AWS)
- Google Cloud Platform (GCP) (preview)

You can enable the **Arc onboarding** solution when you [connect your public cloud to Azure](add-public-cloud.md).

## Prerequisites

In addition to the [general prerequisites for connecting a public cloud](add-public-cloud.md#prerequisites), be sure to meet the requirements for the **Arc onboarding** solution. This includes requirements for each EC2 instance or GCP VM to be onboarded to Azure Arc.

### [AWS](#tab/aws)

AWS prerequisites include the following:

- You must have **AmazonEC2FullAccess** permissions in your public cloud.
- EC2 instances must meet the [general prerequisites for installing the Connected Machine agent](../servers/prerequisites.md).
- EC2 instances must have the AWS Systems Manager (SSM) agent installed. Most EC2 instances are preconfigured with this agent if you use a [supported OS](https://docs.aws.amazon.com/systems-manager/latest/userguide/setup-instance-permissions.html).
- EC2 instances can't already have the Azure Arc Connected Machine agent installed. If the Connected Machine agent is already present on the EC2 instance, the onboarding process will skip that machine, even if the machine is in a Disconnected state. To onboard such machines, remediate the issue on the machine (for example, by restoring connectivity, restarting the agent, or reinstalling the agent if required). After remediation, the machine should reconnect to Azure Arc without requiring a new onboarding attempt.

As part of the Arc onboarding solution, Azure deploys an AWS Lambda function in your AWS account to help assign the required SSM Instance IAM role to eligible EC2 instances. This behavior is configurable, and you can opt out if you prefer to manage IAM role assignment manually. This Lambda function runs on a schedule and may incur minimal AWS costs, depending on how frequently it is configured to execute. By default, the function runs once per day. You can modify the schedule to run more frequently (down to every 15 minutes), which might increase AWS costs accordingly.

### [GCP](#tab/gcp)

GCP prerequisites include the following:

- You must have **OSPolicyAssignment Admin** permissions in your public cloud.
- GCP VMs must meet the [general prerequisites for installing the Connected Machine agent](../servers/prerequisites.md).
- GCP VMs must have the GCP VM Manager installed.

> [!NOTE]
> The Azure Arc process uses the VM manager on GCP to enforce policies on your VMs through the OS Config agent. A VM that has an active OS Config agent incurs a cost according to GCP. To see how this cost might affect your account, refer to the GCP technical documentation.
>
> The **Arc Onboarding** solution doesn't install the OS Config agent to a VM that doesn't have it installed. However, the **Arc Onboarding** solution enables communication between the OS Config agent and the OS Config service if the agent is already installed but not communicating with the service. This communication can change the OS Config agent from inactive to active, which could lead to more costs.

---

## Resource representation in Azure

After you connect your AWS cloud and enable the **Arc onboarding** solution, the multicloud connector creates a new resource group with the naming convention `<PublicCloud>_<AccountId>`.

When EC2 instances or GCP VMs are connected to Azure Arc, [representations of these machines](resource-representation.md) appear in this resource group. These resources are placed in Azure regions, using a [standard mapping scheme](resource-representation.md#region-mapping). You can filter for which Azure regions you would like to scan for. By default, all regions are scanned, but you can choose to exclude certain regions when you [configure the solution](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal).

The `<PublicCloud>_<AccountId>` resource group inherits permissions from its subscription. You can grant additional access to user accounts in your tenant as needed to enable specific scenarios.

## Connectivity method

When creating the [**Arc onboarding** solution](add-public-cloud.md#add-your-public-cloud-in-the-azure-portal), you select whether the Connected Machine agent should connect to the internet via a public endpoint or by proxy server. If you select **Proxy server**, you must provide a **Proxy server URL** to which the EC2 instance or GCP VM can connect. For more information, see [Connected machine agent network requirements](../servers/network-requirements.md?tabs=azure-cloud).

You can also select an Arc gateway resource to handle the connection for your Arc-enabled servers. Arc gateway reduces the number of endpoints that must be allowed in your environment to use Azure Arc. For more information, see [Simplify network configuration requirements with Azure Arc gateway](/azure/azure-arc/servers/arc-gateway).

## Periodic sync options

The periodic sync time that you select when configuring the **Arc onboarding** solution determines how often your source cloud is scanned and synced to Azure. By enabling periodic sync, whenever a new EC2 instance or GCP VM that meets the prerequisites is discovered, the Arc agent is automatically installed. If Arc onboarding fails for an eligible machine (for example, due to insufficient disk space or a transient installation error), the machine is automatically retried during the next periodic sync, provided it still meets the onboarding prerequisites. The connector doesn't permanently mark a failed machine as excluded, so on each subsequent periodic sync, the connector reevaluates eligible machines and reattempts Arc onboarding until the operation succeeds or the machine no longer meets the configured prerequisites.

Machines that already have the Azure Arc Connected Machine agent installed but are in a disconnected state aren't retried during subsequent periodic syncs; these machinese will be reconnected to Azure Arc after you fix the underlying issue by restoring connectivity, restarting the agent, or reinstalling the agent.

The periodic sync option also helps clean up your resources in Azure. For instance, if the EC2 instance or GCP VM is removed from the source cloud, the corresponding Arc server created in the `<PublicCloud>_<AccountId>` resource group in Azure is also deleted.

If you prefer, you can turn periodic sync off when configuring this solution. If you do so, new EC2 instances and GCP VMs aren't automatically onboarded to Azure Arc, because Azure doesn't scan for new instances.

## Filter options

You can choose to filter to scan for EC2 instances or GCP VMs based on:

- EC2 instances based on AWS regions or AWS tags.
- GCP VMs based on GCP regions or GCP labels.

You can select specific regions to scan for AWS or GCP resources. You can also filter by AWS tags or GCP labels so that only machines that have the matching tag or label (case-insensitive) are eligible for Arc onboarding.

## Next steps

- Learn more about [managing connected servers through Azure Arc](../servers/overview.md).
- Learn about the [multicloud connector **Inventory** solution](view-multicloud-inventory.md).
