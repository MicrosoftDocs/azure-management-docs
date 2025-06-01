---
title: Troubleshooting for Workload Orchestration
description: Learn how to troubleshoot issues with Workload Orchestration.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: troubleshooting
ms.date: 06/01/2025
---

# Troubleshooting

## Troubleshoot service groups 

For [service group](service-group.md) related resources, only the user who created the resource can fetch/GET that resources due to RBAC limitations.

Fetch service groups.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'resourcecontainers | where type contains \"microsoft.management/serviceGroups\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
    --method post `
    --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
    --body "{'query': 'resourcecontainers | where type contains \'microsoft.management/serviceGroups\''}" `
    --resource https://management.azure.com
```
***

Fetch service group Sites.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'extensibilityresources | where type =~ \"microsoft.edge/sites\" | where id startswith \"/providers/microsoft.management/servicegroups\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
  --method post `
  --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
  --body "{'query': 'extensibilityresources | where type =~ \'microsoft.edge/sites\' | where id startswith \'/providers/microsoft.management/servicegroups\''}" `
  --resource https://management.azure.com
```
***

Fetch service group-relationships.

### [Bash](#tab/bash)

```bash
az rest \
    --method post \
    --url "https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01" \
    --body "{'query': 'relationshipresources | where type =~ \"microsoft.relationships/servicegroupmember\" | where id startswith \"$targetId\"'}" \
    --resource "https://management.azure.com"
```

### [PowerShell](#tab/powershell)

```powershell
az rest `
    --method post `
    --url https://southeastasia.management.azure.com/providers/Microsoft.ResourceGraph/resources?api-version=2024-04-01 `
    --body "{'query': 'relationshipresources | where type =~ \'microsoft.relationships/servicegroupmember\' | where id startswith \'$targetId\''}" `
    --resource https://management.azure.com
```
***