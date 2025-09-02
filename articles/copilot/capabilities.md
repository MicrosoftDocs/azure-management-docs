---
title:  Microsoft Copilot in Azure capabilities
description: Learn about the things you can do with Microsoft Copilot in Azure.
ms.date: 09/02/2025
ms.topic: concept-article
ms.service: copilot-for-azure
ms.custom:
  - ignite-2023
  - ignite-2023-copilotinAzure
  - build-2024
  - ignite-2024
ms.author: jenhayes
author: JnHs
# Customer intent: As an Azure user, I want to utilize AI-driven assistance in Azure operations, so that I can optimize my app management, enhance troubleshooting, and efficiently navigate Azure services for improved performance and insights.
---

# Microsoft Copilot in Azure capabilities

Microsoft Copilot in Azure amplifies your impact with AI-enhanced operations. You can ask Copilot in Azure for help with designing, operating, optimizing, and troubleshooting your Azure apps and infrastructure. Copilot for Azure helps you gain new insights, discover more benefits of the cloud, and orchestrate data across both the cloud and the edge.

To access Copilot in Azure, you must be signed in to the Azure portal. Select the **Copilot** icon in the page header to open the Copilot pane. You can then enter your prompts in the text box, or try out one of the provided suggestions.

:::image type="content" source="media/capabilities/launch-copilot-azure-portal.png" alt-text="Screenshot showing the Copilot icon and pane in the Azure portal.":::

In some cases, suggested prompts appear near the top of a pane when working with a service or resource. These suggestions are tailored to the context of the service or resource you're working with to help you get started quickly.

You can also use Copilot in Azure from within the [Azure mobile app](../azure-portal/mobile-app/microsoft-copilot-in-azure.md).

The following sections describe some of the ways you can use Copilot in Azure.

## Perform tasks

Use Microsoft Copilot in Azure to perform many basic tasks. There are many things you can do! Take a look at these articles to learn about some of the scenarios in which Microsoft Copilot in Azure can be especially helpful.

- Understand your Azure environment:
  - [Get resource information through Azure Resource Graph queries](get-information-resource-graph.md)
  - [Understand service health events and status](understand-service-health.md)
  - [Analyze, estimate, and optimize costs](analyze-cost-management.md)
  - [Visualize network topology](visualize-network-topology.md)
  - [Query your attack surface](query-attack-surface.md)
  - [Investigate Azure Firewall IDPS attacks](/azure/firewall/firewall-copilot)
