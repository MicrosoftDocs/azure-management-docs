---
ms.service: azure-arc
ms.subservice: azure-arc-container-storage
ms.topic: include
ms.date: 09/26/2025
author: asergaz
ms.author: sergaz
---

1. Create a storage account following the instructions in [Create an Azure storage account](/azure/storage/common/storage-account-create).

   > [!NOTE]
   > When you create your storage account, it's recommended that you create it under the same resource group and region/location as your Kubernetes cluster.

1. Create a container in the storage account that you created previously, following the instructions in [Quickstart: Upload, download, and list blobs with the Azure portal > Create a container](/azure/storage/blobs/storage-quickstart-blobs-portal#create-a-container).