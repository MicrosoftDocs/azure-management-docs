---
title: Edge Volumes storage options
description: Learn about different data storage options in Azure Container Storage enabled by Azure Arc.
author: asergaz
ms.author: sergaz
ms.topic: concept-article
ms.date: 05/05/2025
# Customer intent: As a cloud architect, I want to understand the different storage options available in Azure Container Storage enabled by Azure Arc, so that I can design flexible data flow architectures for my applications at the edge and in the cloud.
---

# Storage options using Azure Container Storage enabled by Azure Arc

Azure Container Storage offers many different storage options for customer data. This article describes the available options.

## Local Shared Edge Volume

The first and most basic option is to use a Local Shared Edge Volume. This volume type offers read-write-many storage, local to your Kubernetes cluster. If installed on a 3 or more node cluster, the option to enable replication of data for failover and resiliency is offered. This volume type is appropriate for scratch space, temporary storage, and locally persistent data unsuitable for cloud destinations. It's also a good destination for data that's actively being worked on, changed, or processed at the edge.

## Cloud Edge Volume

The second storage option is a Cloud Edge Volume, with an ingest synchronization policy. This volume/subvolume combination, [as described here](howto-configure-cloud-ingest-subvolumes.md), gives you access to greater storage capacity by connecting your Kubernetes cluster to all that Azure has to offer. With cloud destinations such as Blob, ADLS generation 2, and OneLake, you can increase your data storage capacity by sending your data to Azure. You input your desired storage account and container information (see [Map file to object path names](howto-map-names.md)), and our application takes care of moving your data to Azure and cleaning out the local copy as desired, and specified in your synchronization policy, to allow for virtually limitless data ingestion.

## Use both storage options

You can also use these two storage options in tandem. For example, see the following scenario:

:::image type="content" source="media/storage-options/data-storage-options.png" alt-text="Diagram showing storage options architecture." lightbox="media/storage-options/data-storage-options.png":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

In this case, we have an application that generates data. We want to send some of that data directly to the cloud, so we put it in an Ingest subvolume configured for our cloud storage destination; in this case, OneLake. But we aren't ready to send some of our other data to the cloud yet, so instead we send that data to the Local Shared volume. Here, our app can go back and forth, interacting with and changing or processing that data as needed. Once we are satisfied with the processed data and are ready to find its home in our cloud destination, we can copy the data from the Local Shared volume to our Ingest subvolume, and the destination file is automatically uploaded to the Azure cloud.

This is just one example of a potential data flow. If you have comments or questions, or want to talk through the best way to set up your data flow, you can [contact our team](mailto:ACSA@microsoft.com).

## Flexibility

You can add a layer of flexibility by having subvolumes underneath the Cloud Edge Volumes. If you're already running an application and mounted the Cloud Edge Volume, but want a new cloud storage destination, you don't need to redeploy your application. You can create a new subvolume CRD to match the new destination, and then point the application at the new subvolume path.

For example, you can use this capability to differentiate between development, test, and production environments. In this case, you can create an Edge Volume path, such as **/myapp**, and then have a subvolume for development data, such as **/myapp/dev**. Then, when the application is ready for production, you can create a new subvolume path, such as **/myapp/prod**, and point the app at the new path name to which to write the production data.

## Next steps

[Overview of volumes and subvolumes](volumes-subvolumes.md)
