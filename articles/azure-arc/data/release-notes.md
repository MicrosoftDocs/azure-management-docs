---
title: Azure Arc-enabled data services - Release notes
description: This article provides highlights for the latest release, and a history of features introduced in previous releases.
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.date: 03/12/2025
ms.topic: conceptual
ms.custom: references_regions, ignite-2023
#Customer intent: As a data professional, I want to understand why my solutions would benefit from running with Azure Arc-enabled data services so that I can leverage the capability of the feature.
---

# Release notes - Azure Arc-enabled data services

This article highlights capabilities, features, and enhancements recently released or improved for Azure Arc-enabled data services.

## April 8, 2025

**Image tag**: `v1.38.0_2025-04-08`

For complete release version information, review [Version log](version-log.md#march-11-2025).

## March 11, 2025

**Image tag**: `v1.37.0_2025-03-11`

For complete release version information, review [Version log](version-log.md#march-11-2025).

## February 28, 2025

**Image tag**: `v1.36.0_2025-02-11`

For complete release version information, review [Version log](version-log.md#february-28-2025).

## February 9, 2025

**Image tag**: `v1.35.0_2024-11-12`

For complete release version information, review [Version log](version-log.md#february-9-2025).

## October 8, 2024

**Image tag**: `v1.34.0_2024-10-08`

For complete release version information, review [Version log](version-log.md#october-8-2024).

## September 9, 2024

**Image tag**: `v1.33.0_2024-09-10`

For complete release version information, review [Version log](version-log.md#september-9-2024). 

## August 13, 2024

**Image tag**: `v1.32.0_2024-08-13`

For complete release version information, review [Version log](version-log.md#august-13-2024). 

## July  9, 2024

**Image tag**: `v1.31.0_2024-07-09`

For complete release version information, review [Version log](version-log.md#july-9-2024). 

## June 11, 2024

**Image tag**: `v1.30.0_2024-06-11` 

For complete release version information, review [Version log](version-log.md#june-11-2024). 

## April 9, 2024

**Image tag**:`v1.29.0_2024-04-09`

For complete release version information, review [Version log](version-log.md#april-9-2024).

## March 12, 2024

**Image tag**:`v1.28.0_2024-03-12`

For complete release version information, review [Version log](version-log.md#march-12-2024).

### SQL Managed Instance enabled by Azure Arc 

Database version for this release (964) has been upgraded beyond the database version for SQL Server 2022 (957). As a result, you can't restore databases from SQL Managed Instance enabled by Azure Arc to SQL Server 2022.

### Streamlined network endpoints

Prior to this release, Azure Arc data processing endpoint was at `san-af-<region>-prod.azurewebsites.net`.

Beginning with this release both Azure Arc data processing, and Azure Arc data telemetry use `*.<region>.arcdataservices.com`. 

## February 13, 2024

**Image tag**:`v1.27.0_2024-02-13`

For complete release version information, review [Version log](version-log.md#february-13-2024).

## December 12, 2023

**Image tag**: `v1.26.0_2023-12-12`

For complete release version information, review [Version log](version-log.md#december-12-2023).

## November 14, 2023

**Image tag**: `v1.25.0_2023-11-14`

For complete release version information, review [Version log](version-log.md#november-14-2023).

## October 10, 2023

**Image tag**: `v1.24.0_2023-10-10`

For complete release version information, review [Version log](version-log.md#october-10-2023).

## September 12, 2023

**Image tag**: `v1.23.0_2023-09-12`

For complete release version information, review [Version log](version-log.md#september-12-2023).

### Release notes

- Portal automatically refreshes status of failover group every 2 seconds. [Monitor failover group status in the portal](managed-instance-disaster-recovery-portal.md#monitor-failover-group-status-in-the-portal).

## August 8, 2023

**Image tag**: `v1.22.0_2023-08-08`

For complete release version information, review [Version log](version-log.md#august-8-2023).

### Release notes

- Support for configuring and managing Azure Failover groups between two instances using Azure portal. For details, review [Configure failover group - portal](managed-instance-disaster-recovery-portal.md).
- Upgraded OpenSearch and OpenSearch Dashboards from 2.7.0 to 2.8.0
- Improvements and examples to [Back up and recover controller database](backup-controller-database.md).

## July 11, 2023

**Image tag**: `v1.21.0_2023-07-11`

For complete release version information, review [Version log](version-log.md#july-11-2023).

### Release notes

- Proxy bypass is now supported for Arc SQL Server Extension. Starting this release, you can also specify services which shouldn't use the specified proxy server.

## June 13, 2023

**Image tag**: `v1.20.0_2023-06-13`

For complete release version information, review [Version log](version-log.md#june-13-2023).

### Release notes

- SQL Managed Instance enabled by Azure Arc
    -  [Added Azure CLI support to manage transparent data encryption (TDE)](configure-transparent-data-encryption-sql-managed-instance.md).

## May 9, 2023

**Image tag**: `v1.19.0_2023-05-09`

For complete release version information, review [Version log](version-log.md#may-9-2023).

New for this release:

### Release notes

- Arc data services
  - OpenSearch replaces Elasticsearch for log database
  - OpenSearch Dashboards replaces Kibana for logs interface
    - There's a known issue with user settings migration to OpenSearch Dashboards for some versions of Elasticsearch, including the version used in Arc data services. 
    
      > [!IMPORTANT]
      > Before upgrade, save any Kibana configuration externally so that it can be re-created in OpenSearch Dashboards.

  - Automatic upgrade is disabled for the Arc data services extension
  - Error-handling in the `az` CLI is improved during data controller upgrade
  - Fixed a bug to preserve the resource limits for Azure Arc Data Controller where the resource limits could get reset during an upgrade.

- SQL Managed Instance enabled by Azure Arc
  - General Purpose: Customer-managed TDE encryption keys (preview). For information, review [Enable transparent data encryption on SQL Managed Instance enabled by Azure Arc](configure-transparent-data-encryption-sql-managed-instance.md).
  - Support for customer-managed keytab rotation. For information, review [Rotate SQL Managed Instance enabled by Azure Arc customer-managed keytab](rotate-customer-managed-keytab.md).
  - Support for `sp_configure` to manage configuration. For information, review [Configure SQL Managed Instance enabled by Azure Arc](configure-managed-instance.md).
  - Service-managed credential rotation. For information, review [How to rotate service-managed credentials in a managed instance](rotate-sql-managed-instance-credentials.md#how-to-rotate-service-managed-credentials-in-a-managed-instance).

## April 12, 2023

**Image tag**: `v1.18.0_2023-04-11`

For complete release version information, see [Version log](version-log.md#april-11-2023).

New for this release:

- SQL Managed Instance enabled by Azure Arc
  - Direct mode for failover groups is generally available az CLI
  - Schedule the HA orchestrator replicas on different nodes when available

- Arc PostgreSQL
  - Ensure postgres extensions work per database/role
  - Arc PostgreSQL | Upload metrics/logs to Azure Monitor

- Bug fixes and optimizations in the following areas:
  - Deploying Arc data controller using the individual create experience has been removed as it sets the auto upgrade parameter incorrectly. Use the all-in-one create experience. This experience creates the extension, custom location, and data controller. It also sets all the parameters correctly. For specific information, see [Create Azure Arc data controller in direct connectivity mode using CLI](create-data-controller-direct-cli.md).

## March 14, 2023

**Image tag**: `v1.17.0_2023-03-14`

For complete release version information, see [Version log](version-log.md#march-14-2023).

New for this release:

- SQL Managed Instance enabled by Azure Arc 
  - [Rotate SQL Managed Instance enabled by Azure Arc service-managed credentials (preview)](rotate-sql-managed-instance-credentials.md) 
- Azure Arc-enabled PostgreSQL 
  - Require client connections to use SSL
  - Extended SQL Managed Instance enabled by Azure Arc authentication control plane to PostgreSQL
