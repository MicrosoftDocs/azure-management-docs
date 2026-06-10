---
title: Workload orchestration architecture
description: Learn about the architecture of workload orchestration on Azure Arc and how resources, control plane, and edge clusters interact.
ms.custom:
  - references_regions
  - build-2025
author: nathmanish
ms.author: nathmanish
ms.topic: conceptual
ms.date: 05/10/2026
# Customer intent: "As an IT admin, I want to onboard onto workload orchestration."
---

# Workload orchestration architecture

Workload orchestration provides a centralized control plane for managing application deployment and configuration across distributed environments. This article walks you through the architecture designed to handle all the capabilities of this tool.

## Architecture overview

The workload orchestration architecture consists of a control plane in Azure and a distributed data plane across Arc-enabled Kubernetes clusters, as shown:

:::image type="content" source="./media/workload-orchestration-architecture.png" alt-text="Diagram of the workload orchestration architecture." lightbox="./media/workload-orchestration-architecture.png":::

Users can interact with workload orchestration through any of the three interfaces: portal, Azure CLI, and SDK. At its core, the cloud-based control plane uses a dedicated Azure Resource Manager as the primary entry point for all operations. All client requests processed by Azure Resource Manager are passed on to the workload orchestration service that validates the configurations and prepares the deployment templates. These templates are then transferred to the edge environment through the Kubernetes bridge. The workload orchestration Arc agents running on the on-premises Kubernetes clusters consume these templates, deploy the user workloads, and apply the necessary customizations for each site.

## Contact support

[!INCLUDE [form-feedback-note](includes/form-feedback.md)]