---
ms.service: azure-arc
ms.topic: include
ms.date: 12/19/2024
author: JnHs
ms.author: jenhayes
---

### Get count and percentage of Arc-enabled servers by domain

This query summarizes the `domainName` property on [Azure Arc-enabled servers](../servers/overview.md) and uses a calculation with `bin` to create a `Pct` column for the percent of Arc-enabled servers per domain.

```kusto
Resources
| where type == 'microsoft.hybridcompute/machines'
| project domain=tostring(properties.domainName)
| summarize Domains=make_list(domain), TotalMachineCount=sum(1)
| mvexpand EachDomain = Domains
| summarize PerDomainMachineCount = count() by tostring(EachDomain), TotalMachineCount
| extend Pct = 100 * bin(todouble(PerDomainMachineCount) / todouble(TotalMachineCount), 0.001)
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.hybridcompute/machines' | project domain=tostring(properties.domainName) | summarize Domains=make_list(domain), TotalMachineCount=sum(1) | mvexpand EachDomain = Domains | summarize PerDomainMachineCount = count() by tostring(EachDomain), TotalMachineCount | extend Pct = 100 * bin(todouble(PerDomainMachineCount) / todouble(TotalMachineCount), 0.001)"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.hybridcompute/machines' | project domain=tostring(properties.domainName) | summarize Domains=make_list(domain), TotalMachineCount=sum(1) | mvexpand EachDomain = Domains | summarize PerDomainMachineCount = count() by tostring(EachDomain), TotalMachineCount | extend Pct = 100 * bin(todouble(PerDomainMachineCount) / todouble(TotalMachineCount), 0.001)"
```

# [Portal](#tab/azure-portal)

- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%20domain%3dtostring(properties.domainName)%0a%7c%20summarize%20Domains%3dmake_list(domain)%2c%20TotalMachineCount%3dsum(1)%0a%7c%20mvexpand%20EachDomain%20%3d%20Domains%0a%7c%20summarize%20PerDomainMachineCount%20%3d%20count()%20by%20tostring(EachDomain)%2c%20TotalMachineCount%0a%7c%20extend%20Pct%20%3d%20100%20*%20bin(todouble(PerDomainMachineCount)%20%2f%20todouble(TotalMachineCount)%2c%200.001)" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%20domain%3dtostring(properties.domainName)%0a%7c%20summarize%20Domains%3dmake_list(domain)%2c%20TotalMachineCount%3dsum(1)%0a%7c%20mvexpand%20EachDomain%20%3d%20Domains%0a%7c%20summarize%20PerDomainMachineCount%20%3d%20count()%20by%20tostring(EachDomain)%2c%20TotalMachineCount%0a%7c%20extend%20Pct%20%3d%20100%20*%20bin(todouble(PerDomainMachineCount)%20%2f%20todouble(TotalMachineCount)%2c%200.001)" target="_blank">portal.azure.us</a>
- Microsoft Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%20domain%3dtostring(properties.domainName)%0a%7c%20summarize%20Domains%3dmake_list(domain)%2c%20TotalMachineCount%3dsum(1)%0a%7c%20mvexpand%20EachDomain%20%3d%20Domains%0a%7c%20summarize%20PerDomainMachineCount%20%3d%20count()%20by%20tostring(EachDomain)%2c%20TotalMachineCount%0a%7c%20extend%20Pct%20%3d%20100%20*%20bin(todouble(PerDomainMachineCount)%20%2f%20todouble(TotalMachineCount)%2c%200.001)" target="_blank">portal.azure.cn</a>

---

### List all extensions installed on an Azure Arc-enabled server

First, this query uses `project` on the hybrid machine resource type to get the ID in uppercase (`toupper()`), get the computer name, and the operating system running on the machine. Getting the resource ID in uppercase is a good way to prepare to `join` to another property. Then, the query uses `join` with `kind` as `leftouter` to get extensions by matching an uppercase `substring` of the extension ID. The portion of the ID before `/extensions/<ExtensionName>` is the same format as the hybrid machine ID, so we use this property for the `join`. `summarize` is then used with `make_list` on the name of the virtual machine extension to combine the name of each extension where _ID_, _OSName_, and _ComputerName_ are the same into a single array property. Lastly, we order by lowercase _OSName_ with `asc`. By default, `order by` is descending.

