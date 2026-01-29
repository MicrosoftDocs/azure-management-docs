---
title: include file
description: include file
services: lighthouse
author: JnHs
ms.service: azure-lighthouse
ms.topic: include
ms.date: 01/20/2026
ms.author: jenhayes
# Customer intent: "As a service provider, I want to set up alerts for customer resources that I manage through Azure Lighthouse, so that I can stay informed about changes and activities across subscriptions."
---

These samples show how to use Azure Monitor to create alerts for subscriptions that are onboarded to Azure Lighthouse.

| **Template** | **Description** |
|---------|---------|
| [monitor-delegation-changes](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/tools/monitor-delegation-changes) | Queries the past day of activity in a managing tenant and [reports on any added or removed delegations](../how-to/monitor-delegation-changes.md) and any attempts that weren't successful.|
| [alert-using-actiongroup](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/alert-using-actiongroup) | Creates an Azure alert and connects to an existing action group.|
| [multiple-loganalytics-alerts](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/multiple-loganalytics-alerts) | Creates multiple log alerts based on Kusto queries.|
| [delegation-alert-for-customer](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/delegation-alert-for-customer) | Deploys an alert in a tenant when a user delegates a subscription to a managing tenant.|
| [workbook-activitylogs-by-domain](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/workbook-activitylogs-by-domain) | Displays Azure Activity logs across subscriptions with an option to filter them by domain name. |
