---
title: Configure DNS for Edge RAG Preview Enabled by Azure Arc
description: "Learn how to configure DNS for your Edge RAG deployment so users and services can access the local portal using the correct domain name."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/20/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to configure DNS for my Edge RAG deployment so that users and services can access the local portal using the correct domain name.
---

# Configure DNS for Edge RAG Preview enabled by Azure Arc

For your Edge RAG deployment, configure DNS by mapping the MetalLB IP address to your local portal domain or updating your hosts file. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Configure DNS for the local portal

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
