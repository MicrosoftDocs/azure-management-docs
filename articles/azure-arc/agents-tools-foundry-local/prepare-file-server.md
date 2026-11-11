---
title: Verify File Share Access for Agents and Tools with Foundry Local
description: "Learn about preparing your file share configuration and connectivity for Agents and Tools with Foundry Local deployment to make sure your data is accessible and ready."
ms.reviewer: assafakiva
ms.topic: concept-article
ms.date: 11/04/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to verify that my file share is configured and reachable for Agents and Tools with Foundry Local deployment so that my documents and images are accessible to the chat solution.
---

# Verify file share access for Agents and Tools with Foundry Local

If you plan to ingest files from a network file system (NFS) share, verify that the share is configured and reachable for Agents and Tools with Foundry Local. This article is part of the deployment prerequisites checklist. You don't need an NFS server if you use only other data sources.

## Verify file share configuration and connectivity

Ensure the documents and images that you want to ingest from NFS are available on a supported NFS server. Configure the server and make it reachable from the cluster. Before it accepts an ingestion request, the API checks that it can resolve the server name and connect to TCP port 2049. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources)
- [Configure the network file system server for Agents and Tools with Foundry Local](configure-nfs-server.md)

Use the file share you configured to store the documents and images you plan to test with Agents and Tools with Foundry Local.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster on Azure Local](prepare-aks-cluster.md)
