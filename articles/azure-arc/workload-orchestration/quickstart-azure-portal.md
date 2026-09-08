---
title: "Quickstart: Set up workload orchestration using Azure portal"
description: Quickly create a workload orchestration environment, target, and optional sample application by using the Azure portal.
author: nathmanish
ms.author: nathmanish
ms.topic: quickstart
ms.date: 08/27/2026
ai-usage: ai-assisted
# Customer intent: As an IT administrator, I want to quickly create a basic workload orchestration environment and deploy a sample application by using the Azure portal.
---

# Quickstart: Set up workload orchestration

This article shows you how to quickly set up a basic workload orchestration environment and deploy a sample application. Use this flow for testing or proof-of-concept environments. For production-scale environments with custom hierarchy and configuration needs, see [Set up workload orchestration using Azure portal](set-up-workload-orchestration-azure-portal.md).

## Prerequisites

- An Azure subscription. If you don't have an Azure subscription, [create one for free](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
- Permissions to create and manage workload orchestration resources. For more information, see the [Role-Based Access Control (RBAC) guide](rbac-guide.md).
- An Azure Arc-enabled Kubernetes cluster. For more information, see [Quickstart: Connect an existing Kubernetes cluster to Azure Arc](../kubernetes/quickstart-connect-cluster.md). The workload orchestration Arc extension doesn't support Arm-based architecture nodes. Ensure your cluster uses a non-Arm virtual machine and meets the following requirements:

  | Cluster type | RAM | CPUs | Disk |
   | --- | --- | --- | --- |
  | Single-node K3s | Minimum 4 GB | 2 | Not applicable |
  | Multinode Kubernetes | Minimum 4 GB per node | 2 per node | Extra 1 GB |

> [!NOTE]
> Each Azure tenant supports one workload orchestration environment. If your tenant already has an environment, manage its resources from the workload orchestration portal instead of creating another environment.

## Open express start

1. Go to the [workload orchestration](https://aka.ms/configManagerGA) page in Azure Arc.
1. On the **Environments** tab, select **Get started**.

   :::image type="content" source="./media/azure-portal-workload-orchestration-homepage.png" alt-text="Screenshot of the workload orchestration page in Azure Arc with the Get started button highlighted." lightbox="./media/azure-portal-workload-orchestration-homepage.png":::

1. Under **Express Start**, select **Try Express Start**.

   :::image type="content" source="./media/azure-portal-onboarding-options.png" alt-text="Screenshot of the Create an Environment page showing the Express Start and Regular Setup options." lightbox="./media/azure-portal-onboarding-options.png":::

## Enter the environment details

1. On the **Basics** tab, enter an [Environment](resource-model.md#context) name.
1. Select the **Subscription**, **Resource group**, and **Region** for the workload orchestration resources.
1. Under **Azure Arc-enabled Kubernetes cluster**, select your Arc-enabled cluster or create a cluster.
1. Review the generated **Custom location**. If the selected cluster doesn't have a custom location associated with the workload orchestration extension, Azure creates one for you.
1. Enter a **Target name**. The target represents the deployment endpoint in the selected cluster.
1. To deploy the Cluster Health Dashboard sample application as part of onboarding, select **Deploy sample application (preview)**.
1. Select **Next: Review & create**.

   :::image type="content" source="./media/it-portal-quickstart.png" alt-text="Screenshot of the Express Start Basics tab showing environment, cluster, target, and sample application settings." lightbox="./media/it-portal-quickstart.png":::

## Review and create the environment

1. On the **Review & create** tab, confirm the environment, Arc-enabled cluster, custom location, target, and optional application details.
1. After validation passes, select **Review+Create** to start the deployment.

   :::image type="content" source="./media/it-portal-quickstart-validation.png" alt-text="Screenshot of the Express Start Review and create tab showing successful validation and the resources to be created." lightbox="./media/it-portal-quickstart-validation.png":::

When the deployment finishes, the environment appears on the **Environments** tab. You can view all generated resources in the resource group you selected.

:::image type="content" source="./media/it-portal-environment-created.png" alt-text="Screenshot of the workload orchestration Environments tab showing the newly created Express Start environment." lightbox="./media/it-portal-environment-created.png":::

## Validate the sample application

If you deployed the sample application, validate it from a terminal:

1. Connect to the Kubernetes cluster.

   ```azurecli
   az connectedk8s proxy -g <resource-group> -n <arc-cluster-name>
   ```

1. In another terminal, find the application service and its port.

   ```azurecli
   kubectl get svc -n <namespace>
   ```

1. Forward the service to your device.

   ```azurecli
   kubectl port-forward svc/<service-name> 3000:<service-port> -n <namespace>
   ```

1. Open <http://localhost:3000> in a browser.

## Next steps

For a more custom onboarding experience with advanced configurations, see [Set up workload orchestration using Azure portal](set-up-workload-orchestration-azure-portal.md). To start deploying your own applications using workload orchestration, see [Deploy a basic solution](solution-without-common-configuration.md)
