---
title: Configure an NFS server for Agents and Tools with Foundry Local
description: "Learn how to configure an NFS server for Agents and Tools with Foundry Local to enable shared storage for high availability and scalable applications."
ms.reviewer: assafakiva
ms.topic: how-to #Don't change
ms.date: 11/05/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - build-2025
# Customer intent: As a system administrator, I want to configure an NFS server for Agents and Tools with Foundry Local, so that I can implement shared storage that enhances the high availability and scalability of my applications.
---

# Configure an NFS server for Agents and Tools with Foundry Local

Install network file system (NFS) server packages and create the shared directory to use as a data source for Agents and Tools with Foundry Local.

## Configure the NFS server

Complete the following steps to set up your NFS server to use with Agents and Tools with Foundry Local.

This procedure configures `AUTH_SYS` access, where NFS authorizes requests by numeric user ID (UID) and group ID (GID). Use the same UID and GID when you create the ingestion data source, and give that identity read and directory-traverse access to the export. To use Kerberos instead, follow [Set up NFS with Kerberos authentication](connect-file-share-kerberos-setup.md).

1. Install NFS server packages. On the server to host the NFS shares, install the necessary NFS packages.

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

     # Create the user with UID 1003 and assign it to the group if it doesn't exist.

     sudo useradd -u 1003 -g 1003 edgerag_user 

     # Set ownership to uid 1003 and gid 1003

     sudo chown 1003:1003 /srv/edgerag/share 

     # Set appropriate permissions

     sudo chmod 755 /srv/edgerag/share

     # Edit the /etc/exports file to specify the directories to share and their permissions.

     sudo nano /etc/exports 

     # Add the following line at the end of the file where you change "*" to the client IP to restrict random access to the share folder.

    /srv/edgerag/share *(rw,sync,no_subtree_check)

     # Run the following command to reflect the change:

     sudo exportfs -a
    ```

1. Start and enable the NFS server service.

   ```bash
   sudo systemctl restart nfs-kernel-server 
   sudo systemctl enable nfs-kernel-server 
   ```

1. (Optional) Set a password for the user to allow password-based SSH authentication. This step is needed if you want to copy files to the share by using the Secure Copy Protocol (SCP) command.

   ```bash
   sudo passwd edgerag_user
   ```

## Related content

- [Supported data sources](requirements.md#supported-data-sources)
- [Complete Agents and Tools with Foundry Local deployment prerequisites](complete-prerequisites.md)
