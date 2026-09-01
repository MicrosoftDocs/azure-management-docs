---
title: Azure Copilot Resiliency Agent (preview)
description: The Azure Copilot Resiliency Agent helps you strengthen the reliability of your Azure workloads and get recommendations for high availability, disaster recovery, and fault tolerance.
ms.date: 07/14/2026
ms.service: azure-copilot
ms.topic: concept-article
# Customer intent: "As an Azure Copilot user, I want to understand how to use the Resiliency Agent, so that I can improve the reliability of my Azure workloads."
---

# Azure Copilot Resiliency Agent (preview)

The Azure Copilot Resiliency Agent is an AI-powered assistant integrated into Azure Copilot that helps you design, assess, and improve the resiliency of your Azure workloads through natural language. Instead of navigating multiple tools and interpreting raw signals, you can interact with the agent conversationally to understand your resiliency posture, get actionable recommendations, and execute remediation - all from within the Azure portal.

> [!IMPORTANT]
> The Azure Copilot Resiliency Agent is currently in PREVIEW.
> See the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) for legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

To use the Resiliency Agent, select **Resiliency** from the **New chat** options in Azure Copilot.

> [!NOTE]
> Administrators can [enable or disable access to the Azure Copilot Resiliency Agent](manage-access.md#manage-user-access-to-azure-copilot-and-agents) and other [Azure Copilot agents](agents.md). If you don't see the Resiliency Agent as an option when starting a new chat, check with your administrator.

The agent supports two primary capability areas:

### Start Resilient — Design with resiliency from day one

Start Resilient helps you build resiliency into your applications before they are deployed. Describe your application architecture in natural language, and the agent will:

- Analyze your application topology and identify resiliency best practices relevant to your resources.
- Generate a detailed resiliency report with recommendations tailored to your architecture.
- Produce deployment-ready Bicep templates with the recommended resiliency configurations already enabled.

This capability is designed for teams that are building new applications or modernizing existing ones and want to ensure resiliency is a deployment default — not a remediation afterthought.

### Get Resilient — Assess and improve your existing workloads

Get Resilient helps you understand and improve the resiliency posture of workloads that are already running in Azure. The agent can:

- Check whether individual resources or entire application groups (Service Groups) are zonally resilient.
- Identify prerequisites and blockers for enabling resiliency on your resources.
- Provide guided remediation — including direct conversions where supported, or step-by-step migration guidance for more complex scenarios.
- Generate enablement scripts in your preferred format (PowerShell, Azure CLI, or Bicep templates).
- Help you organize resources into Service Groups, set resiliency goals, and track compliance over time.
- Surface indicative cost impact so you can make informed decisions about remediation.

This capability is designed for teams managing existing workloads who want to systematically close resiliency gaps and maintain compliance with their organization's resiliency objectives.

## How to Access the Resiliency Agent

There are two ways to launch the Resiliency Agent:

### Option 1: Via Azure Copilot

1. Open the Azure portal.
1. Navigate to the Resiliency overview page.
1. Select the **Copilot icon** in the top-right corner of the Azure portal header bar. This action opens the Copilot side panel.
1. In the Copilot panel, locate the **agent selector dropdown** near the top of the panel.
1. Select the dropdown and choose **"Resiliency"** from the list of available agents. Other options include Copilot, Troubleshooting, Deployment, and Optimization.
1. The panel header updates to show **"Resiliency agent"** and you see suggested starter prompts to begin your interaction.

### Option 2: Via top actions on the Resiliency screen

When you're on the Resiliency overview page, you see top action buttons that provide quick entry points into common resiliency workflows. When you select any of these actions, the portal automatically opens the Copilot panel with the Resiliency Agent selected and prepopulated with a relevant prompt.

> [!TIP] 
> After the agent loads, you can interact with it by using the suggested prompts or by typing your own questions in natural language. The agent maintains context across multiple turns, so you can have a continuous conversation that progresses from assessment to remediation.


## Start Resilient — scenarios and sample prompts

The Start Resilient capability helps you design resilient architectures from scratch. The following sections describe key scenarios you can explore, along with sample prompts to get started.

### Describe your application and get a resiliency plan

Tell the agent about your application's resources in natural language. The agent analyzes your topology, asks clarifying questions if needed (for example, region, configuration preferences), and generates a resiliency summary with a detailed recommendation report.

**Sample prompts:**

> *"My app has a VM, a PostgreSQL database, and a load balancer. Help me start my resilient journey."*

> *"I'm building an application with AKS, Redis, Storage, and Cosmos DB. What should I do to make it resilient?"*

> *"I have a web app with an App Service, SQL Database, and a Front Door. Can you assess the resiliency?"*

### Generate resilient Bicep templates

Ask the agent to generate deployment-ready Bicep templates for your resources with the recommended resiliency configurations already enabled. The agent produces modular templates including `main.bicep`, `parameters.json`, and per-resource files.

**Sample prompts:**

> *"Generate a Bicep template for a zonally resilient VM setup."*

> *"Create Bicep templates for a resilient architecture with VMs, AKS, and Storage."*

> *"Generate deployment templates for my application with resiliency best practices included."*

### Larger application architectures

The agent can handle more complex application descriptions with multiple resource types. It produces a consolidated resiliency summary and detailed recommendations across all resources.

**Sample prompts:**

> *"My app includes VMs, AKS, Redis, Storage, and Cosmos DB. Help me design a resilient setup."*

> *"Generate Bicep templates for a zonally resilient architecture with VMs, AKS, and Storage."*

## Get Resilient — Scenarios and Sample Prompts

The Get Resilient capability helps you assess and improve the resiliency of your existing Azure workloads. The following sections describe key scenarios you can explore.

### Check resource resiliency

Ask the agent to check whether a specific resource or your entire service group is zonally resilient. The agent returns the current resiliency status along with supporting configuration details.

**Sample prompts:**

> *"Is my VM zonally resilient?"*

> *"Check the resiliency of my storage account."*

> *"Get the zonal resiliency posture of my service group."*

### Check prerequisites for resiliency

Before enabling resiliency, you might need to address certain prerequisites. The agent can identify blocking conditions such as region limitations, permission requirements, or configuration dependencies.

**Sample prompts:**

> *"What are the prerequisites to make my VM zonally resilient?"*

> *"What do I need to do before enabling zone redundancy on my database?"*

### Enable resiliency on your resources

For resources that support in-place conversion (mutable resources), the agent can directly enable zonal resiliency. For resources that require redeployment (such as VMs), the agent provides step-by-step guidance and scripts.

**Sample prompts:**

> *"Enable zonal resiliency for my PostgreSQL server."*

> *"Enable zonal resiliency for my VM."*

> *"How do I make my storage account zone-redundant?"*

> [!IMPORTANT]
> For some resource types, the agent executes the change directly. For others (like VMs), it provides guidance and scripts rather than making changes, since these resources require redeployment.

### Generate enablement scripts

The agent can generate scripts to enable or create zonally resilient resources in multiple formats, including PowerShell, Azure CLI, Bicep, Terraform, and ARM templates.

**Sample prompts:**

> *"Give me scripts to enable zonal resiliency for my resources."*

> *"Generate scripts to create a zonally resilient storage account."*

> *"Generate scripts for all resources in my service group."*

### Organize resources into service groups

Use service groups to logically group the Azure resources that make up an application. The agent can help you create service groups, add resources to them, and view their membership.

**Sample prompts:**

> *"Create a service group for my application."*

> *"Add my VM and database to the service group."*

> *"Show the resources in my service group."*

### Set resiliency goals and track compliance

After you create a service group, set resiliency goals and track how your resources comply over time. The agent supports creating goals, checking compliance status, excluding specific resources, and refreshing compliance after changes.

**Sample prompts:**

> *"Set a zonal resiliency goal for my service group."*

> *"What is my goal compliance status?"*

> *"Exclude this VM from my resiliency goal."*

> *"Refresh my goal compliance after recent changes."*

### End-to-end guided workflow

The agent supports multi-turn conversations that progress naturally from assessment to remediation. You can start by checking your posture, then ask for scripts to fix problems, and continue through the full resiliency lifecycle in a single conversation.

**Example conversation flow:**

1. Start with: *"Create a service group for my app"*
1. Then: *"Set resiliency goals"*
1. Then: *"Check posture"*
1. Then: *"Fix the problems"* or *"Give me scripts to fix it"*

> [!TIP]
> The agent maintains context across turns, so each step builds on the previous one. You don't need to repeat resource names or configuration details.

## Known limitations

As this feature is in public preview, be aware of the following limitations:

- The agent is optimized for zonal resiliency scenarios. Regional resiliency and other resiliency dimensions will be added in future releases.
- Template generation currently supports Bicep format. Support for additional IaC formats in the Start Resilient flow is planned.
- Some operations might take a few moments to complete, especially for larger service groups.