```kusto
Resources
| where type == 'microsoft.hybridcompute/machines'
| project
  id,
  JoinID = toupper(id),
  ComputerName = tostring(properties.osProfile.computerName),
  OSName = tostring(properties.osName)
| join kind=leftouter(
  Resources
  | where type == 'microsoft.hybridcompute/machines/extensions'
  | project
    MachineId = toupper(substring(id, 0, indexof(id, '/extensions'))),
    ExtensionName = name
) on $left.JoinID == $right.MachineId
| summarize Extensions = make_list(ExtensionName) by id, ComputerName, OSName
| order by tolower(OSName) asc
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "Resources | where type == 'microsoft.hybridcompute/machines' | project id, JoinID = toupper(id), ComputerName = tostring(properties.osProfile.computerName), OSName = tostring(properties.osName) | join kind=leftouter( Resources | where type == 'microsoft.hybridcompute/machines/extensions' | project  MachineId = toupper(substring(id, 0, indexof(id, '/extensions'))),  ExtensionName = name ) on \$left.JoinID == \$right.MachineId | summarize Extensions = make_list(ExtensionName) by id, ComputerName, OSName | order by tolower(OSName) asc"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "Resources | where type == 'microsoft.hybridcompute/machines' | project id, JoinID = toupper(id), ComputerName = tostring(properties.osProfile.computerName), OSName = tostring(properties.osName) | join kind=leftouter( Resources | where type == 'microsoft.hybridcompute/machines/extensions' | project  MachineId = toupper(substring(id, 0, indexof(id, '/extensions'))),  ExtensionName = name ) on $left.JoinID == $right.MachineId | summarize Extensions = make_list(ExtensionName) by id, ComputerName, OSName | order by tolower(OSName) asc"
```

# [Portal](#tab/azure-portal)

- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%0a%09id%2c%0a%09JoinID%20%3d%20toupper(id)%2c%0a%09ComputerName%20%3d%20tostring(properties.osProfile.computerName)%2c%0a%09OSName%20%3d%20tostring(properties.osName)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%2fextensions%27%0a%09%7c%20project%0a%09%09MachineId%20%3d%20toupper(substring(id%2c%200%2c%20indexof(id%2c%20%27%2fextensions%27)))%2c%0a%09%09ExtensionName%20%3d%20name%0a)%20on%20%24left.JoinID%20%3d%3d%20%24right.MachineId%0a%7c%20summarize%20Extensions%20%3d%20make_list(ExtensionName)%20by%20id%2c%20ComputerName%2c%20OSName%0a%7c%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%0a%09id%2c%0a%09JoinID%20%3d%20toupper(id)%2c%0a%09ComputerName%20%3d%20tostring(properties.osProfile.computerName)%2c%0a%09OSName%20%3d%20tostring(properties.osName)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%2fextensions%27%0a%09%7c%20project%0a%09%09MachineId%20%3d%20toupper(substring(id%2c%200%2c%20indexof(id%2c%20%27%2fextensions%27)))%2c%0a%09%09ExtensionName%20%3d%20name%0a)%20on%20%24left.JoinID%20%3d%3d%20%24right.MachineId%0a%7c%20summarize%20Extensions%20%3d%20make_list(ExtensionName)%20by%20id%2c%20ComputerName%2c%20OSName%0a%7c%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.us</a>
- Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/Resources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%7c%20project%0a%09id%2c%0a%09JoinID%20%3d%20toupper(id)%2c%0a%09ComputerName%20%3d%20tostring(properties.osProfile.computerName)%2c%0a%09OSName%20%3d%20tostring(properties.osName)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%2fextensions%27%0a%09%7c%20project%0a%09%09MachineId%20%3d%20toupper(substring(id%2c%200%2c%20indexof(id%2c%20%27%2fextensions%27)))%2c%0a%09%09ExtensionName%20%3d%20name%0a)%20on%20%24left.JoinID%20%3d%3d%20%24right.MachineId%0a%7c%20summarize%20Extensions%20%3d%20make_list(ExtensionName)%20by%20id%2c%20ComputerName%2c%20OSName%0a%7c%20order%20by%20tolower(OSName)%20asc" target="_blank">portal.azure.cn</a>

---

### List Arc-enabled servers not running latest released agent version

