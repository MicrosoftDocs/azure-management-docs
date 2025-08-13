---
title: Cloud-native licensing and cost management
description: Cloud-native licensing options for Azure Arc help reduce overhead of license management and ensure your servers have appropriate, up-to-date coverage.
ms.date: 08/12/2025
ms.topic: concept-article
# Customer intent: "As an administrator managing a hybrid cloud environment, I want to understand licensing and cost management options for Azure Arc, so I can manage my hybrid server licenses and have visibility into costs."
---

# Cloud-native licensing and cost management

In a traditional setup, licensing for servers can be complex, especially in hybrid environments. Cloud-native licensing models for your Azure Arc-enabled servers provide flexibility and reduce overhead, so you can avoid buying perpetual licenses and doing annual true-ups for compliance. This shift lets you handle your Windows and SQL Server licensing like a cloud service: you only pay for what you use, when you use it, and you can manage it all centrally through Azure. For a system administrator, this means less time tracking license keys or entering activation codes, and more confidence that your servers are properly licensed and receiving critical updates.

## Windows Server Pay-as-you-go

Historically, if you ran Windows Server on-premises, you activated it with a product key and that was that. With Azure Arc, Microsoft introduced an alternative: [Windows Server Pay-as-you-go licensing](/windows-server/get-started/windows-server-pay-as-you-go) via Azure. 

Essentially, this model turns Windows Server into a subscription. Rather than worrying about buying new licenses for new servers, just connect them to Arc and enable Pay-as-you-go. This is great for environments that fluctuate, or for avoiding large upfront costs. It also enables compliance, since Azure handles the metering.

With this licensing model, you connect your server to Azure Arc and [enable Pay-as-you-go](/windows-server/get-started/windows-server-pay-as-you-go#set-up-windows-server-pay-as-you-go) in the Azure portal (or alternately, enable it when you install the OS and then connect to Azure Arc). The server then uses Azure for activation and billing, without requiring a product key. You'll see charges on your Azure bill per core, per hour for that server's OS license, similar to how Azure VM pricing includes Windows licensing. If the server is shut down, you can stop billing by disabling the feature for that machine.

Pay-as-you-go is currently available for Windows Server 2025 and beyond, with the same pricing for Standard or Datacenter editions. Importantly, no client access licenses (CALs) are required for base functionality.

## Extended Security Updates (ESUs)

One of the pain points with older servers is figuring out what to do when they reach end of support. For example, Windows Server 2012 and Windows Server 2012 R2 reached end of support on Oct 10, 2023. To ensure these older servers keep getting critical security patches, you'd normally have to purchase [Extended Security Update (ESU)](/windows-server/get-started/extended-security-updates-overview) licenses separately for each server. 

By [onboarding Windows Server 2012 servers to Azure Arc and signing up for ESUs in the Azure portal](/azure/azure-arc/servers/prepare-extended-security-updates), you get up to three more years of security patches. There's no need to install separate ESU product keys on each server; the Connected Machine agent enables the updates on each enrolled machine. Those servers then become eligible to receive ESU patches via your normal update tools (including Azure Update Manager). It’s keyless and much simpler operationally.

Rather than requiring a lump sum purchase for the full period, ESUs enabled by Azure Arc are pay-as-you-go on a monthly basis. Charges are billed through Azure, so you can use existing Azure credits or commitments and use [Microsoft Cost Management and Billing](/azure/cost-management-billing/cost-management-billing-overview) to analyze your costs. The Azure portal also provides a central inventory of which servers are ESU-covered, letting you easily determine which servers aren't yet covered by ESUs. 

## SQL Server Pay-as-you-go

## Centralized purchasing and FinOps

## Licensing and patching together