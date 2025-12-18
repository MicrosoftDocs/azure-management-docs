---
title: Azure Resource Graph sample queries for Azure Arc-enabled servers
description: Sample Azure Resource Graph queries for Azure Arc-enabled servers showing use of resource types and tables to access Azure Arc-enabled servers related resources and properties.
ms.date: 05/14/2025
ms.topic: sample
ms.custom:
  - subject-resourcegraph-sample
  - build-2025
# Customer intent: "As a cloud administrator, I want to run sample queries using Azure Resource Graph for Azure Arc-enabled servers, so that I can efficiently access and manage resource information across my environment."
---

# Azure Resource Graph sample queries for Azure Arc-enabled servers

[Azure Resource Graph](/azure/governance/resource-graph/overview) is an Azure service that lets you query at scale, helping you effectively govern your environment. Queries are created using Kusto Query Language (KQL). For more information, see [Understanding the Azure Resource Graph query language](/azure/governance/resource-graph/concepts/query-language).

This page provides a list of sample Azure Resource Graph queries for Azure Arc-enabled servers. These queries target the `microsoft.hybridcompute/machines` resource type and return information such as domain membership, installed extensions, agent versions, and OS details. You can run these queries through Azure PowerShell or Azure CLI, or in the Azure portal using the Resource Graph Explorer. Modify the queries to suit your needs.

[!INCLUDE [azure-lighthouse-supported-service](~/reusable-content/ce-skilling/azure/includes/azure-resource-graph-copilot.md)]

## Sample queries

[!INCLUDE [azure-resource-graph-samples-cat-arc-servers](../includes/azure-arc-enabled-servers.md)]

## Next steps

- Learn more about the [query language](/azure/governance/resource-graph/concepts/query-language).
- Learn more about how to [explore resources](/azure/governance/resource-graph/concepts/explore-resources).