This query returns all Arc-enabled servers running an outdated version of the Connected Machine agent. Agents with a status of **Expired** are excluded from the results. The query uses `leftouter` `join` to bring together the Advisor recommendations raised about any Connected Machine agents identified as out of date, and Hybrid Computer machines to filter out any agent that hasn't communicated with Azure over a period of time.

```kusto
AdvisorResources
| where type == 'microsoft.advisor/recommendations'
| where properties.category == 'HighAvailability'
| where properties.shortDescription.solution == 'Upgrade to the latest version of the Azure Connected Machine agent'
| project
    id,
    JoinId = toupper(properties.resourceMetadata.resourceId),
    machineName = tostring(properties.impactedValue),
    agentVersion = tostring(properties.extendedProperties.installedVersion),
    expectedVersion = tostring(properties.extendedProperties.latestVersion)
| join kind=leftouter(
  Resources
  | where type == 'microsoft.hybridcompute/machines'
  | project
    machineId = toupper(id),
    status = tostring (properties.status)
  ) on $left.JoinId == $right.machineId
| where status != 'Expired'
| summarize by id, machineName, agentVersion, expectedVersion
| order by tolower(machineName) asc
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "AdvisorResources | where type == 'microsoft.advisor/recommendations' | where properties.category == 'HighAvailability' | where properties.shortDescription.solution == 'Upgrade to the latest version of the Azure Connected Machine agent' | project  id,  JoinId = toupper(properties.resourceMetadata.resourceId),  machineName = tostring(properties.impactedValue),  agentVersion = tostring(properties.extendedProperties.installedVersion),  expectedVersion = tostring(properties.extendedProperties.latestVersion) | join kind=leftouter( Resources | where type == 'microsoft.hybridcompute/machines' | project  machineId = toupper(id),  status = tostring (properties.status) ) on \$left.JoinId == \$right.machineId | where status != 'Expired' | summarize by id, machineName, agentVersion, expectedVersion | order by tolower(machineName) asc"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "AdvisorResources | where type == 'microsoft.advisor/recommendations' | where properties.category == 'HighAvailability' | where properties.shortDescription.solution == 'Upgrade to the latest version of the Azure Connected Machine agent' | project  id,  JoinId = toupper(properties.resourceMetadata.resourceId),  machineName = tostring(properties.impactedValue),  agentVersion = tostring(properties.extendedProperties.installedVersion),  expectedVersion = tostring(properties.extendedProperties.latestVersion) | join kind=leftouter( Resources | where type == 'microsoft.hybridcompute/machines' | project  machineId = toupper(id),  status = tostring (properties.status) ) on $left.JoinId == $right.machineId | where status != 'Expired' | summarize by id, machineName, agentVersion, expectedVersion | order by tolower(machineName) asc"
```

# [Portal](#tab/azure-portal)

- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/AdvisorResources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.advisor%2frecommendations%27%0a%7c%20where%20properties.category%20%3d%3d%20%27HighAvailability%27%0a%7c%20where%20properties.shortDescription.solution%20%3d%3d%20%27Upgrade%20to%20the%20latest%20version%20of%20the%20Azure%20Connected%20Machine%20agent%27%0a%7c%20project%0a%09%09id%2c%0a%09%09JoinId%20%3d%20toupper(properties.resourceMetadata.resourceId)%2c%0a%09%09machineName%20%3d%20tostring(properties.impactedValue)%2c%0a%09%09agentVersion%20%3d%20tostring(properties.extendedProperties.installedVersion)%2c%0a%09%09expectedVersion%20%3d%20tostring(properties.extendedProperties.latestVersion)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%09%7c%20project%0a%09%09machineId%20%3d%20toupper(id)%2c%0a%09%09status%20%3d%20tostring%20(properties.status)%0a%09)%20on%20%24left.JoinId%20%3d%3d%20%24right.machineId%0a%7c%20where%20status%20!%3d%20%27Expired%27%0a%7c%20summarize%20by%20id%2c%20machineName%2c%20agentVersion%2c%20expectedVersion%0a%7c%20order%20by%20tolower(machineName)%20asc" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/AdvisorResources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.advisor%2frecommendations%27%0a%7c%20where%20properties.category%20%3d%3d%20%27HighAvailability%27%0a%7c%20where%20properties.shortDescription.solution%20%3d%3d%20%27Upgrade%20to%20the%20latest%20version%20of%20the%20Azure%20Connected%20Machine%20agent%27%0a%7c%20project%0a%09%09id%2c%0a%09%09JoinId%20%3d%20toupper(properties.resourceMetadata.resourceId)%2c%0a%09%09machineName%20%3d%20tostring(properties.impactedValue)%2c%0a%09%09agentVersion%20%3d%20tostring(properties.extendedProperties.installedVersion)%2c%0a%09%09expectedVersion%20%3d%20tostring(properties.extendedProperties.latestVersion)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%09%7c%20project%0a%09%09machineId%20%3d%20toupper(id)%2c%0a%09%09status%20%3d%20tostring%20(properties.status)%0a%09)%20on%20%24left.JoinId%20%3d%3d%20%24right.machineId%0a%7c%20where%20status%20!%3d%20%27Expired%27%0a%7c%20summarize%20by%20id%2c%20machineName%2c%20agentVersion%2c%20expectedVersion%0a%7c%20order%20by%20tolower(machineName)%20asc" target="_blank">portal.azure.us</a>
- Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/AdvisorResources%0a%7c%20where%20type%20%3d%3d%20%27microsoft.advisor%2frecommendations%27%0a%7c%20where%20properties.category%20%3d%3d%20%27HighAvailability%27%0a%7c%20where%20properties.shortDescription.solution%20%3d%3d%20%27Upgrade%20to%20the%20latest%20version%20of%20the%20Azure%20Connected%20Machine%20agent%27%0a%7c%20project%0a%09%09id%2c%0a%09%09JoinId%20%3d%20toupper(properties.resourceMetadata.resourceId)%2c%0a%09%09machineName%20%3d%20tostring(properties.impactedValue)%2c%0a%09%09agentVersion%20%3d%20tostring(properties.extendedProperties.installedVersion)%2c%0a%09%09expectedVersion%20%3d%20tostring(properties.extendedProperties.latestVersion)%0a%7c%20join%20kind%3dleftouter(%0a%09Resources%0a%09%7c%20where%20type%20%3d%3d%20%27microsoft.hybridcompute%2fmachines%27%0a%09%7c%20project%0a%09%09machineId%20%3d%20toupper(id)%2c%0a%09%09status%20%3d%20tostring%20(properties.status)%0a%09)%20on%20%24left.JoinId%20%3d%3d%20%24right.machineId%0a%7c%20where%20status%20!%3d%20%27Expired%27%0a%7c%20summarize%20by%20id%2c%20machineName%2c%20agentVersion%2c%20expectedVersion%0a%7c%20order%20by%20tolower(machineName)%20asc" target="_blank">portal.azure.cn</a>

---

### List Arc-enabled servers with SQL Server, PostgreSQL, or MySQL installed

This query returns all Arc-enabled servers that have SQL Server, PostgreSQL, or MySQL installed. 

```kusto
resources
| where type =~ 'microsoft.hybridcompute/machines'
| extend machineId = tolower(tostring(id)), datacenter = iif(isnull(tags.Datacenter), '', tags.Datacenter), status = tostring(properties.status)
| extend mssqlinstalled = coalesce(tobool(properties.detectedProperties.mssqldiscovered),false)
| extend pgsqlinstalled = coalesce(tobool(properties.detectedProperties.pgsqldiscovered),false)
| extend mysqlinstalled = coalesce(tobool(properties.detectedProperties.mysqldiscovered),false)
| extend osSku = properties.osSku, osName = properties.osName, osVersion = properties.osVersion
| extend coreCount = tostring(properties.detectedProperties.logicalCoreCount), totalPhysicalMemoryinGB = tostring(properties.detectedProperties.totalPhysicalMemoryInGigabytes) 
| extend operatingSystem = iif(isnotnull(osSku), osSku, osName)
| where mssqlinstalled or mysqlinstalled or pgsqlinstalled
| project id ,name, type, resourceGroup, subscriptionId, location, kind, osVersion, status, osSku,coreCount,totalPhysicalMemoryinGB,tags, mssqlinstalled, mysqlinstalled, pgsqlinstalled
| sort by (tolower(tostring(name))) asc
```

# [Azure CLI](#tab/azure-cli)

