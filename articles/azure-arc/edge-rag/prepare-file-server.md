---
title: Verify NFS Server Access for Edge RAG Preview Enabled by Azure Arc
description: "Learn about preparing your NFS server configuration and connectivity for Edge RAG deployment to make sure your data is accessible and ready."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 06/20/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to verify that my NFS server is configured and reachable for Edge RAG deployment so that my documents and images are accessible to the chat solution.
---

# Verify NFS server access for Edge RAG Preview enabled by Azure Arc

As part of preparing for your deployment, verify your network file share (NFS) server is configured and reachable for Edge RAG. Check the connectivity and make sure your data is accessible. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Verify NFS server configuration and connectivity

Make sure all your documents and images are available on a supported NFS server. That NFS server must be configured correctly and reachable from the Edge RAG deployment on Azure Local. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources) 
- [Configure the network file system server for Edge RAG](configure-nfs-server.md).

Use the NFS server you configured to store the documents and images you plan to test with Edge RAG.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster on Azure Local](prepare-aks-cluster.md)