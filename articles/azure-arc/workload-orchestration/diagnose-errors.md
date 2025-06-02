---
title: Diagnose Workload Orchestration Errors
description: Learn how to diagnose configuration, staging, deployment, and cloud errors across the workload orchestration lifecycle.
author: SoniaLopezBravo
ms.author: sonialopez
ms.topic: how-to
ms.date: 06/02/2025
---

# Diagnose workload orchestration errors

This article describes how to diagnose configuration, staging, deployment, and cloud errors across the workload orchestration lifecycle. Errors are surfaced contextually in the portal for all users for quick resolution.

## Validation and templating errors

Some common validation and templating errors include:

- Schema syntax errors, for example, missing or extra commas, incorrect brackets, non-YAML format, etc.
- Unsupported data types or invalid values, for example, using a string where an integer is expected.
- Missing required properties, for example, not specifying a required field in the configuration.
- Incorrect hierarchy levels, for example, placing a property at the wrong level in the configuration hierarchy.

These type of errors are visible in the [workload orchestration portal](configure.md) during the configuration phase. They can be resolved by correcting the syntax, ensuring all required fields are present, and validating the data types used.


## Staging and deployment failures

Some common staging failures include:

- Insufficient disk space on the connected registry. 
- Network latency or connectivity issues.
- Authentication errors with Azure Container Registry (ACR) or edge cluster. 
- Image download failures. 

Some common deployment failures include:

- Helm chart timeouts or failures.
- Application crash. 
- Missing or corrupted image/config files.
- Partial failures across lines with different root during bulk deployments.

Staging errors are visible in the "Published solutions" tab of the [workload orchestration portal](configure.md#view-the-published-solutions). Deployment errors are visible both in the [workload orchestration portal](deploy.md) and [Azure portal](azure-portal-monitoring.md). They can be resolved by checking the logs for specific error messages, ensuring sufficient resources are available, and verifying network connectivity.

## Cloud errors for IT users

Some common cloud errors include misconfigured RBAC permissions or policy restrictions, such as:ç

- Azure Monitor ingestion failures due to misconfigured diagnostic settings.
- Authentication failures or expired credentials.
- Regional latency or service outages.

These errors are visible in the Azure portal through the Azure Monitor service. They can be resolved by checking the diagnostic settings, ensuring proper RBAC permissions are in place, and verifying the health of Azure services in the region.