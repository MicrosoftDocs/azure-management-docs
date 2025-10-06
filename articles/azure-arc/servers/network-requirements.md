---
title: Connected Machine Agent Network Requirements
description: Learn about the networking requirements for using the Azure Connected Machine agent for Azure Arc-enabled servers.
ms.date: 09/29/2025
ms.topic: concept-article 
# Customer intent: "As a system administrator, I want to understand the networking requirements for deploying the Connected Machine agent so that I can successfully onboard physical servers or virtual machines to Azure Arc-enabled servers."
---

# Connected Machine agent network requirements

This article describes the networking requirements for using the Azure Connected Machine agent to onboard a physical server or virtual machine to Azure Arc-enabled servers.

> [!TIP]
> For the Azure public cloud platform, you can reduce the number of required endpoints by using the [Azure Arc gateway](arc-gateway.md).

## Details

[!INCLUDE [network-requirement-principles](../includes/network-requirement-principles.md)]

[!INCLUDE [network-requirements](./includes/network-requirements.md)]

## Subset of endpoints for ESU only

[!INCLUDE [esu-network-requirements](./includes/esu-network-requirements.md)]

## Related content

* For information on more prerequisites for deploying the Connected Machine agent, see [Connected Machine agent prerequisites](prerequisites.md).
* Before you deploy the Connected Machine agent and integrate with other Azure management and monitoring services, review the [planning and deployment guide](plan-at-scale-deployment.md).
* To resolve problems, review the [agent connection issues troubleshooting guide](troubleshoot-agent-onboard.md).
* For a complete list of network requirements for Azure Arc features and Azure Arc-enabled services, see [Azure Arc network requirements (Consolidated)](../network-requirements-consolidated.md).
