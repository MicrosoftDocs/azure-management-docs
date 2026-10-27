---
title: Set up workload orchestration using Azure portal
description: Learn how to create and manage a workload orchestration environment, hierarchy, and targets in the Azure portal.
author: nathmanish
ms.author: nathmanish
ms.topic: install-set-up-deploy
ms.date: 08/25/2026
ai-usage: ai-assisted
# Customer intent: As an IT administrator, I want to onboard to workload orchestration in the Azure portal, so that I can create an environment, hierarchy, and deployment targets without using command-line tools.
---

# Set up workload orchestration using Azure portal

This article shows you how to set up workload orchestration in the Azure portal. Use this guide to define a custom production-scale environment with an associated site hierarchy and targets. For a test or proof-of-concept environment with minimal configuration and quick sample application deployment, use the [Express Start](quickstart-azure-portal.md) guide.

> [!TIP]
> For alternative setup methods, see [Set up using CLI](set-up-workload-orchestration.md) for a command line interface, [Set up using scripts](onboarding-scripts.md) for automated provisioning, or [Set up using Git](workload-orchestration-multicluster-git.md) for a declarative Git-based workflow.

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Permissions to create and manage workload orchestration resources. For more information, see the [Role-Based Access Control (RBAC) guide](rbac-guide.md).
- An Azure Arc-enabled Kubernetes cluster. For more information, see [Quickstart: Connect an existing Kubernetes cluster to Azure Arc](../kubernetes/quickstart-connect-cluster.md). The workload orchestration Arc extension doesn't support Arm-based architecture nodes. Ensure your cluster uses a non-Arm virtual machine and meets the following requirements:

  | Cluster type | RAM | CPUs | Disk |
    | --- | --- | --- | --- |
  | Single-node K3s | Minimum 4 GB | 2 | Not applicable |
  | Multinode Kubernetes | Minimum 4 GB per node | 2 per node | Extra 1 GB |

