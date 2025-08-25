## Automatic connection for SQL Server

When you connect a Windows or Linux server to Azure Arc that also has Microsoft SQL Server installed, the SQL Server instances are automatically connected to Azure Arc as well. [SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/overview) provides a detailed inventory and additional management capabilities for your SQL Server instances and databases. As part of the connection process, an extension is deployed to your Azure Arc-enabled server, and [new roles](/sql/sql-server/azure-arc/permissions-granted-agent-extension) are applied to your SQL Server and databases. If you don't want to automatically connect your SQL Servers to Azure Arc, you can opt out by adding a tag to the Windows or Linux server with the name `ArcSQLServerExtensionDeployment` and value `Disabled` when connecting it to Azure Arc.

For more information, see [Manage automatic connection for SQL Server enabled by Azure Arc](/sql/sql-server/azure-arc/manage-autodeploy).
