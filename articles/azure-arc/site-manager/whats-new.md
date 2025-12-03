---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: What's new with Site Manager
description: Release notes for Site Manager
author:      AgarkarNikhil # GitHub alias
ms.author:   nagarkar # Microsoft alias
ms.service: azure-arc
ms.topic: whats-new
ms.date: 06/15/2025
ms.subservice: azure-arc-site-manager
---



# What's new with Site Manager

Azure Arc Site Manager is continually evolving to enhance performance and reliability for hybrid management scenarios. This article outlines what’s new, improved, or changed in these updates.

## November 2025

Site Manager Public Preview Refresh v2 leverages Azure Service Groups to enable flexibility of resource selection for a site and multi-level hierarchical representation of an infrastructure organization. The Site scope for Azure Service Group allows organizations to logically group resources across multiple resource groups and subscriptions, such that they reflect real-world operations, making it easier to manage distributed environments.

## June 2025
Site Manager 2.0 public preview refresh with significant performance improvements.

## May 2025
Address migration: Site address was previously stored as a separate resource via [Azure Edge Hardware Center](/azure/azure-edge-hardware-center/azure-edge-hardware-center-overview). With this release site address has been merged into the site resource and is no longer using a separate resource via Azure Edge Hardware Center. Additionally, only the physical address fields have been moved to site properties, and personal contact information is no longer stored. The fields that have been moved are as follows: Address line 1, Address line 2, City, Zip code, State/Province/Region, and Country/Region.

