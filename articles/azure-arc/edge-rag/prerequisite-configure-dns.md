---
title: Prerequisite: Configure DNS
description: "Configure DNS for Edge RAG deployment."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/06/2025
ai-usage: ai-assisted
---

# Prerequisite: Configure DNS

In this article, configure DNS for your Edge RAG deployment by mapping the MetalLB IP address to your local portal domain or updating your hosts file. This article is part of the deployment prerequisites checklist.

## Configure DNS for Edge RAG

Make sure you configure your DNS to map the IP address assigned to MetalLB to the domain name of the local portal like `arcrag.contoso.com`.

Alternately, update your local machine (laptop or DVM) to reach the local portal domain by using the following steps:

1. Open a text editor like Notepad with **Administrator** privileges.

2. Then open file "**C:\Windows\System32\drivers\etc\hosts**"

3. At the end of the file, add a line: "100.XX.XX.XX arcrag.contoso.com" where you replace the IP address and url for your local portal.

4. Save the file

5. In the browser, go to your local portal like `https://arcrag.contoso.com`.

## Next step

> [!div class="nextstepaction"]
> [Deploy the Edge RAG extension](deploy.md)
