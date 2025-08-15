---
title: Cloud-native monitoring and alerts with Azure Arc-enabled servers
description: Azure Monitor is the central service for collecting, analyzing, and acting on telemetry from your Azure and hybrid infrastructure, including Arc-enabled servers. 
ms.date: 08/15/2025
ms.topic: concept-article
# Customer intent: "As a cloud administrator managing a hybrid environment, I want to understand how Azure Arc lets me use Azure's powerful monitoring solutions, so I can maintain visibility into my entire server landscape."
---

# Cloud-native monitoring and alerts with Azure Arc-enabled servers

Monitoring in a cloud-native world means moving beyond siloed tools and toward a unified, scalable observability platform. [Azure Monitor](/azure/azure-monitor/fundamentals/overview) is the central service for collecting, analyzing, and acting on telemetry from your Azure and hybrid infrastructure, including Arc-enabled servers.

For system administrators used to on-premises tools like System Center Operations Manager (SCOM), Azure Monitor offers a modern, integrated alternative that spans metrics, logs, alerts, and visualizations, all accessible from the Azure portal, command line, or API.

## Monitor Arc-enabled servers

Azure Monitor provides a tailored experience for virtual machine (VM) monitoring through [VM Insights](/azure/azure-monitor/vm/vminsights-overview), which offers curated dashboards displaying performance metrics and overall health status. VM Insights also work with your on-premises machines connected to Azure Arc, via the Azure Monitor Agent.

Once the agent has been deployed to your Arc-enabled servers, you can create data collection rules (DCRs) to collect metrics and logs from the client operating system. DCRs give you granular control over what data is collected and where it should be sent. Send metric data to Azure Monitor metrics, where it can be analyzed with Metrics Explorer. Send log data from Windows Event logs, Linux syslog, and other custom sources to a Log Analytics workspace where you can analyze it with queries written in Kusto Query Language (KQL). This flexibility enables deep troubleshooting and trend analysis. To share visualizations combining metrics, logs, and text across multiple teams in your organization, you can create custom workbooks.

## Alerts

To be proactively notified of issues detected in your collected data, create alert rules. Azure Monitor supports different types of alerts to make sure you have the details you need about your hybrid environment.

Metrics alerts are near-real-time alerts that compare collected values to either static or dynamic thresholds, using machine learning to determine optimal performance ranges.

Log query alerts help identify issues in the log data stored in the Log Analytics workspace. This may be a simple detection of an error event or the results of a complex log query analyzing multiple sets of data across multiple servers.

## Monitor applications running on Arc-enabled servers

Use Application Insights in Azure Monitor to deliver end-to-end tracing and telemetry for web applications and APIs, capturing request/response data and performance metrics. Track requests across services and components, helping identify performance bottlenecks and latency issues using Distributed Tracing.

The Smart Detection feature uses machine learning to automatically detect anomalies in application behavior or usage patterns. This can give you  proactive insights into potential problems before they impact your users.

## Monitor networks

Tools in Network Watcher help you monitor network connectivity, latency, and packet loss across Azure and your hybrid environment. Connection Monitor tracks end-to-end network paths and generates alerts when degradation is detected, helping ensure reliable connectivity. Additionally, Traffic Analytics analyzes NSG flow logs to visualize traffic patterns and identify anomalies in network behavior, supporting both performance optimization and security monitoring.
