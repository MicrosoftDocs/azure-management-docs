---
title: Restore the AdventureWorks Sample Database into SQL Managed Instance
description: Restore the AdventureWorks sample database into SQL Managed Instance
author: twright-msft
ms.author: twright
ms.reviewer: mikeray, randolphwest
ms.date: 03/16/2026
ms.service: azure-arc
ms.subservice: azure-arc-sql-mi
ms.topic: how-to
# Customer intent: As a database administrator, I want to restore the AdventureWorks sample database into my SQL managed instance, so that I can use it for testing and learning purposes with my cloud-based SQL environment.
---

# Restore the AdventureWorks sample database into SQL Managed Instance - Azure Arc

[AdventureWorks](/sql/samples/adventureworks-install-configure) is a sample database containing an OLTP database that you often use in tutorials and examples. Microsoft provides and maintains it as part of the [SQL Server samples GitHub repository](https://github.com/microsoft/sql-server-samples/tree/master/samples/databases).

This article describes a basic process to get the `AdventureWorks` sample database restored into your SQL managed instance - Azure Arc.

## Download the AdventureWorks backup file

Download the `AdventureWorks` backup (`.bak`) file into your SQL managed instance container. In this example, use the `kubectl exec` command to remotely run a command inside the SQL managed instance container to download the `.bak` file into the container. Download this file from any location accessible by `wget` if you have other database backup files you want to pull to be inside of the SQL managed instance container. Once it's inside the SQL managed instance container, you can restore it using standard [RESTORE DATABASE](/sql/t-sql/statements/restore-statements-transact-sql) Transact-SQL command.

Run a command similar to the following example to download the `.bak` file, substituting the value of the pod name and namespace name before you run it.

> [!NOTE]  
> Your container needs to have internet connectivity over TCP port 443 to download the file from GitHub.

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2025.bak -O /var/opt/mssql/data/AdventureWorks2025.bak
```

Example

```console
kubectl exec sqltest1-0 -n arc -c arc-sqlmi -- wget https://github.com/Microsoft/sql-server-samples/releases/download/adventureworks/AdventureWorks2025.bak -O /var/opt/mssql/data/AdventureWorks2025.bak
```

## Restore the AdventureWorks database

Similarly, you can run a `kubectl` exec command to use the **sqlcmd** CLI tool (included in the SQL managed instance container) to run the T-SQL command to `RESTORE DATABASE`.

Restore the database using a command similar to the following example. Replace the value of the pod name, the password, and the namespace name before you run it.

```console
kubectl exec <SQL pod name> -n <namespace name> -c arc-sqlmi -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P <password> -Q "RESTORE DATABASE AdventureWorks2025 FROM  DISK = N'/var/opt/mssql/data/AdventureWorks2025.bak' WITH MOVE 'AdventureWorks2025' TO '/var/opt/mssql/data/AdventureWorks2025.mdf', MOVE 'AdventureWorks2025_Log' TO '/var/opt/mssql/data/AdventureWorks2025_Log.ldf'"
```

Example

```console
kubectl exec sqltest1-0 -n arc -c arc-sqlmi -- /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P MyPassword! -Q "RESTORE DATABASE AdventureWorks2025 FROM DISK = N'/var/opt/mssql/data/AdventureWorks2025.bak' WITH MOVE 'AdventureWorks2025' TO '/var/opt/mssql/data/AdventureWorks2025.mdf', MOVE 'AdventureWorks2025_Log' TO '/var/opt/mssql/data/AdventureWorks2025_Log.ldf'"
```