```azurecli-interactive
az graph query -q "resources | where type =~ 'microsoft.hybridcompute/machines' | extend machineId = tolower(tostring(id)), datacenter = iif(isnull(tags.Datacenter), '', tags.Datacenter), status = tostring(properties.status) | extend mssqlinstalled = coalesce(tobool(properties.detectedProperties.mssqldiscovered),false) | extend pgsqlinstalled = coalesce(tobool(properties.detectedProperties.pgsqldiscovered),false) | extend mysqlinstalled = coalesce(tobool(properties.detectedProperties.mysqldiscovered),false) | extend osSku = properties.osSku, osName = properties.osName, osVersion = properties.osVersion | extend coreCount = tostring(properties.detectedProperties.logicalCoreCount), totalPhysicalMemoryinGB = tostring(properties.detectedProperties.totalPhysicalMemoryInGigabytes)  | extend operatingSystem = iif(isnotnull(osSku), osSku, osName) | where mssqlinstalled or mysqlinstalled or pgsqlinstalled | project id ,name, type, resourceGroup, subscriptionId, location, kind, osVersion, status, osSku,coreCount,totalPhysicalMemoryinGB,tags, mssqlinstalled, mysqlinstalled, pgsqlinstalled | sort by (tolower(tostring(name))) asc"
```

# [Azure PowerShell](#tab/azure-powershell)

```azurepowershell-interactive
Search-AzGraph -Query "resources | where type =~ 'microsoft.hybridcompute/machines' | extend machineId = tolower(tostring(id)), datacenter = iif(isnull(tags.Datacenter), '', tags.Datacenter), status = tostring(properties.status) | extend mssqlinstalled = coalesce(tobool(properties.detectedProperties.mssqldiscovered),false) | extend pgsqlinstalled = coalesce(tobool(properties.detectedProperties.pgsqldiscovered),false) | extend mysqlinstalled = coalesce(tobool(properties.detectedProperties.mysqldiscovered),false) | extend osSku = properties.osSku, osName = properties.osName, osVersion = properties.osVersion | extend coreCount = tostring(properties.detectedProperties.logicalCoreCount), totalPhysicalMemoryinGB = tostring(properties.detectedProperties.totalPhysicalMemoryInGigabytes)  | extend operatingSystem = iif(isnotnull(osSku), osSku, osName) | where mssqlinstalled or mysqlinstalled or pgsqlinstalled | project id ,name, type, resourceGroup, subscriptionId, location, kind, osVersion, status, osSku,coreCount,totalPhysicalMemoryinGB,tags, mssqlinstalled, mysqlinstalled, pgsqlinstalled | sort by (tolower(tostring(name))) asc"
```

# [Portal](#tab/azure-portal)