- Work smarter with Azure services:
  - [Execute commands](execute-commands.md)
  - [Deploy and manage virtual machines](deploy-vms-effectively.md)
  - [Discover and deploy workload templates](deploy-workload-templates.md)
  - [Work with AKS clusters efficiently](work-aks-clusters.md)
  - [Get information about Azure Monitor metrics and logs](get-monitoring-information.md)
  - [Work smarter with Azure Local](work-smarter-edge.md)
  - [Manage and troubleshoot storage accounts](improve-storage-accounts.md)
  - [Troubleshoot disk performance](troubleshoot-disk-performance.md)
  - [Design, troubleshoot, and secure networks](network-management.md)
  - [Improve Azure SQL Database-driven applications](/azure/azure-sql/copilot/copilot-azure-sql-overview#microsoft-copilot-for-azure-enhanced-scenarios)
  - [Discover Azure Marketplace solutions](discover-marketplace.md)
- Write and optimize code:
  - [Generate Azure CLI scripts](generate-cli-scripts.md)
  - [Generate PowerShell scripts](generate-powershell-scripts.md)
  - [Generate Terraform and Bicep configurations](generate-terraform-configurations.md)
  - [Author API Management policies](author-api-management-policies.md)
  - [Create Kubernetes YAML files](generate-kubernetes-yaml.md)
  - [Troubleshoot apps faster with App Service](troubleshoot-app-service.md)

## Get information

From anywhere in the Azure portal, you can ask Microsoft Copilot in Azure to explain more about Azure concepts, services, or offerings. You can ask questions to learn how a feature works, or which configurations best meet your budgets, security, and scale requirements. Copilot in Azurecan guide you to the right user experience, or even author scripts and other artifacts that you can use to deploy your solutions. Answers are grounded in the latest Azure documentation, so you can get up-to-date guidance just by asking a question.

## Solve problems and troubleshoot resources

Asking questions to understand more can be helpful when troubleshooting problems. For example, you can say things like "Cluster stuck in upgrading state while performing update operation" or "Azure database unable to connect from Power BI." Copilot in Azure responds with information about the problem and possible ways to fix it.

Copilot in Azure also helps you understand more about information presented in Azure. For example, when viewing diagnostic details for a resource, you can say "Give me a summary of this page" or "What's the issue with my app?" You can ask what an error means, or ask what the next steps would be to implement a recommended solution.

In some cases, Copilot in Azure can help resolve the issue. For example, if you say "Help me troubleshoot my Arc server extension," Copilot in Azure prompts you to select the Arc-enabled server and the extension that you are interested in troubleshooting. If the extension needs to be reinstalled, Copilot in Azure offers to help you reinstall it. Similarly, if you experience problems with your Azure Container Instances, you can ask things like "Why did my container restart?" After prompting you to select the container resource, Copilot in Azure provides steps to help resolve the issue.

Another area where Copilot in Azure can be especially helpful is troubleshooting [Azure Native observability solution integrations](/azure/partner-solutions/overview). For example, you can say "Why are metrics not reaching Datadog from my AKS cluster?" or "Dynatrace logs are missing for my App Service." Copilot in Azure executes relevant detectors on the target resource to identify issues, provides remediation solutions, and help you understand more about the problem.

Copilot in Azure can also help explain errors directly from the Notifications pane.

:::image type="content" source="media/capabilities/help-notifications.png" alt-text="Screenshot of Copilot in Azure providing a link to help troubleshoot an error in the Notifications pane.":::

## Find recommendations

Copilot in Azure can show Azure Advisor recommendations across the [Well Architected Framework](/azure/well-architected/what-is-well-architected-framework) pillars: Reliability, Cost Optimization, Operational Excellence, Performance Efficiency, and Security. You can ask about general Advisor recommendations, or get specific recommendations related to cost savings, performance improvements, security enhancements, or other areas.

For example, you can say "Show me my reliability recommendations." Copilot in Azure returns a list of top reliability recommendations for you, including links to each recommendation page in Advisor. You can optionally select specific resources for which to view associated recommendations.

You can also ask questions to get recommendations about services that help support your objectives. For instance, you can ask "What service would you recommend to implement distributed caching?" or "What are popular services used with Azure Container Apps?" Where applicable, Microsoft Copilot in Azure provides links to start working with the service or learn more. In some cases, you also see metrics about how often a service is used.

## Navigation

Rather than searching for a service to open, ask Microsoft Copilot in Azure to open the service for you. If you can't remember the exact name, choose from the provided suggestions, or ask Microsoft Copilot in Azure which service to use for your intended task.

## Manage portal settings

Use Microsoft Copilot in Azure to confirm your settings selection or change options, without having to open the **Settings** pane. For example, you can ask Copilot in Azure to change from Light to Dark theme, or vice versa.

## Current limitations

While Microsoft Copilot in Azure can perform many types of tasks, it might not be able to complete all requests. In these cases, Copilot in Azure typically explains the limitation and provides information about how you can carry out the intended action.

Keep in mind these general current limitations:

- You can't continue the same conversation beyond 24 hours.
- Any action taken on more than 10 resources must be performed outside of Microsoft Copilot in Azure.
- Some responses that display lists are limited to the top five items.
- For some tasks and queries, using a resource's name doesn't work, and the Azure resource ID must be provided.
- Excessive use of Copilot in Azure may result in temporary throttling of access to Copilot in Azure.

## Next steps

- [Get tips for writing effective prompts](write-effective-prompts.md) to use with Microsoft Copilot in Azure.
- Learn about [managing access to Copilot in Azure](manage-access.md) in your organization.
- Learn how to use Copilot in Azure with [AI Shell](ai-shell-overview.md) or [in the Azure mobile app](/azure/azure-portal/mobile-app/microsoft-copilot-in-azure).
