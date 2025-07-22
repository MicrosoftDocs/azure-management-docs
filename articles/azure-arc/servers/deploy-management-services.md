---
title: Deploy Azure Management services to Arc-enabled servers at scale
description: Learn how to deploy Azure Management services to Arc-enabled servers at scale.
ms.date: 11/03/2024
ms.topic: how-to
# Customer intent: "As an IT administrator managing a fleet of Arc-enabled servers, I want to deploy Azure Management services at scale, so that I can efficiently monitor, manage updates, and ensure security across all machines."
---

# Deploy Azure Management services to Arc-enabled servers at scale (Preview)

This article explains how to enable multiple Azure Management Services across your entire fleet of Azure Arc-enabled machines in one streamlined workflow. You can quickly view available management services and configure them for scalable deployment across your machines. 

Currently, four management services are available for deployment at scale through this workflow:

- [Change Tracking and Inventory](/azure/automation/change-tracking/overview): Collect inventory and track changes.
- [Azure Update Manager](/azure/update-manager/overview): Manage and govern updates for all your machines.
- [Azure Monitor Insights](/azure/azure-monitor/insights/insights-overview): Monitor the operating system and any workloads on the machine.
- [Microsoft Defender for Cloud](/azure/defender-for-cloud/defender-for-cloud-introduction): Perform security monitoring in Azure.

To enable these services on your machines, use the following procedure:

1. From your browser, sign in to the [Azure portal](https://portal.azure.com/).

1. Navigate to **Machines-Azure Arc**.

1. Select the machines to which you want to deploy management services.

    :::image type="content" source="media/deploy-management-services/management-services-select-machines.png" alt-text="Screenshot of Azure portal showing Arc-enabled machines selected for Management Services." lightbox="media/deploy-management-services/management-services-select-machines.png":::
   
1. Select **Enable services (preview)**.

1. Select the management services you want deployed on the machines.

    :::image type="content" source="media/deploy-management-services/management-services-select-services.png" alt-text="Screenshot of Management Services that can be selected." lightbox="media/deploy-management-services/management-services-select-services.png":::

    The **Eligibility** column indicates if an extension is available for each machine. Machines where these extensions are already deployed may show as ineligible. Reasons for ineligibility can be viewed by selecting the link.

1. If necessary, select **Edit** to edit the configuration for a service (for example, Log Analytics workspaces).

1. Select **Next**, review the management services to be deployed to the machines, and then select **Enable**.



