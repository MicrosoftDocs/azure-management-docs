---
title: Configure DNS for Agents and Tools with Foundry Local
description: "Learn how to configure DNS for your Agents and Tools with Foundry Local deployment so users and services can access the local portal using the correct domain name."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 10/29/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to configure DNS for my Agents and Tools with Foundry Local deployment so that users and services can access the local portal using the correct domain name.
---

# Configure DNS for Agents and Tools with Foundry Local

For your Agents and Tools with Foundry Local deployment, configure DNS by mapping the MetalLB IP address to your local portal domain or updating your hosts file. This article is part of the deployment prerequisites checklist.

## Configure DNS for the local portal

Ensure that you configure your DNS to map the IP address assigned to MetalLB to the domain name of the local portal, such as `arcrag.contoso.com`.

Alternatively, update your local machine (laptop or DVM) to reach the local portal domain by using the following steps:

1. Open a text editor such as Notepad with **Administrator** privileges.

1. Open the file **C:\Windows\System32\drivers\etc\hosts**.

1. At the end of the file, add a line such as `100.XX.XX.XX arcrag.contoso.com`, replacing the IP address and URL with those for your local portal.

1. Save the file.

1. In the browser, go to your local portal, such as `https://arcrag.contoso.com`.

## Next step

> [!div class="nextstepaction"]
> [Deploy the Agents and Tools with Foundry Local extension](deploy.md)
