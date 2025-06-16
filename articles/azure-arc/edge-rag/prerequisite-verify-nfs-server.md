---
title: Prerequisite: Verify NFS server is configured and reachable
description: "Ensure NFS server is set up for Edge RAG deployment."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/06/2025
ai-usage: ai-assisted
---

# Prerequisite: Verify NFS server is configured and reachable

In this article, verify your NFS server is configured and reachable for your Edge RAG deployment by checking connectivity and ensuring your data is accessible. This article is part of the deployment prerequisites checklist.

## Verify NFS server configuration and connectivity

Make sure all your documents and images are available on a supported network file system (NFS) server. That NFS server must be configured correctly and reachable from the Edge RAG deployment on Azure Local. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources) 
- [Configure the network file system server for Edge RAG](configure-nfs-server.md).

Use the NFS server you configured to host the documents and images you want to use for testing with Edge RAG.

## Next step

> [!div class="nextstepaction"]
> [Prerequisite: Prepare AKS cluster on Azure Local](prerequisites-prepare-aks-cluster.md)