- A custom location associated with the workload orchestration extension on your Arc-enabled cluster. To prepare a cluster and create a custom location, see [Set up the workload orchestration environment](set-up-workload-orchestration.md#set-up-the-workload-orchestration-environment).

> [!NOTE]
> Each Azure tenant supports one workload orchestration environment. If your tenant already has an environment, manage its resources from the workload orchestration portal instead of creating another environment.

## Set up your environment

### Open the setup wizard

1. Go to the [workload orchestration](https://aka.ms/configManagerGA) page in Azure Arc.
1. On the **Environments** tab, select **Get started**.
1. Under **Regular Setup**, select **Start**.

    :::image type="content" source="./media/it-portal-regular-selection.png" alt-text="Screenshot of the Create an Environment page with the Start button highlighted under Regular Setup." lightbox="./media/it-portal-regular-selection.png":::

### Configure the environment basics

1. On the **Basics** tab, enter an **Environment** name.
1. Select the **Subscription**, **Resource group**, and **Region** for the workload orchestration resources.
1. Under **Azure Arc-enabled Kubernetes cluster**, select the **ARC enabled cluster** and its **Custom location**.
1. Select **Validate**. Continue after the portal confirms that the workload orchestration extension is enabled on the custom location.
1. Select **Next: Environment configurations**.

	:::image type="content" source="./media/it-portal-regular-basics.png" alt-text="Screenshot of the custom environment Basics tab showing the environment, Arc-enabled cluster, custom location, and successful extension validation." lightbox="./media/it-portal-regular-basics.png":::

### Define the hierarchy levels

1. On the **Environment configurations** tab, enter a unique name for each hierarchy level, for example, `level-1` and `level-2`. You can also use names that represent your physical topology, such as *Region*, *Factory*, and *Line*.
1. Select **Next: Environment details**.

    :::image type="content" source="./media/it-portal-regular-hierarchy.png" alt-text="Screenshot of the Environment configurations tab showing a two-level site structure and fields for L0, L1, and L2 hierarchy level names." lightbox="./media/it-portal-regular-hierarchy.png":::

For more information about hierarchy levels, sites, and targets, see the [workload orchestration resource model](resource-model.md).

### Add sites and targets

1. On the **Environment details** tab, use **Add level-1** and **Add level-2** to add sites to your corresponding [hierarchy](resource-model.md#hierarchy) levels. One site is added to each level by default, except the leaf level, which is level-3 in this case.
1. Use the edit icon next to a site or level to change its name.
1. To proceed, add at least one [target](resource-model.md#target) under any site or the leaf level. Next to the site or level where you want to create the deployment endpoint, select **Add Target**.

	:::image type="content" source="./media/it-portal-regular-targets.png" alt-text="Screenshot of the Environment details tab showing controls to add hierarchy levels and targets." lightbox="./media/it-portal-regular-targets.png":::

1. In the **Create target** pane, enter the following values:

   - **Target Name**: Enter the Azure resource name for the target.
   - **Display Name**: Enter the name shown in the workload orchestration portal.
   - **Description**: Describe the workloads or physical endpoint represented by the target.
   - **ARC enabled cluster** and **Custom location**: Confirm the cluster and custom location selected for the environment.
   - **Solution scope**: Enter the Kubernetes cluster namespace where solutions for this target are deployed. Don't use a system namespace such as `azure-arc`, `kube-system`, `workloadorchestration`, or `cert-manager`.
   - **Capability tags**: Select existing tags or create a new tag by entering the name and then selecting **Create "&lt;tag-name&gt;" tag**. Capability tags map solutions to compatible targets.

	:::image type="content" source="./media/it-portal-regular-add-target.png" alt-text="Screenshot of the Create target pane showing target details and the option to create a capability tag." lightbox="./media/it-portal-regular-add-target.png":::

1. Confirm that the capability tags appear under **Added tags**, and then select **Create**.

    :::image type="content" source="./media/it-portal-environment-setup-target-capability-tags.png" alt-text="Screenshot of the Create target pane showing an added capability tag and the Create button." lightbox="./media/it-portal-environment-setup-target-capability-tags.png":::

1. Repeat these steps for each target in your environment. When the hierarchy includes all required sites and targets, select **Review+Create**.
1. After the portal validates all details on the **Review** tab, select **Review+Create** to start the deployment.

	:::image type="content" source="./media/it-portal-regular-create.png" alt-text="Screenshot of the custom environment Review tab showing the project, environment, hierarchy, and target details." lightbox="./media/it-portal-regular-create.png":::

When the deployment finishes, the portal lists the environment on the **Environments** tab. You can view all generated resources in the resource group you selected. To configure and deploy applications to your target, follow [Deploy a basic solution](solution-without-common-configuration.md).

## Add a site hierarchy

You can associate additional hierarchies by linking the parent site of an existing hierarchy to your environment. If you don't have a site, create one in Azure Arc Site Manager before continuing.

1. Open your workload orchestration environment in the Azure portal.

    :::image type="content" source="./media/it-portal-environments-overview.png" alt-text="Screenshot of the environment grid page." lightbox="./media/it-portal-environments-overview.png":::

1. On **Overview**, under **Add sites to hierarchy**, select **Add**.

    :::image type="content" source="./media/it-portal-add-sites-to-hierarchy.png" alt-text="Screenshot of the environment Overview page with Add highlighted under Add sites to hierarchy." lightbox="./media/it-portal-add-sites-to-hierarchy.png":::

1. In the **Add/Create site** pane, select an existing parent site. To create a site, select **Create new in Site manager**, complete the Site Manager flow, and then return to this pane.

    :::image type="content" source="./media/it-portal-select-parent-site.png" alt-text="Screenshot of the Add or Create site pane showing the parent Site selector and the link to create a Site in Site Manager." lightbox="./media/it-portal-select-parent-site.png":::

1. After you add the parent site, select **Save**.

    :::image type="content" source="./media/it-portal-save-parent-site.png" alt-text="Screenshot of the Add or Create site pane with a parent Site selected and the Save button highlighted." lightbox="./media/it-portal-save-parent-site.png":::

1. Confirm that the portal displays a notification that the site was linked to the environment.

    :::image type="content" source="./media/it-portal-site-linked-notification.png" alt-text="Screenshot of the environment Overview page showing a notification that the Site was linked successfully." lightbox="./media/it-portal-site-linked-notification.png":::

## Create a target

You can create more targets that point to namespaces of your Arc clusters. Follow these steps to create a target:

1. On the environment **Overview** page, select **Add** under **Add targets**, or switch to the **Targets** tab and select **Create**.

    :::image type="content" source="./media/it-portal-add-target-from-overview.png" alt-text="Screenshot of the environment Overview page with Add highlighted under Add targets." lightbox="./media/it-portal-add-target-from-overview.png":::

    :::image type="content" source="./media/it-portal-create-target-from-targets-page.png" alt-text="Screenshot of the environment Targets page with Create highlighted." lightbox="./media/it-portal-create-target-from-targets-page.png":::

1. Enter a **Target Name**, **Display Name**, and optional **Description**.
1. Select the **Region** and **ARC enabled cluster**.
1. Select the **Custom location**, and then select **Validate**. Continue after the portal confirms that the workload orchestration extension is enabled on the selected custom location.
1. Select the **Hierarchy Level** and **parent site** for the target.
1. For **Solution scope**, enter the Kubernetes namespace where solutions for this target are deployed.

    :::image type="content" source="./media/it-portal-create-target-details.png" alt-text="Screenshot of the Create target pane showing target details, cluster settings, validation, hierarchy placement, and solution scope." lightbox="./media/it-portal-create-target-details.png":::

1. Under **Capability tags**, select one or more tags that map the target to compatible solutions, and then select **Create**.

    :::image type="content" source="./media/it-portal-target-validation-capability-tags.png" alt-text="Screenshot of the Create target pane showing successful custom location validation, selected capability tags, and the Create button." lightbox="./media/it-portal-target-validation-capability-tags.png":::

## Next steps

> [!div class="nextstepaction"]
> [Deploy a basic solution](solution-without-common-configuration.md)
