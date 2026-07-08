---
title: Troubleshoot Azure Connected Machine agent connection issues
description: This article tells how to troubleshoot and resolve issues with the Connected Machine agent that arise with Azure Arc-enabled servers when trying to connect to the service.
ms.date: 05/14/2025
ms.topic: troubleshooting
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to troubleshoot Azure Connected Machine agent connection errors, so that I can ensure successful configuration and management of Azure Arc-enabled servers."
---

# Troubleshoot Azure Connected Machine agent connection problems

This article provides information for troubleshooting problems that might occur while configuring the Azure Connected Machine agent for Windows or Linux. It includes tips for both [the interactive and at-scale installation methods when configuring connection](deployment-options.md) to the service.

For general information, see [Azure Arc-enabled servers overview](./overview.md), [Azure Connected Machine agent overview](./agent-overview.md), and [Manage and maintain the Connected Machine agent](./manage-agent.md).

## Agent error codes

Use the following table to identify and resolve issues when configuring the Azure Connected Machine agent. The console or script output displays the `AZCM0000` error code, where "0000" can be any four-digit number.

| Error code | Probable cause | Suggested remediation |
| --- | --- | --- |
| AZCM0000 | The action was successful | N/A |
| AZCM0001 | An unknown error occurred | Contact Microsoft Support for assistance. |
| AZCM0002 | An internal error occurred in the agent | Restart the agent service (HIMDS) and try again. If the issue persists, contact Microsoft Support. |
| AZCM0003 | The requested operation isn't supported | Verify the command is valid for your operating system and agent version. |
| AZCM0004 | The Azure Arc proxy service isn't running | Ensure the Hybrid Instance Metadata Service (HIMDS) is running, then try again. |
| AZCM0005 | No file logger is available | Check that the log directory exists and has sufficient disk space and permissions. |
| AZCM0006 | Agent initialization failed | Check prerequisites (network connectivity, permissions), then run the command again. |
| AZCM0011 | The user canceled the action (CTRL+C) | Retry the previous command. |
| AZCM0012 | The access token is invalid | If authenticating via access token, obtain a new token and try again. If authenticating via service principal or device logins, contact Microsoft Support for assistance. |
| AZCM0016 | Missing mandatory parameter | Review the error message in the output to identify which parameters are missing. For the complete syntax of the command, run `azcmagent <command> --help`. |
| AZCM0018 | The command was executed without administrative privileges | Retry the command in an elevated user context (administrator/root). |
| AZCM0019 | The path to the configuration file is incorrect | Ensure the path to the configuration file is correct and try again. |
| AZCM0020 | An unknown region was specified | Check the region name for typos and ensure the region is supported. |
| AZCM0023 | The value provided for a parameter (argument) is invalid | Check for more specific information in the error message. Refer to the syntax of the command (`azcmagent <command> --help`) for valid values or expected format for the arguments. |
| AZCM0026 | There's an error in network configuration or some critical services are temporarily unavailable | Check if the required endpoints are reachable (for example, hostnames are resolvable, endpoints aren't blocked). If the network is configured for Private Link Scope, a Private Link Scope resource ID must be provided for onboarding using the `--private-link-scope` parameter. |
| AZCM0027 | A configuration conflict was detected | Remove conflicting settings in the agent configuration file and try again. |
| AZCM0028 | Failed to open the TPM device | Verify that the Trusted Platform Module (TPM) is enabled and accessible, then try again. |
| AZCM0041 | The credentials supplied are invalid | For device logins, verify that the user account specified has access to the tenant and subscription where the server resource will be created. For service principal logins, check the client ID and secret for correctness and a valid expiration date, and ensure that the service principal is from the same tenant where the server resource will be created. |
| AZCM0042 | Creation of the Azure Arc-enabled server resource failed | Review the error message in the output to identify the cause of the failure to create resource and the suggested remediation. For more information, see [Required permissions](prerequisites.md#required-permissions). |
| AZCM0043 | Deletion of the Azure Arc-enabled server resource failed | Verify that the user or service principal specified has permissions to delete Azure Arc-enabled server or resources in the specified group. For more information, see [Required permissions](prerequisites.md#required-permissions). If the resource no longer exists in Azure, use the `--force-local-only` flag to proceed. |
| AZCM0044 | A resource with the same name already exists | Specify a different name for the `--resource-name` parameter or delete the existing Azure Arc-enabled server in Azure and try again. |
| AZCM0045 | Failed to update the service with the new public key to reconnect | Verify network connectivity, then retry. Review the agent logs for details. |
| AZCM0061 | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0062 | An error occurred while connecting the server | Review the error message in the output for more specific information. If the error occurred after the Azure resource was created, delete this resource before retrying. |
| AZCM0063 | An error occurred while disconnecting the server | Review the error message in the output for more specific information. If this error persists, delete the resource in Azure, and then run `azcmagent disconnect --force-local-only` on the server. |
| AZCM0064 | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0065 | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0066 | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0067 | The machine is already connected to Azure | Run `azcmagent disconnect` to remove the current connection, then try again. |
| AZCM0068 | Subscription name was provided, and an error occurred while looking up the corresponding subscription GUID. | Retry the command with the subscription GUID instead of subscription name. |
| AZCM0069 | An error occurred while completing the local configuration update | Check file permissions for the agent configuration directory and try again. |
| AZCM0070 | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0072 | An error occurred while running an extension tool | Review the extension logs for details and retry. |
| AZCM0073 | Unable to obtain partner configuration properties | Validate the partner integration settings and try again. |
| AZCM0074 | An error occurred while adding extensions | Ensure the extension package is valid and try again. |
| AZCM0075 | Unable to obtain cloud configuration | Check connectivity to the required Azure endpoints and try again. |
| AZCM0076 | Failed to connect the machine to Azure using TPM-based authentication | Verify that the Trusted Platform Module (TPM) is enabled and accessible, then try again. |
| AZCM0081 | An error occurred while downloading the Microsoft Entra managed identity certificate | If this message is encountered while attempting to connect the server to Azure, the agent won't be able to communicate with the Azure Arc service. Delete the resource in Azure and try connecting again. |
| AZCM0082 | Failed to get the managed identity certificate from the Hybrid Instance Metadata Service (HIMDS) using TPM | Ensure the HIMDS service is running and the TPM is accessible, then try again. |
| AZCM0083 | Failed to register the Arc persistent credential with the service | Verify network connectivity and try again. Review the agent logs for details. |
| AZCM0101 | An error occurred while executing the command | Review the command syntax (`azcmagent <command> --help`) and check the agent logs for more specific information. |
| AZCM0102 | An error occurred while retrieving the computer hostname | Retry the command and specify a resource name (with parameter `--resource-name` or `–n`). Use only alphanumeric characters, hyphens, and/or underscores; note that resource name can't end with a hyphen or underscore. |
| AZCM0103 | An error occurred while generating RSA keys | Contact Microsoft Support for assistance. |
| AZCM0105 | Failed to get the signed message | Verify network connectivity to the Azure Arc service and try again. |
| AZCM0107 | Failed to retrieve the certificate | Validate the certificate store and try again. |
| AZCM0108 | Failed to process TPM keys | Verify that the Trusted Platform Module (TPM) is enabled and accessible, then try again. |
| AZCM0130 | The agent requires systemd, but the `systemctl` command wasn't found in PATH (Linux). | Use a systemd-based init system on a [supported distribution](prerequisites.md#supported-operating-systems). |
| AZCM0131 | The Linux distribution couldn't be identified during installation. | Verify the OS is a [supported distribution](prerequisites.md#supported-operating-systems). |
| AZCM0132 | The processor architecture or platform isn't supported (Linux). | Use a [supported platform/architecture](prerequisites.md#supported-operating-systems). |
| AZCM0133 | The Linux distribution or its ARM64 variant isn't supported. | Use a [supported distribution and architecture](prerequisites.md#supported-operating-systems). |
| AZCM0141 | The agent can't be installed on an Azure virtual machine (Linux). | Azure VMs are managed natively; don't install the Connected Machine agent on them. For testing only, see [https://aka.ms/azcmagent-testwarning](https://aka.ms/azcmagent-testwarning). |
| AZCM0143 | The package manager (`apt`/`dnf`/`zypper`) failed to install the azcmagent package (Linux). | Review the package-manager command logs for details and retry. |
| AZCM0145 | The apt/dpkg lock was still held after 5 minutes (Linux). | Ensure no other apt/dpkg operation is running, then retry. |
| AZCM0146 | Failed to download the Microsoft package repository configuration (`packages-microsoft-prod`) (Linux). | Check whether a firewall is blocking `packages.microsoft.com` and retry. |
| AZCM0147 | On Linux, the requested `--desiredversion` wasn't found; on Windows, the agent can't be installed on an Azure virtual machine. | Linux: specify an available version. Windows: don't install on Azure VMs (for testing only, see [https://aka.ms/azcmagent-testwarning](https://aka.ms/azcmagent-testwarning)). |
| AZCM0148 | Failed to download the agent installer (.msi) (Windows). | Verify network, proxy, firewall access to the download endpoint and retry. |
| AZCM0149 | The Windows installer (msiexec) returned an unexpected error code. | Review the installation log and the reported msiexec exit code, and then retry. |
| AZCM0150 | Generic failure during installation. | Submit a support ticket to get assistance. |
| AZCM0151 | The required .NET Framework version isn't installed (Windows). | Install the required .NET Framework version or later and try again. |
| AZCM0152 | The server is running Azure Stack HCI (Windows). | Connect it to Azure Arc by using the built-in registration experience: [https://aka.ms/install-arc-on-hci-host](https://aka.ms/install-arc-on-hci-host). |
| AZCM0153 | The operating system architecture isn't supported; the agent requires a 64-bit (x64) OS (Windows). | Install on a supported x64 operating system. |
| AZCM0154 | The installed PowerShell version is too old (Windows). | Upgrade to PowerShell 3.0 or later and try again. |
| AZCM0155 | The installation wasn't run with administrator permissions (Windows). | Run the installation script again as an administrator. |
| AZCM0156 | A fatal error occurred during MSI installation (Windows; msiexec 1603). | Review the installation log at `%SystemRoot%\AzureConnectedMachineAgent\temp\installationlog.txt` for details and retry. |
| AZCM0157 | The installer couldn't run `curl` because of a permission error (Linux). | Ensure `curl` is installed and executable by the install user, and then retry. |
| AZCM0158 | Installation failed due to insufficient disk space (Windows; msiexec 112). | Free up disk space and retry. |
| AZCM0159 | Another installation is already in progress (Windows; msiexec 1618). | Wait for the other installation to finish, and then retry. |
| AZCM0160 | The installer couldn't open its log file (Windows; msiexec 1622). | Verify the log directory exists and is writable, and then retry. |
| AZCM0171 | The agent MSI failed signature validation (Windows). | Ensure the installer is the genuine, Microsoft-signed package and isn't tampered with, and then retry. |

## Agent exit codes

When you run Azure Connected Machine Agent (azcmagent) commands, the process might terminate with an exit code. These codes indicate the outcome of the operation and help diagnose problems. Unlike error codes, the operating system returns exit codes when the agent process exits.

| Exit Code | Description | Suggested Remediation |
| --- | --- | --- |
| **0** | No error occurred. | No action required. |
| **1** | Default error. | Ensure prerequisites are met, check agent logs, and retry. |
| **2** | Internal error in the agent. | Restart the agent service. |
| **3** | Operation not supported. | Verify the command is valid for your OS and agent version. |
| **4** | Arc proxy service isn't running. | Ensure `Hybrid Instance Metadata Service (himds)` service is running. |
| **5** | File logger unavailable. | Check disk space and permissions for log directory. |
| **6** | Initialization failed. | Check prerequisites (network, permissions). Re-run `azcmagent connect` after fixing issues. |
| **11** | Operation interrupted by user (Ctrl+C). | Re-run the command without interruption. |
| **12** | Invalid access token provided. | Refresh your Azure credentials by using `az login` or provide a valid token. |
| **18** | Administrative privileges required. | Run the command with elevated privileges (`sudo` or elevated command prompt). |
| **19** | Configuration file not found. | Verify the config file path or regenerate by using `azcmagent config`. |
| **20** | Unknown region specified. | Check region spelling and ensure the region is supported. |
| **23** | Invalid arguments supplied. | Review command syntax by using `azcmagent --help`. |
| **26** | Network error occurred. | Validate connectivity to Azure endpoints. Check firewall and proxy settings. |
| **27** | Configuration conflict detected. | Remove conflicting settings in `/etc/azcmagent/config.json` or `%ProgramData%\AzureConnectedMachineAgent\Config\localconfig.json` and retry. |
| **28** | Failed to open the TPM device. | Verify that the Trusted Platform Module (TPM) is enabled and accessible, then retry. |
| **41** | Failed to obtain access token. | Ensure `az login` is successful and MSI is enabled if applicable. |
| **42** | Failed to create Azure resource. | Check subscription permissions and resource quota. |
| **43** | Failed to delete Azure resource. | Verify resource exists and you have delete permissions. |
| **44** | Resource already exists. | Use `azcmagent reconnect` instead of `connect`. |
| **45** | Failed to update reconnect public key. | Retry after verifying network connectivity and agent logs. |
| **61** | Agent communication error. | Restart the himds service. |
| **62** | Failed to connect machine to Azure. | Check network connectivity and subscription permissions. |
| **63** | Failed to disconnect machine from Azure. | Retry after ensuring the machine is online and agent is healthy. |
| **64** | Unable to obtain establish communication with HIMDS server. | Restart `himds` service and verify logs. |
| **65** | Unable to obtain agent metadata. | Check agent logs and retry. |
| **66** | Unable to obtain agent status. | Restart agent and verify connectivity. |
| **67** | Machine already connected. | Use `azcmagent reconnect` instead of `connect`. |
| **68** | Unable to fetch subscription ID. | Validate Azure credentials and retry. |
| **69** | Error updating local configuration. | Check file permissions and retry. |
| **70** | Unable to obtain local configuration. | Verify config file integrity and retry. |
| **72** | Error running extension tool. | Check extension logs and retry. |
| **73** | Unable to obtain partner configuration. | Validate partner integration settings. |
| **74** | Error adding extension. | Ensure extension package is valid and retry. |
| **75** | Unable to obtain cloud configuration. | Check connectivity to Azure endpoints. |
| **76** | Failed to connect machine to Azure using TPM based authentication. | Verify that the TPM is enabled and accessible, then retry. |
| **81** | Failed to get MSI certificate from HIS. | Ensure HIS service is running and retry. |
| **82** | Failed to get MSI certificate from HIS using TPM. | Ensure HIS service is running and the TPM is accessible, then retry. |
| **83** | Failed to register Arc persistent credential with the service. | Verify network connectivity and retry. |
| **101** | Command execution error. | Validate command syntax and check logs for details. |
| **102** | Unable to generate resource name. | Ensure hostname meets Azure naming requirements. |
| **103** | Failed to process RSA keys. | Contact Microsoft Support for assistance. |
| **104** | Failed to retrieve private key. | Validate key storage and retry. |
| **105** | Failed to get signed message. | Check connectivity and retry. |
| **106** | Failed to save parameters file. | Verify disk space and permissions. |
| **107** | Failed to retrieve certificate. | Validate certificate store and retry. |
| **108** | Failed to process TPM keys. | Verify that the TPM is enabled and accessible, then retry. |

## Agent verbose log

To follow the troubleshooting steps described later in this article, you need the verbose log. This log contains the output of the **azcmagent** tool commands when you use the verbose (`-v`) argument. The log files are written to `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` for Windows, and Linux to `/var/opt/azcmagent/log/azcmagent.log`.

### Windows

The following command is an example that enables verbose logging by using the Connected Machine agent for Windows when performing an interactive installation.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

The following command is an example that enables verbose logging by using the Connected Machine agent for Windows when performing an at-scale installation by using a service principal.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect `
  --service-principal-id "{serviceprincipalAppID}" `
  --service-principal-secret "{serviceprincipalPassword}" `
  --resource-group "{ResourceGroupName}" `
  --tenant-id "{tenantID}" `
  --location "{resourceLocation}" `
  --subscription-id "{subscriptionID}"
  --verbose
```

### Linux

The following command is an example that enables verbose logging by using the Connected Machine agent for Linux when performing an interactive installation.

> [!NOTE]
> You must have _root_ access permissions on Linux machines to run **azcmagent**.

```bash
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

The following command is an example that enables verbose logging by using the Connected Machine agent for Linux when performing an at-scale installation by using a service principal.

```bash
azcmagent connect \
  --service-principal-id "{serviceprincipalAppID}" \
  --service-principal-secret "{serviceprincipalPassword}" \
  --resource-group "{ResourceGroupName}" \
  --tenant-id "{tenantID}" \
  --location "{resourceLocation}" \
  --subscription-id "{subscriptionID}"
  --verbose
```

## Agent connection issues to service

The following table lists various errors and suggestions on how to troubleshoot and resolve them.

| Error | Probable cause | Solution |
| --- | --- | --- |
| Failed to acquire authorization token device flow:<br>`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` | Can't reach `login.windows.net` endpoint | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID. |
| Failed to acquire authorization token device flow:<br>`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. | Proxy or firewall is blocking access to `login.windows.net` endpoint. | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID. |
| Failed to acquire authorization token from SPN:<br>`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` | Proxy or firewall is blocking access to `login.windows.net` endpoint. | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID. |
| Failed to acquire authorization token from SPN:<br>`Invalid client secret is provided` | Wrong or invalid service principal secret. | Verify the service principal secret. |
| Failed to acquire authorization token from SPN:<br>`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' wasn't found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` | Incorrect service principal and/or Tenant ID. | Verify the service principal and/or the tenant ID. |
| Get ARM Resource Response:<br>`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/`<br>`providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` | Wrong credentials and/or permissions | Verify you or the service principal is a member of the **Azure Connected Machine Onboarding** role. |
| Failed to AzcmagentConnect ARM resource:<br>`The subscription isn't registered to use namespace 'Microsoft.HybridCompute'` | Azure resource providers aren't registered. | Register the [resource providers](prerequisites.md#azure-resource-providers). |
| Failed to AzcmagentConnect ARM resource:<br>`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/`<br>`Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` | Proxy server or firewall is blocking access to `management.azure.com` endpoint. | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Azure Resource Manager. |

## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Open a support request to get assistance. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).

