---
title:  Microsoft Copilot in Azure capabilities
description: Learn about the things you can do with Microsoft Copilot in Azure.
ms.date: 04/28/2025
ms.topic: concept-article
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
  - ignite-2024
ms.author: jenhayes
author: JnHs
---

# Microsoft Copilot in Azure capabilities

Microsoft Copilot in Azure amplifies your impact with AI-enhanced operations. You can ask Copilot in Azure for help with designing, operating, optimizing, and troubleshooting your Azure apps and infrastructure. Copilot for Azure can help you gain new insights, discover more benefits of the cloud, and orchestrate data across both the cloud and the edge.

To access Copilot in Azure, you must be signed in to the Azure portal. Select the **Copilot** icon in the page header to open the Copilot pane. You can then enter your prompts in the text box, or try out one of the provided suggestions.

:::image type="content" source="media/capabilities/launch-copilot-azure-portal.png" alt-text="Screenshot showing the Copilot icon and pane in the Azure portal.":::

In some cases, suggested prompts appear near the top of a pane when working with a service or resource. These suggestions are tailored to the context of the service or resource you're working with to help you get started quickly.

You can also use Copilot in Azure from within the [Azure mobile app](../azure-portal/mobile-app/microsoft-copilot-in-azure.md).

The following sections describe some common ways you can use Copilot in Azure.

## Perform tasks

Use Microsoft Copilot in Azure to perform many basic tasks. There are many things you can do! Take a look at these articles to learn about some of the scenarios in which Microsoft Copilot in Azure can be especially helpful.

- Understand your Azure environment:
  - [Get resource information through Azure Resource Graph queries](get-information-resource-graph.md)
  - [Understand service health events and status](understand-service-health.md)
  - [Analyze, estimate, and optimize costs](analyze-cost-management.md)
  - [Find Azure Advisor recommendations](find-recommendations-advisor.md)
  - [Visualize network topology](visualize-network-topology.md)
  - [Query your attack surface](query-attack-surface.md)
  - [Investigate Azure Firewall IDPS attacks](/azure/firewall/firewall-copilot)
- Work smarter with Azure services:
  - [Execute commands](execute-commands.md)
  - [Deploy virtual machines effectively](deploy-vms-effectively.md)
  - [Discover and deploy workload templates](deploy-workload-templates.md)
  - [Work with AKS clusters efficiently](work-aks-clusters.md)
  - [Get information about Azure Monitor metrics and logs](get-monitoring-information.md)
  - [Work smarter with Azure Local](work-smarter-edge.md)
  - [Manage and troubleshoot storage accounts](improve-storage-accounts.md)
  - [Troubleshoot disk performance](troubleshoot-disk-performance.md)
  - [Design, troubleshoot, and secure networks](network-management.md)
  - [Troubleshoot Azure Arc extension issues](troubleshoot-arc-extension.md)
  - [Improve Azure SQL Database-driven applications](/azure/azure-sql/copilot/copilot-azure-sql-overview#microsoft-copilot-for-azure-enhanced-scenarios)
- Write and optimize code:
  - [Generate Azure CLI scripts](generate-cli-scripts.md)
  - [Generate PowerShell scripts](generate-powershell-scripts.md)
  - [Generate Terraform and Bicep configurations](generate-terraform-configurations.md)
  - [Author API Management policies](author-api-management-policies.md)
  - [Create Kubernetes YAML files](generate-kubernetes-yaml.md)
  - [Troubleshoot apps faster with App Service](troubleshoot-app-service.md)

## Get information

From anywhere in the Azure portal, you can ask Microsoft Copilot in Azure to explain more about Azure concepts, services, or offerings. You can ask questions to learn how a feature works, or which configurations best meet your budgets, security, and scale requirements. Copilot can guide you to the right user experience or even author scripts and other artifacts that you can use to deploy your solutions. Answers are grounded in the latest Azure documentation, so you can get up-to-date guidance just by asking a question.

## Solve problems

Asking questions to understand more can be especially helpful when you're troubleshooting problems. Describe the problem, and Microsoft Copilot in Azure will provide some suggestions on how you might be able to resolve the issue. For example, you can say things like "Cluster stuck in upgrading state while performing update operation" or "Azure database unable to connect from Power BI". Copilot in Azure will show information about the problem and possible resolution options.

Copilot in Azure can also help you understand more about information presented in Azure. This can be especially helpful when looking at diagnostic details. For example, when viewing diagnostics for a resource, you can say "Give me a summary of this page" or "What's the issue with my app?" You can ask what an error means, or ask what the next steps would be to implement a recommended solution.

Copilot in Azure can also help explain errors from the Notifications pane.

:::image type="content" source="media/capabilities/help-notifications.png" alt-text="Screenshot of Copilot in Azure providing a link to help troubleshoot an error in the Notifications pane.":::

## Find recommended services

Ask questions to learn which services are best suited for your workloads, or get ideas about additional services that might help support your objectives. For instance, you can ask "What service would you recommend to implement distributed caching?" or "What are popular services used with Azure Container Apps?" Where applicable, Microsoft Copilot in Azure provides links to start working with the service or learn more. In some cases, you also see metrics about how often a service is used. You can also ask additional questions to find out more about the service and whether it's right for your needs.

## Navigation

Rather than searching for a service to open, simply ask Microsoft Copilot in Azure to open the service for you. If you can't remember the exact name, choose from the provided suggestions or ask Microsoft Copilot in Azure which service to use for your intended task.

## Manage portal settings

Use Microsoft Copilot in Azure to confirm your settings selection or change options, without having to open the **Settings** pane. For example, you can ask Copilot in Azure to change from Light to Dark theme, or vice versa. You can also ask Copilot in Azure to make changes to your language or region settings.

## Current limitations

While Microsoft Copilot in Azure can perform many types of tasks, it's important to understand what not to expect. In some cases, Microsoft Copilot in Azure might not be able to complete your request. In these cases, you'll generally see an explanation along with more information about how you can carry out the intended action.

Keep in mind these current limitations:

- You can't continue the same conversation beyond 24 hours.
- Any action taken on more than 10 resources must be performed outside of Microsoft Copilot in Azure.
- Some responses that display lists are limited to the top five items.
- For some tasks and queries, using a resource's name doesn't work, and the Azure resource ID must be provided.
- Excessive use of Copilot in Azure may result in temporary throttling of access to Copilot in Azure.

## Next steps

- [Get tips for writing effective prompts](write-effective-prompts.md) to use with Microsoft Copilot in Azure.
- Learn about [managing access to Copilot in Azure](manage-access.md) in your organization.
- Learn how to use Copilot in Azure with [AI Shell](ai-shell-overview.md) or [in the Azure mobile app](/azure/azure-portal/mobile-app/microsoft-copilot-in-azure).
- Explore the [Microsoft Copilot in Azure video series](/shows/microsoft-copilot-in-azure/).
