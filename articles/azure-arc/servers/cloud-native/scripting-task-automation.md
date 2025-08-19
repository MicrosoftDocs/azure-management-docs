---
title: Cloud-native scripting and task automation with Azure Arc-enabled servers
description: Azure Arc-enabled servers helps you automate tasks to make managing servers more like managing code.
ms.date: 08/19/2025
ms.topic: concept-article
# Customer intent: "As an administrator managing a hybrid cloud environment, I want to use Azure Arc-enabled servers' capabilities to automate tasks, so I can manage my hybrid environment more efficiently."
---

# Cloud-native scripting and task automation with Arc-enabled servers

Cloud-native task automation makes managing servers more like managing code. Central, scriptable control means you spend less time remoting in or writing one-off scripts, instead focusing on orchestration through Azure.

In a traditional environment, you might use Remote Desktop Protocol (RDP) to connect to a Windows server and run a PowerShell script, or use SSH for a Linux box, or use a tool like the  Run Script feature in System Center Configuration Manager (SCCM). Azure brings these capabilities to the cloud platform, letting you run tasks on any server from the Azure portal or the command line, without requiring a direct login. Azure also lets you ensure that only authorized individuals can run these commands through the use of [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview). All actions get logged in Azure Activity logs for auditing.

Let's explore the key tools that can help streamline and automate tasks: Azure Run Command, SSH via Azure Arc, and related automation that works with Arc-enabled servers.

## Azure Run Command

The [Azure Run Command feature](/azure/virtual-machines/run-command-overview) lets you execute scripts or commands on a virtual machine (VM) or Arc-enabled server remotely from Azure, like a remote hands-on-keyboard that doesn't require you to remote desktop or SSH in. [Run Command for Azure Arc-enabled servers](../run-command.md) is currently in public preview, bringing that power to on-premises machines.

Run Command is similar to using RDP and PowerShell, but more efficient and scriptable. With Run Command, you can do things like install or update software, change a configuration, or restart a service on a target server, all by sending the command through Azure. Imagine running a quick script on 50 servers at once – you can create a script in Azure CLI that invokes Run Command on a group of Arc-enabled servers in parallel, something that would be tedious with manual RDP. Using Run Command on Arc-enabled servers lowers overhead and helps improve security, since no additional open ports are required. Commands are run using the existing Arc agent's connection, so you don't need any open inbound ports or VPN to each server.

Run Command also supports centralized script management. You can store scripts in Azure and call them as needed, so that common automation tasks are ready when needed. For example, you could create and save a script that clears temporary files on a server, then run it across any of your Arc-enabled servers as needed. For critical security configurations or patches, you can deploy quickly by running a script without having to log into each server individually. These capabilities support modern cloud practices, treating scripts as assets and executing with minimal human intervention.

## SSH access via Azure Arc

With [SSH access to Arc-enabled servers](../ssh-arc-overview.md), you can open an interactive shell session to a Linux or Windows server through Azure, without requiring a public IP or direct network access. Essentially, Azure acts as a jump host, with no VPN or exposed SSH needed. You connect to the server's Arc agent, which proxies the SSH session either by running an Azure CLI command or by using the Azure portal (or for Linux, Azure can open a browser-based SSH). [PowerShell remoting over SSH](../ssh-arc-powershell-remoting.md) is also supported, and you can log in with local accounts. For Linux there's also support to use [Microsoft Entra authentication](../ssh-arc-overview.md#microsoft-entra-authentication). Only Azure users with the Owner or Contributor role can initiate SSH functionality.

Think of all those times you need to quickly hop onto a server to check something. Now you can do it from anywhere, and your connection is brokered by Azure, respecting Azure RBAC. This moves your break-fix and maintenance workflows into Azure's orbit and avoids fiddling with jump boxes or IP allowlists. As a system administrator, you’ll appreciate the ability to just run `az ssh` and get into a machine, and the transparency of Azure's logs tracking who connected when.

## Azure Automation and Logic Apps

Beyond on-demand scripts, Arc-enabled servers let you use [Azure Automation](/azure/automation/overview) accounts for PowerShell and Python runbooks, and [Azure Logic Apps](/azure/logic-apps/logic-apps-overview) and [Azure Functions](/azure/azure-functions/functions-overview) for server management tasks. For instance, you could schedule a runbook to clean up logs on all Arc-enabled servers every week, similar to a scheduled task or SCCM baseline script. While these are broader automation tools, they complement the on-demand scripting by providing orchestration and scheduling, effectively replacing your Windows Task Scheduler or SCCM scheduled jobs with cloud-based runbooks.

## Examples

Using these cloud-native scripting and automation tools in conjunction with Arc-enabled servers, you can achieve a wide range of tasks more efficiently. For instance, here are examples of things you can do with cloud-native scripting:

- Run a script across dozens of servers to gather log files and push them to a storage account, without logging into any server individually.
- Install an application or update by running an installer via Run Command on a server (Arc’s agent will execute it as LocalSystem). You could also automate rolling out an agent to many servers using this method.
- Quickly trigger a configuration change, such as a one-line fix needed to address a zero-day vulnerability, by crafting a script and running it on all affected machines. This could be done in minutes across on-premises and cloud, whereas traditionally you'd script it through whatever remote execution tools you have.
- Use [Azure Change Tracking and Inventory](/azure/automation/change-tracking/overview-monitoring-agent) in tandem with Run Command. For instance, you could detect that a certain service stopped on a server, trigger a Run Command to restart a certain service when it stops on a server. You could even automate this with alerts triggering a Logic App that calls Run Command. This is the kind of self-healing that becomes possible with everything connected through Azure.
