---
title: Azure Resource Graph sample queries for Azure Arc
description: Sample Azure Resource Graph queries for Azure Arc showing use of resource types and tables to access Azure Arc related resources and properties.
ms.date: 05/14/2025
ms.topic: sample
ms.custom:
  - subject-resourcegraph-sample
  - build-2025
---

# Azure Resource Graph sample queries for Azure Arc

[Azure Resource Graph](/azure/governance/resource-graph/overview) is an Azure service that lets you query at scale, helping you effectively govern your environment. Queries are created using Kusto Query Language (KQL). For more information, see [Understanding the Azure Resource Graph query language](/azure/governance/resource-graph/concepts/query-language).

This page provides a list of sample Azure Resource Graph queries for Azure Arc. You can run these queries through Azure PowerShell or Azure CLI, or in the Azure portal using the Resource Graph Explorer. Feel free to modify the queries to suit your needs.

[!INCLUDE [azure-resource-graph-copilot](~/reusable-content/ce-skilling/azure/includes/azure-resource-graph-copilot.md)]

## General Arc sample queries

[!INCLUDE [azure-resource-graph-samples-cat-arc](./includes/azure-arc.md)]

## Arc-enabled servers sample queries

[!INCLUDE [azure-resource-graph-samples-cat-arc-servers](./includes/azure-arc-enabled-servers.md)]

## Arc-enabled Kubernetes sample queries

[!INCLUDE [azure-resource-graph-samples-cat-arck8s](./includes/azure-arc-enabled-kubernetes.md)]

## Next steps

- Learn more about the [query language](/azure/governance/resource-graph/concepts/query-language).
- Learn more about how to [explore resources](/azure/governance/resource-graph/concepts/explore-resources).
