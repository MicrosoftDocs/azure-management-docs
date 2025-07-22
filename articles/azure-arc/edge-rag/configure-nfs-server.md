---
title: Configure NFS Server for Edge RAG 
description: "Learn how to configure an NFS server for Edge RAG to enable shared storage for high availability and scalable applications."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to #Don't change
ms.date: 05/13/2025
ms.subservice: edge-rag
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to configure an NFS server for Edge RAG, so that I can implement shared storage that enhances the high availability and scalability of my applications.
---

# Configure an NFS server for Edge RAG Preview, enabled by Azure Arc 

Install network file system (NFS) server packages and create the shared directory to use as a data source for Edge RAG.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Configure the NFS server

Complete the following steps to set up your NFS server to use with Edge RAG.

1. Install NFS server packages. On the server to host the NFS shares, install the necessary NFS packages:

   ```bash
   sudo apt update
   sudo apt install nfs-kernel-server
   ```

1. Create the shared directory.

    ```bash
     # Create the directory.
     
     sudo mkdir -p /srv/edgerag/share 
         
     # Create the group with a GID of your choice if it doesn't exist. The following example uses GID 1003.
     
     sudo groupadd -g 1003 edgerag_group 

      # Create the user with uid 1003 and assign to the group if it doesn't exist. The following example uses UID 1003.

     sudo useradd -u 1003 -g 1003 edgerag_user 
     
     # Set ownership to uid 1003 and gid 1003 **

     sudo chown 1003:1003 /srv/edgerag/share 

     # Set appropriate permissions **

     sudo chmod 755 /srv/edgerag/share

     # Edit the /etc/exports file to specify the directories to share and their permissions.

     sudo nano /etc/exports 

     # Add the following line at the end of the file where you change "*" to the client IP to restrict random access to the share folder.

    /srv/edgerag/share *(rw,sync,no_subtree_check)

     # Run the following command reflect the change:

     sudo exportfs -a
    ```

1. Start and Enable the NFS Server Service**

   ```bash
   sudo systemctl restart nfs-kernel-server 
   sudo systemctl enable nfs-kernel-server 
   ```

1. Optional - Set a password for the user to allow password-based SSH authentication. This step is needed if you want to copy files to the share by using the Secure Copy Protocol (SCP) command.

   ```bash
   sudo passwd edgerag_user
   ```

## Related content

- [Supported data sources](requirements.md#supported-data-sources)
- [Complete Edge RAG deployment prerequisites](complete-prerequisites.md)
