---
title: Monitor your solutions with Azure portal as an IT Developer
description: Learn how to monitor your solutions with workload orchestration using the Azure portal as an IT Developer.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 05/01/2025
ms.custom:
  - build-2025
---

# Use Azure portal to monitor your solutions as an IT developer

IT developers can use the Azure portal to monitor their solutions. In the Azure portal, you can:

- View the applications on lines and their statuses.
- View alerts related to deployment.
- Trace the issues causing failures.

[!INCLUDE [public-preview-note](includes/public-preview-note.md)]

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) before you begin.

## Open the Azure portal

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. In the search box at the top of the page, type **Azure Arc** and select **Azure Arc** from the list of results.
1. In the left-side menu of the Azure Arc page, open the **Management** section and select **Workload Orchestration (preview)**.
1. The workload orchestration page has two tabs: **Deployment alerts** and **All resources**. Deployment alerts tab provides the list of issues to be debugged. All resources tab lists all the sites, K8 clusters and deployment targets.

    :::image type="content" source="./media/azure-portal-1.png" alt-text="Screenshot of the Azure portal showing the Workload Orchestration menu under management in Azure Arc." lightbox="./media/azure-portal-1.png":::

## Monitor deployment alerts

1. In the **Deployment alerts** tab, you can see the list of deployment alerts. The list shows the target name, type, and other details.
1. Click on the name of the target to view the details of the alert.

    :::image type="content" source="./media/azure-portal-2.png" alt-text="Screenshot of the Azure portal showing how to select a target in the deployment alerts of the Workload Orchestration menu." lightbox="./media/azure-portal-2.png":::

1. The alert summary shows the operation name, solution, fired time and affected targets. 

    :::image type="content" source="./media/azure-portal-3.png" alt-text="Screenshot of the Azure portal showing the summary alerts of a deployment failure." lightbox="./media/azure-portal-3.png":::

## Monitor all resources

1. In the **All resources** tab, the default view shows the flattened list of sites and targets.

    :::image type="content" source="./media/azure-portal-4.png" alt-text="Screenshot of the Azure portal showing how to change the view to monitor all resources in the Workload Orchestration menu." lightbox="./media/azure-portal-4.png":::

1. Click on **View solutions**. 

    :::image type="content" source="./media/azure-portal-5.png" alt-text="Screenshot of the Azure portal showing how to change the view to view solutions in the Workload Orchestration menu." lightbox="./media/azure-portal-5.png":::

1. The new view shows the list of solutions and statuses across targets.

    :::image type="content" source="./media/azure-portal-6.png" alt-text="Screenshot of the Azure portal showing the Workload Orchestration menu and all solutions." lightbox="./media/azure-portal-6.png":::

1. Click on **View targets** to go back to the previous list of targets.
1. Click on **Create site** to create a new site. For more information, see [Create a site in Azure Arc](/azure/azure-arc/site-manager/how-to-crud-site#create-a-site).

    :::image type="content" source="./media/azure-portal-7.png" alt-text="Screenshot of the Azure portal showing how to create a new site in the Workload Orchestration menu." lightbox="./media/azure-portal-7.png":::

## Related content

- [RBAC guide](rbac-guide.md)
- [Configuration template](configuring-template.md)
- [Configuration schema](configuring-schema.md)
- [Delete resources in workload orchestration](delete-resources.md)