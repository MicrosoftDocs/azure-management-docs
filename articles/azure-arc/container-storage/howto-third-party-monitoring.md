---
title: Third-party monitoring with Prometheus and Grafana
description: Learn how to monitor your Azure Container Storage enabled by Azure Arc deployment using third-party monitoring with Prometheus and Grafana.
author: asergaz
ms.author: sergaz
ms.topic: how-to
ms.date: 07/21/2025

# Customer intent: "As a system administrator, I want to configure Prometheus and Grafana to monitor Azure Container Storage enabled by Azure Arc, so that I can effectively visualize and manage my containerized applications' performance and metrics."
---

# Third-party monitoring with Prometheus and Grafana

This article describes how to monitor your deployment using third-party monitoring with Prometheus and Grafana.

## Metrics

### Configure an existing Prometheus instance for use with Azure Container Storage enabled by Azure Arc

This guidance assumes that you previously worked with or configured Prometheus for Kubernetes. For more information about how to enable Prometheus and Grafana, see the [Enable Prometheus and Grafana](/azure/azure-monitor/containers/kubernetes-monitoring-enable#enable-prometheus-and-grafana) section of [Enable monitoring for Kubernetes clusters](/azure/azure-monitor/containers/kubernetes-monitoring-enable).

For more information about the required Prometheus scrape configuration, see the [Metrics configuration](howto-azure-monitor-kubernetes.md#metrics-configuration) section of [Azure Monitor and Kubernetes monitoring](howto-azure-monitor-kubernetes.md). Once you configure Prometheus metrics, you can then deploy [Grafana](/azure/azure-monitor/visualize/grafana-plugin) to monitor and visualize your Azure services and applications.

## Logs

The logs for Azure Container Storage enabled by Azure Arc are accessible through the Azure Kubernetes Service [kubelet logs](/azure/aks/kubelet-logs). You can also collect this log data using the [syslog collection feature in Azure Monitor Container Insights](/azure/azure-monitor/containers/container-insights-syslog).

## Next steps

[What is Azure Container Storage enabled by Azure Arc?](overview.md)
