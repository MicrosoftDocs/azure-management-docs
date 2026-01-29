---
title: View Security Compliance Status
description: This article details how to view the security baseline status for an Azure Arc site.
author: AgarkarNikhil
ms.author: nagarkar
ms.service: azure-arc
ms.topic: how-to
ms.date: 05/29/2025
ms.subservice: azure-arc-site-manager
---
# View compliance status for an Azure security baseline

This article shows you how to view the security baseline status for an Azure Arc site. A site's security baseline status reflects the compliance status of the underlying resources to the [Azure compute security baselines](/azure/governance/policy/samples/guest-configuration-baseline-windows).

## Prerequisites

- An Azure subscription. If you don't have a service subscription, create a [free trial account in Azure](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
- Azure portal access.
- Internet connectivity.
- A resource group or subscription or service group in Azure with at least one resource for a site. For more information, see [Supported resource types](/azure/azure-arc/site-manager/overview).
- A site created for the associated resource group or subscription or service group. If you don't have one, see [Create and manage sites](/azure/azure-arc/site-manager/how-to-crud-site).

## Security status and meanings

In the Azure portal, the color indicates the status:  

- **Green**: Means **Compliant**.
- **Red**: Means **Not compliant**.
- **Blue**: Means **Enable** (security baseline isn't yet enabled).
- **Gray**: Means **N/A** and stands for *not applicable* (security baseline isn't yet supported for the resource type).

## View security baseline status

You can view the security baseline status for an Azure Arc site as a whole from the main page of the Azure Arc site manager (preview).

1. In the [Azure portal](https://portal.azure.com/), go to **Azure Arc** and select **Site manager (preview)** to open the site manager.
1. From the Azure Arc site manager, go to the **Overview** page.

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-2.png" alt-text="Screenshot that shows going to the Overview page.":::

1. On the **Overview** page, you can view the summarized security statuses of your sites. This site-level status is aggregated from the statuses of its managed resources. You can see the security status with respect to the security baselines and Microsoft Defender for Cloud recommendations. In the following example, sites are shown with different statuses.

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-3.png" alt-text="Screenshot that shows viewing the summarized security statuses.":::

1. To understand which site has which status, select either the **Sites** tab or the status text to go to the **Sites** page.  

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-4.png" alt-text="Screenshot that shows going to the Sites tab.":::

1. On the **Sites** page, you can view the top-level status for each site. This site-level status reflects the most significant resource-level status for the site.

1. Select the **not compliant** link to view the resource details.  

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-6.png" alt-text="Screenshot that shows selecting the not compliant link.":::

1. On the site's resource page, you can view the security status for each resource within the site. This list also highlights the specific resources that are noncompliant, which contribute to the parent resource being marked as **Not compliant**.

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-7.png" alt-text="Screenshot that shows viewing the security status for each resource within the site.":::

## Enable security baseline status

The Azure Arc site manager (preview) provides a centralized view to see security status, but it doesn't provide security capabilities itself. Instead, customers can set up security baselines via Azure Policy. After these policies are configured, the Azure Arc site manager (preview) shows the relevant status within the site pages.

If you aren't familiar with Azure Arc security baselines, learn more about the [compute security baselines](/security/benchmark/azure/security-baselines-overview).

## Configure security baselines for sites in Azure Arc

This section provides basic steps for configuring the security baselines from the Azure Arc site manager. You can also do this step separately via Azure Policy by applying the relevant policy for [Windows](/azure/governance/policy/samples/guest-configuration-baseline-windows) or [Linux](/azure/governance/policy/samples/guest-configuration-baseline-linux).

1. Go to the individual site page.

   :::image type="content" source="media/view-security-compliance-status/how-to-enable-security-baseline-status-1.png" alt-text="Screenshot that shows going to the site.":::

1. Go to the **All resources** pane.

   :::image type="content" source="media/view-security-compliance-status/view-security-baseline-status-7.png" alt-text="Screenshot that shows going to the All resources pane.":::

1. Select **Enable** for the relevant policy.

   :::image type="content" source="media/view-security-compliance-status/how-to-enable-security-baseline-status-3.png" alt-text="Screenshot that shows enabling the relevant policy.":::

1. Select **Apply**, or if you want, select **Advanced settings**. The policy is applied to all resources within the scope of the site.

   :::image type="content" source="media/view-security-compliance-status/how-to-enable-security-baseline-status-4.png" alt-text="Screenshot that shows viewing Advanced settings.":::

1. The compliance status of each resource to the policy appears within the site manager in 5 to 10 minutes.  

   :::image type="content" source="media/view-security-compliance-status/how-to-enable-security-baseline-status-5.png" alt-text="Screenshot that shows how to enable the security baseline.":::
