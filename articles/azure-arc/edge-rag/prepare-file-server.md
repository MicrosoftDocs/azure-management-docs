---
title: Verify NFS Server Access for Edge RAG Deployment
description: "Learn about preparing your NFS server configuration and connectivity for Edge RAG deployment to make sure your data is accessible and ready."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 06/21/2025
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to verify that my NFS server is configured and reachable for Edge RAG deployment so that my documents and images are accessible to the chat solution.
---

# Verify NFS server access for Edge RAG deployment

As part of preparing for your Edge RAG deployment, verify your NFS server is configured and reachable for your Edge RAG deployment. Check the connectivity and make sure your data is accessible. This article is part of the deployment prerequisites checklist.

## Verify NFS server configuration and connectivity

Make sure all your documents and images are available on a supported network file system (NFS) server. That NFS server must be configured correctly and reachable from the Edge RAG deployment on Azure Local. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources) 
- [Configure the network file system server for Edge RAG](configure-nfs-server.md).

Use the NFS server you configured to host the documents and images you want to use for testing with Edge RAG.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster on Azure Local for Edge RAG deployment](prepare-aks-cluster.md)