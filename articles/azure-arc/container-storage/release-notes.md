---
title: Release notes for Azure Container Storage enabled by Azure Arc
description: Learn about new features in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: release-notes
ms.date: 05/13/2025

# Customer intent: "As a cloud administrator, I want to review the latest release notes for Azure Container Storage enabled by Azure Arc, so that I can understand the new features and improvements to manage container data effectively."
ms.custom:
  - build-2025
---

# Release notes for Azure Container Storage enabled by Azure Arc

This article provides information about new features in Azure Container Storage enabled by Azure Arc.

## Version 2.3.1 

- Removal of OSM dependency
- Supportability improvements including Edgevolume and Edgesubvolume status message
- Fix for volume resize causing hanging 

## Version 2.2.4 

- Fixed minor issue when downgrading from 2.2.3 to 2.2.2
- Bug fixes and security updates 

## Version 2.2.3 

- Supportability improvements
- Bug fixes and security updates 

## Version 2.2.1 

- Added Edge Volumes capabilities (Local Shared and Cloud Ingest)
- ACStor can be configured and enabled 

## Version 2.1.0-preview

- CRD operator
- Cloud Ingest Tunable Timers
- Uninstall during version updates
- Added regions: West US, West US 2, North Europe

## Version 1.2.0-preview

- Extension identity and OneLake support: Azure Container Storage enabled by Azure Arc now allows use of a system-assigned extension identity for access to blob storage or OneLake lake houses.
- Security fixes: security maintenance (package/module version updates).

## Version 1.1.0-preview

- Kernel versions: the minimum supported Linux kernel version is 5.1. Currently there are known issues with 6.4 and 6.2.