---
title: Verify File Share Access for Agentic Retrieval in Foundry Local
description: "Learn about preparing your file share configuration and connectivity for Agentic Retrieval in Foundry Local deployment to make sure your data is accessible and ready."
author: cwatson-cat
ms.author: cwatson
ms.topic: concept-article
ms.date: 09/22/2025
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to verify that my file share is configured and reachable for Agentic Retrieval in Foundry Local deployment so that my documents and images are accessible to the chat solution.
---

# Verify file share access for Agentic Retrieval in Foundry Local

As part of preparing for your deployment, verify your network file system (NFS) server share is configured and reachable for Agentic Retrieval. Check the connectivity and make sure your data is accessible. This article is part of the deployment prerequisites checklist.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Verify file share configuration and connectivity

Make sure all your documents and images are available on a supported NFS server. That server must be configured correctly and reachable from the Agentic Retrieval deployment on Azure Local. For more information, see the following articles:

- [Supported data sources](requirements.md#supported-data-sources)
- [Configure the network file system server for Agentic Retrieval](configure-nfs-server.md)

Use the file share you configured to store the documents and images you plan to test with Agentic Retrieval.

## Next step

> [!div class="nextstepaction"]
> [Prepare AKS cluster on Azure Local](prepare-aks-cluster.md)