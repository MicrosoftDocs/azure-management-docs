---
title: include file
description: include file
services: lighthouse
author: JnHs
ms.service: azure-lighthouse
ms.topic: include
ms.date: 07/10/2024
ms.author: jenhayes
# Customer intent: As a service provider, I want to use samples to deploy Azure Policy to delegated subscriptions, so that I can ensure compliance and effective governance across the customer resources that I manage.
---

These samples show how to use Azure Policy with subscriptions that are onboarded to Azure Lighthouse.

| **Template** | **Description** |
|---------|---------|
| [policy-add-or-replace-tag](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-add-or-replace-tag) | Assigns a policy that adds or removes a tag (using the modify effect) to a delegated subscription. For more info, see [Deploy a policy that can be remediated within a delegated subscription](../how-to/deploy-policy-remediation.md). |
| [policy-allow-certain-managing-tenants](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-allow-certain-managing-tenants) | Assigns a policy restricting Azure Lighthouse delegations to specific managing tenants. |
| [policy-audit-delegation](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-audit-delegation) | Assigns a policy that audits for delegation assignments. |
| [policy-delegate-management-groups](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-delegate-management-groups) | Assigns a policy to confirm that [subscriptions within a management group are delegated to a managing tenant](../how-to/onboard-management-group.md), and if not, creates the assignment. |
| [policy-enforce-keyvault-monitoring](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-enforce-keyvault-monitoring) | Assigns a policy that enables diagnostics on Azure Key Vault resources in a delegated subscription (using the deployIfNotExists effect). For more info, see [Deploy a policy that can be remediated within a delegated subscription](../how-to/deploy-policy-remediation.md). |
| [policy-enforce-sub-monitoring](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-enforce-sub-monitoring) | Assigns several policies to enable diagnostics on a delegated subscription, and connects all Windows & Linux VMs to the Log Analytics workspace created by the policy. For more info, see [Deploy a policy that can be remediated within a delegated subscription](../how-to/deploy-policy-remediation.md). |
| [policy-initiative](https://github.com/Azure/Azure-Lighthouse-samples/tree/master/templates/policy-initiative) | Applies a [policy initiative](/azure/governance/policy/overview#initiative-definition) (multiple related policy definitions) to a delegated subscription. |

