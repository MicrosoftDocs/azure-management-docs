---
author: JnHs
ms.author: jenhayes
ms.service: azure-arc
ms.topic: include
ms.date: 06/27/2023
# Customer intent: "As a network administrator, I want to ensure that my agents meet the connectivity requirements for cloud onboarding, so that I can successfully establish secure outbound connections using HTTPS and compliant proxy settings."
---

Generally, connectivity requirements include these principles:

- All connections are TCP unless otherwise specified.
- All HTTP connections use HTTPS and SSL/TLS with officially signed and verifiable certificates.
- All connections are outbound unless otherwise specified.

To use a proxy, verify that the agents and the machine performing the onboarding process meet the network requirements in this article.