---
title: Frequently asked questions
description: "Frequently asked questions to understand and troubleshoot Azure Arc sites and site manager"
author: torreymicrosoft
ms.author: torreyt
ms.service: azure-arc
ms.subservice: azure-arc-site-manager
ms.topic: faq #Don't change
ms.date: 02/16/2024
ms.custom: references_regions

#customer intent: As a customer, I want answers to questions so that I can answer my own questions.

---

# Frequently asked questions: Azure Arc site manager (preview)

The following are frequently asked questions and answers for Azure Arc site manager.

__Question__: What is changing with site address?

__Answer__: Site address is currently stored as a separate resource via [Azure Edge Hardware Center](/azure/azure-edge-hardware-center/azure-edge-hardware-center-overview). As of May, site address has been merged into the site resource and is no longer using a separate resource via Azure Edge Hardware Center. Additionally, only the physical address fields have been moved to site properties, and personal contact information is no longer stored. The fields that have been moved are as follows: Address line 1, Address line 2, City, Zip code, State/Province/Region, and Country/Region.

**Question:** I have resources in the resource group, which aren't yet supported by site manager. Do I need to move them?

**Answer:** Site manager provides status aggregation for only the supported resource types. Resources of other types won't be managed via site manager. They continue to function normally as they would without otherwise.

**Question:** Does site manager have a subscription or fee for usage?

**Answer:** Site manager is free. However, the Azure services that integrate with sites and site manager might have a fee. Few examples: alerts used with site manager via monitor might have a fees, or enabling defender for monitoring site security might have a fees.

**Question:** What regions are currently supported via site manager? What regions of these supported regions aren't fully supported?

**Answer:** Site manager supports resources that exist in [supported regions](https://azure.microsoft.com/explore/global-infrastructure/products-by-region/?products=azure-arc&regions=all), with a few exceptions. For the following regions, connectivity and update status aren't supported for Arc-enabled machines or Arc-enabled Kubernetes clusters:

* Brazil South
* UAE North
* South Africa North
