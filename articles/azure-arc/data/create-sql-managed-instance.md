---
title: Create a SQL Managed Instance enabled by Azure Arc
description: Deploy SQL Managed Instance enabled by Azure Arc
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-sql-mi
ms.custom: devx-track-azurecli
author: anatripathi-git
ms.author: anatripathi
ms.reviewer: mikeray
ms.date: 07/30/2021
ms.topic: how-to
# Customer intent: As a database administrator, I want to deploy a SQL Managed Instance using Azure Arc, so that I can manage my SQL databases in a consistent manner across hybrid cloud environments.
---

# Deploy a SQL Managed Instance enabled by Azure Arc

[!INCLUDE [azure-arc-common-prerequisites](./includes/azure-arc-common-prerequisites.md)]

To view available options for the create command for SQL Managed Instance enabled by Azure Arc, use the following command:

```azurecli
az sql mi-arc create --help
```

To create a SQL Managed Instance enabled by Azure Arc, use `az sql mi-arc create`.

> [!NOTE]
>  A ReadWriteMany (RWX) capable storage class needs to be specified for backups. Learn more about [access modes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#access-modes)

If no storage class is specified for backups, the default storage class in Kubernetes is used. If the default storage class isn't RWX capable, the installation may not succeed.

```azurecli
az sql mi-arc create --name <name> --resource-group <group> -–subscription <subscription>  --custom-location <custom-location> --storage-class-backups <RWX capable storageclass>
```

Example:

```azurecli
az sql mi-arc create --name sqldemo --resource-group rg -–subscription xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx  --custom-location private-location --storage-class-backups mybackups
```

> [!NOTE]
>
> Names must be fewer than 60 characters in length and conform to [DNS naming conventions](https://kubernetes.io/docs/concepts/overview/working-with-objects/names/#rfc-1035-label-names).
>
> When specifying memory allocation and vCore allocation, use this formula to ensure your performance is acceptable: For each 1 vCore, plan at least 4GB of RAM of capacity available on the Kubernetes node.
>
> If you want to automate the creation of SQL Managed Instance enabled by Azure Arc and avoid the interactive prompt for the admin password, set the `AZDATA_USERNAME` and `AZDATA_PASSWORD` environment variables to the desired username and password before you run the `az sql mi-arc create` command.
>
> If you created the data controller using AZDATA_USERNAME and AZDATA_PASSWORD in the same terminal session, then the values for AZDATA_USERNAME and AZDATA_PASSWORD will be used to create the SQL Managed Instance enabled by Azure Arc too.
> 


## View instance on Azure Arc

To view the instance, use the following command:

```azurecli
az sql mi-arc list --k8s-namespace <namespace> --use-k8s
```

Copy the external IP and port number from the result. Use the external IP address to connect to the instance.

## Related content
- [Connect to SQL Managed Instance enabled by Azure Arc](connect-managed-instance.md)
- [Register your instance with Azure and upload metrics and logs about your instance](upload-metrics-and-logs-to-azure-monitor.md)