- Azure portal: <a href="https://portal.azure.com/#blade/HubsExtension/ArgQueryBlade/query/resources%0A%7C%20where%20type%20%3D~%20%27microsoft.hybridcompute%2Fmachines%27%0A%7C%20extend%20machineId%20%3D%20tolower%28tostring%28id%29%29%2C%20datacenter%20%3D%20iif%28isnull%28tags.Datacenter%29%2C%20%27%27%2C%20tags.Datacenter%29%2C%20status%20%3D%20tostring%28properties.status%29%0A%7C%20extend%20mssqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mssqldiscovered%29%2Cfalse%29%0A%7C%20extend%20pgsqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.pgsqldiscovered%29%2Cfalse%29%0A%7C%20extend%20mysqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mysqldiscovered%29%2Cfalse%29%0A%7C%20extend%20osSku%20%3D%20properties.osSku%2C%20osName%20%3D%20properties.osName%2C%20osVersion%20%3D%20properties.osVersion%0A%7C%20extend%20coreCount%20%3D%20tostring%28properties.detectedProperties.logicalCoreCount%29%2C%20totalPhysicalMemoryinGB%20%3D%20tostring%28properties.detectedProperties.totalPhysicalMemoryInGigabytes%29%20%0A%7C%20extend%20operatingSystem%20%3D%20iif%28isnotnull%28osSku%29%2C%20osSku%2C%20osName%29%0A%7C%20where%20mssqlinstalled%20or%20mysqlinstalled%20or%20pgsqlinstalled%0A%7C%20project%20id%20%2Cname%2C%20type%2C%20resourceGroup%2C%20subscriptionId%2C%20location%2C%20kind%2C%20osVersion%2C%20status%2C%20osSku%2CcoreCount%2CtotalPhysicalMemoryinGB%2Ctags%2C%20mssqlinstalled%2C%20mysqlinstalled%2C%20pgsqlinstalled%0A%7C%20sort%20by%20%28tolower%28tostring%28name%29%29%29%20asc" target="_blank">portal.azure.com</a>
- Azure Government portal: <a href="https://portal.azure.us/#blade/HubsExtension/ArgQueryBlade/query/resources%0A%7C%20where%20type%20%3D~%20%27microsoft.hybridcompute%2Fmachines%27%0A%7C%20extend%20machineId%20%3D%20tolower%28tostring%28id%29%29%2C%20datacenter%20%3D%20iif%28isnull%28tags.Datacenter%29%2C%20%27%27%2C%20tags.Datacenter%29%2C%20status%20%3D%20tostring%28properties.status%29%0A%7C%20extend%20mssqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mssqldiscovered%29%2Cfalse%29%0A%7C%20extend%20pgsqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.pgsqldiscovered%29%2Cfalse%29%0A%7C%20extend%20mysqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mysqldiscovered%29%2Cfalse%29%0A%7C%20extend%20osSku%20%3D%20properties.osSku%2C%20osName%20%3D%20properties.osName%2C%20osVersion%20%3D%20properties.osVersion%0A%7C%20extend%20coreCount%20%3D%20tostring%28properties.detectedProperties.logicalCoreCount%29%2C%20totalPhysicalMemoryinGB%20%3D%20tostring%28properties.detectedProperties.totalPhysicalMemoryInGigabytes%29%20%0A%7C%20extend%20operatingSystem%20%3D%20iif%28isnotnull%28osSku%29%2C%20osSku%2C%20osName%29%0A%7C%20where%20mssqlinstalled%20or%20mysqlinstalled%20or%20pgsqlinstalled%0A%7C%20project%20id%20%2Cname%2C%20type%2C%20resourceGroup%2C%20subscriptionId%2C%20location%2C%20kind%2C%20osVersion%2C%20status%2C%20osSku%2CcoreCount%2CtotalPhysicalMemoryinGB%2Ctags%2C%20mssqlinstalled%2C%20mysqlinstalled%2C%20pgsqlinstalled%0A%7C%20sort%20by%20%28tolower%28tostring%28name%29%29%29%20asc" target="_blank">portal.azure.us</a>
- Azure operated by 21Vianet portal: <a href="https://portal.azure.cn/#blade/HubsExtension/ArgQueryBlade/query/resources%0A%7C%20where%20type%20%3D~%20%27microsoft.hybridcompute%2Fmachines%27%0A%7C%20extend%20machineId%20%3D%20tolower%28tostring%28id%29%29%2C%20datacenter%20%3D%20iif%28isnull%28tags.Datacenter%29%2C%20%27%27%2C%20tags.Datacenter%29%2C%20status%20%3D%20tostring%28properties.status%29%0A%7C%20extend%20mssqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mssqldiscovered%29%2Cfalse%29%0A%7C%20extend%20pgsqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.pgsqldiscovered%29%2Cfalse%29%0A%7C%20extend%20mysqlinstalled%20%3D%20coalesce%28tobool%28properties.detectedProperties.mysqldiscovered%29%2Cfalse%29%0A%7C%20extend%20osSku%20%3D%20properties.osSku%2C%20osName%20%3D%20properties.osName%2C%20osVersion%20%3D%20properties.osVersion%0A%7C%20extend%20coreCount%20%3D%20tostring%28properties.detectedProperties.logicalCoreCount%29%2C%20totalPhysicalMemoryinGB%20%3D%20tostring%28properties.detectedProperties.totalPhysicalMemoryInGigabytes%29%20%0A%7C%20extend%20operatingSystem%20%3D%20iif%28isnotnull%28osSku%29%2C%20osSku%2C%20osName%29%0A%7C%20where%20mssqlinstalled%20or%20mysqlinstalled%20or%20pgsqlinstalled%0A%7C%20project%20id%20%2Cname%2C%20type%2C%20resourceGroup%2C%20subscriptionId%2C%20location%2C%20kind%2C%20osVersion%2C%20status%2C%20osSku%2CcoreCount%2CtotalPhysicalMemoryinGB%2Ctags%2C%20mssqlinstalled%2C%20mysqlinstalled%2C%20pgsqlinstalled%0A%7C%20sort%20by%20%28tolower%28tostring%28name%29%29%29%20asc" target="_blank">portal.azure.cn</a>

---
