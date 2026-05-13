---
title: Workload orchestration resource model
description: Understand the resource model of workload orchestration, including context, hierarchy, and target, and how they define deployment topology.
author: nathmanish
ms.author: nathmanish
ms.topic: conceptual
ms.date: 05/10/2026
# Customer intent: As a platform engineer, I want to understand the resource model of workload orchestration, so that I can model environments, organize infrastructure, and deploy workloads effectively.
---

# Workload orchestration resource model

Workload orchestration uses a set of core resource types to model environments, define deployment topology, and enable application deployment across distributed clusters. These resources work together to represent both the organizational structure and the deployment endpoints for applications.

The resource model consists of three primary building blocks:

- **Context** – Defines the overall environment and resource boundary
- **Hierarchy** – Represents the physical structure of your organization
- **Target** – Represents the deployment unit or namespace within a Kubernetes cluster

## Context

A context, also referred to as an environment, is the top-level resource that groups all workload orchestration resources within an Azure tenant. It links to your organizational hierarchy and defines the set of capabilities available for deployment. Capability tags are name-description pairs that describe what a resource is capable of doing. They're used to map applications to be deployed, to the relevant targets.

Each Azure tenant supports a single context, which ensures consistency in how resources are organized and managed.

## Hierarchy

A hierarchy represents the organizational structure of your environment using level-based definitions such as *factory*, *region*, or *line*. It maps closely to the physical layout of your operations.

Hierarchies are defined as name-description pairs that correspond to levels in your organization. For example:

- Factory → Line
- Region → City → Store

Hierarchies are implemented using **Sites**, which represent physical entities such as plants, factories, or stores. All levels of a hierarchy, except the leaf level, must be associated to a Site. Sites must be tagged with the correct hierarchy level so that they're interpreted correctly by workload orchestration.

Two hierarchy models are supported:

### Resource group hierarchy

Resource group hierarchy is used for simpler, two-level structures. This model uses a single parent Site at the parent (for example, factory, store, city, etc.) level, along with a child level beneath it. It is suitable for:

- Proof-of-concept environments that don't require more than two hierarchy levels
- Scenarios where you don't need to logically group resources beyond your existing resource group boundaries

### Service group hierarchy

Service group hierarchy is used for more complex, multi-level organizational structures. Service groups are tenant-level Azure Resource Manager (ARM) resource containers that allow you to organize related resources—such as resource groups, subscriptions, and deployment targets—under a unified logical structure, without changing the existing Azure resource layout. Relationships among service groups involved in the hierarchy are used to define parent-child relationships within your Site hierarchy. A service group hierarchy is suitable for:

- Enterprise-scale deployments requiring you to model real-world organizational hierarchies (for example, Region → City → Factory → Line)
- Multi-region or multi-site topologies
- Scenarios requiring grouping of resources across multiple resource groups

In this model:

- Each level (except the leaf) is backed by a service group in addition to a Site.
- Up to four levels of hierarchy can be modeled.

## Target

A target represents the deployment unit within a Kubernetes cluster. It corresponds to a namespace within an Arc-enabled cluster where applications are deployed, and is associated with a **custom location** created on your cluster. To be able to use a target for deployment, it needs to be mapped to a level and Site of your hierarchy at the time of creation. A target is also tagged with capabilities that determine which solutions can be deployed on it.

## How the resource model works together

The three resource types work together to define both structure and deployment flow:

1. The **context** establishes the environment and defines available capabilities.
2. The **hierarchy** maps the organizational structure and physical layout using sites.
3. The **target** defines where workloads are deployed within Kubernetes clusters.
