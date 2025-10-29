---
title: Troubleshoot Azure Connected Machine agent connection issues
description: This article tells how to troubleshoot and resolve issues with the Connected Machine agent that arise with Azure Arc-enabled servers when trying to connect to the service.
ms.date: 05/14/2025
ms.topic: troubleshooting
ms.custom:
  - build-2025
# Customer intent: "As a system administrator, I want to troubleshoot Azure Connected Machine agent connection errors, so that I can ensure successful configuration and management of Azure Arc-enabled servers."
---

# Troubleshoot Azure Connected Machine agent connection issues

This article provides information for troubleshooting issues that might occur while configuring the Azure Connected Machine agent for Windows or Linux. Tips for both [the interactive and at-scale installation methods when configuring connection](deployment-options.md) to the service are included.

For general information, see [Azure Arc-enabled servers overview](./overview.md), [Azure Connected Machine agent overview](./agent-overview.md), and [Manage and maintain the Connected Machine agent](./manage-agent.md).

## Agent error codes

Use the following table to identify and resolve issues when configuring the Azure Connected Machine agent using the `AZCM0000` ("0000" can be any four digit number) error code printed to the console or script output.

| Error code | Probable cause | Suggested remediation |
|------------|----------------|-----------------------|
| AZCM0000 | The action was successful | N/A |
| AZCM0001 | An unknown error occurred | Contact Microsoft Support for assistance. |
| AZCM0011 | The user canceled the action (CTRL+C) | Retry the previous command. |
| AZCM0012 | The access token is invalid | If authenticating via access token, obtain a new token and try again. If authenticating via service principal or device logins, contact Microsoft Support for assistance. |
| AZCM0016 | Missing mandatory parameter | Review the error message in the output to identify which parameters are missing. For the complete syntax of the command, run `azcmagent <command> --help`. |
| AZCM0018 | The command was executed without administrative privileges | Retry the command in an elevated user context (administrator/root). |
| AZCM0019 | The path to the configuration file is incorrect | Ensure the path to the configuration file is correct and try again. |
| AZCM0023 | The value provided for a parameter (argument) is invalid | Check for more specific information in the error message. Refer to the syntax of the command (`azcmagent <command> --help`) for valid values or expected format for the arguments. |
| AZCM0026 | There's an error in network configuration or some critical services are temporarily unavailable | Check if the required endpoints are reachable (for example, hostnames are resolvable, endpoints aren't blocked). If the network is configured for Private Link Scope, a Private Link Scope resource ID must be provided for onboarding using the `--private-link-scope` parameter. |
| AZCM0041 | The credentials supplied are invalid | For device logins, verify that the user account specified has access to the tenant and subscription where the server resource will be created. For service principal logins, check the client ID and secret for correctness and a valid expiration date, and ensure that the service principal is from the same tenant where the server resource will be created. |
| AZCM0042 | Creation of the Azure Arc-enabled server resource failed | Review the error message in the output to identify the cause of the failure to create resource and the suggested remediation. For more information, see [Required permissions](prerequisites.md#required-permissions). |
| AZCM0043 | Deletion of the Azure Arc-enabled server resource failed | Verify that the user/service principal specified has permissions to delete Azure Arc-enabled server/resources in the specified group. For more information, see [Required permissions](prerequisites.md#required-permissions). If the resource no longer exists in Azure, use the `--force-local-only` flag to proceed. |
| AZCM0044 | A resource with the same name already exists | Specify a different name for the `--resource-name` parameter or delete the existing Azure Arc-enabled server in Azure and try again. |
| AZCM0062 | An error occurred while connecting the server | Review the error message in the output for more specific information. If the error occurred after the Azure resource was created, delete this resource before retrying. |
| AZCM0063 | An error occurred while disconnecting the server | Review the error message in the output for more specific information. If this error persists, delete the resource in Azure, and then run `azcmagent disconnect --force-local-only` on the server. |
| AZCM0067 | The machine is already connected to Azure | Run `azcmagent disconnect` to remove the current connection, then try again. |
| AZCM0068 | Subscription name was provided, and an error occurred while looking up the corresponding subscription GUID. | Retry the command with the subscription GUID instead of subscription name. |
| AZCM0061<br>AZCM0064<br>AZCM0065<br>AZCM0066<br>AZCM0070<br> | The agent service isn't responding or unavailable | Verify the command is run in an elevated user context (administrator/root). Ensure that the HIMDS service is running (start or restart HIMDS as needed) then try the command again. |
| AZCM0081 | An error occurred while downloading the Microsoft Entra managed identity certificate | If this message is encountered while attempting to connect the server to Azure, the agent won't be able to communicate with the Azure Arc service. Delete the resource in Azure and try connecting again. |
| AZCM0101 | The command wasn't parsed successfully | Run `azcmagent <command> --help` to review the command syntax. |
| AZCM0102 | An error occurred while retrieving the computer hostname | Retry the command and specify a resource name (with parameter `--resource-name` or `–n`). Use only alphanumeric characters, hyphens and/or underscores; note that resource name can't end with a hyphen or underscore. |
| AZCM0103 | An error occurred while generating RSA keys | Contact Microsoft Support for assistance. |
| AZCM0105 | An error occurred while downloading the Microsoft Entra ID managed identify certificate | Delete the resource created in Azure and try again. |
| AZCM0147-<br>AZCM0152 | An error occurred while installing Azcmagent on Windows | Review the error message in the output for more specific information. |
| AZCM0127-<br>AZCM0146 | An error occurred while installing Azcmagent on Linux | Review the error message in the output for more specific information. |
| AZCM0150 | Generic failure during installation | Submit a support ticket to get assistance. |
| AZCM0153 | The system platform isn't supported | Review the [prerequisites](prerequisites.md) for supported platforms |
| AZCM0154 | The version of PowerShell installed on the system is too old | Upgrade to PowerShell 4 or later and try again. |
| AZCM0155 | The user running the installation script doesn't have administrator permissions | Run the script again as an administrator. |
| AZCM0156 | Installation of the agent failed | Confirm that the machine isn't running on Azure. Detailed errors might be found in the installation log at `%TEMP%\installationlog.txt`. |
| AZCM0157 | Unable to download repo metadata for the Microsoft Linux software repository | Check if a firewall is blocking access to `packages.microsoft.com` and try again. |


## Agent exit codes

When running Azure Connected Machine Agent (azcmagent) commands, the process may terminate with an exit code. These codes indicate the outcome of the operation and help diagnose issues. Unlike error codes, exit codes are returned by the operating system when the agent process exits.

| Exit Code | Description | Suggested Remediation |
|-----------|-------------|------------------------|
| **0** | No error occurred. | No action required. |
| **1** | Default error | Ensure prerequisites are met, check agent logs and retry. |
| **2** | Internal error in the agent. | Restart the agent service. |
| **3** | Operation not supported. | Verify the command is valid for your OS and agent version. |
| **4** | Arc proxy service is not running. | Ensure `Hybrid Instance Metadata Service (himds)` service is running. |
| **5** | File logger unavailable. | Check disk space and permissions for log directory. |
| **6** | Initialization failed. | Check prerequisites (network, permissions). Re-run `azcmagent connect` after fixing issues. |
| **11** | Operation interrupted by user (Ctrl+C). | Re-run the command without interruption. |
| **12** | Invalid access token provided. | Refresh your Azure credentials using `az login` or provide a valid token. |
| **18** | Administrative privileges required. | Run the command with elevated privileges (`sudo` or elevated command prompt). |
| **19** | Configuration file not found. | Verify the config file path or regenerate using `azcmagent config`. |
| **20** | Unknown region specified. | Check region spelling and ensure the region is supported. |
| **23** | Invalid arguments supplied. | Review command syntax using `azcmagent --help`. |
| **26** | Network error occurred. | Validate connectivity to Azure endpoints. Check firewall and proxy settings. |
| **27** | Configuration conflict detected. | Remove conflicting settings in `/etc/azcmagent/config.json` or `%ProgramData%\AzureConnectedMachineAgent\Config\localconfig.json` and retry. |
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
| **76** | Failed to open TPM device. | Verify TPM availability and permissions. |
| **77** | Failed to process TPM keys. | Check TPM health and retry. |
| **78** | Failed to connect using TPM-based authentication. | Validate TPM configuration and retry. |
| **81** | Failed to get MSI certificate from HIS. | Ensure HIS service is running and retry. |
| **82** | Failed to get MSI certificate from HIS using TPM. | Validate TPM configuration and HIS connectivity. |
| **101** | Command execution error. | Validate command syntax and check logs for details. |
| **102** | Unable to generate resource name. | Ensure hostname meets Azure naming requirements. |
| **103** | Failed to process RSA keys. | Check TPM availability and permissions. |
| **104** | Failed to retrieve private key. | Validate key storage and retry. |
| **105** | Failed to get signed message. | Check connectivity and retry. |
| **106** | Failed to save parameters file. | Verify disk space and permissions. |
| **107** | Failed to retrieve certificate. | Validate certificate store and retry. |


## Agent verbose log

To follow the troubleshooting steps described later in this article, the minimum information you need is the verbose log. This log contains the output of the **azcmagent** tool commands, when the verbose (`-v`) argument is used. The log files are written to `%ProgramData%\AzureConnectedMachineAgent\Log\azcmagent.log` for Windows, and Linux to `/var/opt/azcmagent/log/azcmagent.log`.

### Windows

The following command is an example that enables verbose logging with the Connected Machine agent for Windows when performing an interactive installation.

```console
& "$env:ProgramFiles\AzureConnectedMachineAgent\azcmagent.exe" connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

The following command is an example that enables verbose logging with the Connected Machine agent for Windows when performing an at-scale installation using a service principal.

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

The following command is an example that enables verbose logging with the Connected Machine agent for Linux when performing an interactive installation.

>[!NOTE]
>You must have *root* access permissions on Linux machines to run **azcmagent**.

```bash
azcmagent connect --resource-group "resourceGroupName" --tenant-id "tenantID" --location "regionName" --subscription-id "subscriptionID" --verbose
```

The following command is an example that enables verbose logging with the Connected Machine agent for Linux when performing an at-scale installation using a service principal.

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

|Error |Probable cause |Solution |
|--------|---------------|---------|
|Failed to acquire authorization token device flow:<br>`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is unreachable.` |Can't reach `login.windows.net` endpoint | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID. |
|Failed to acquire authorization token device flow:<br>`Error occurred while sending request for Device Authorization Code: Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/devicecode?api-version=1.0:  dial tcp 40.126.9.7:443: connect: network is Forbidden`. |Proxy or firewall is blocking access to `login.windows.net` endpoint. | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID.|
|Failed to acquire authorization token from SPN:<br>`Failed to execute the refresh request. Error = 'Post https://login.windows.net/fb84ce97-b875-4d12-b031-ef5e7edf9c8e/oauth2/token?api-version=1.0: Forbidden'` |Proxy or firewall is blocking access to `login.windows.net` endpoint. |Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Microsoft Entra ID. |
|Failed to acquire authorization token from SPN:<br>`Invalid client secret is provided` |Wrong or invalid service principal secret. |Verify the service principal secret. |
| Failed to acquire authorization token from SPN:<br>`Application with identifier 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' wasn't found in the directory 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'. This can happen if the application has not been installed by the administrator of the tenant or consented to by any user in the tenant` |Incorrect service principal and/or Tenant ID. |Verify the service principal and/or the tenant ID.|
|Get ARM Resource Response:<br>`The client 'username@domain.com' with object id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx' does not have authorization to perform action 'Microsoft.HybridCompute/machines/read' over scope '/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/`<br>`providers/Microsoft.HybridCompute/machines/MSJC01' or the scope is invalid. If access was recently granted, please refresh your credentials."}}" Status Code=403` |Wrong credentials and/or permissions |Verify you or the service principal is a member of the **Azure Connected Machine Onboarding** role. |
|Failed to AzcmagentConnect ARM resource:<br>`The subscription isn't registered to use namespace 'Microsoft.HybridCompute'` |Azure resource providers aren't registered. |Register the [resource providers](prerequisites.md#azure-resource-providers). |
|Failed to AzcmagentConnect ARM resource:<br>`Get https://management.azure.com/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourcegroups/myResourceGroup/providers/`<br>`Microsoft.HybridCompute/machines/MSJC01?api-version=2019-03-18-preview:  Forbidden` |Proxy server or firewall is blocking access to `management.azure.com` endpoint. | Run [`azcmagent check`](azcmagent-check.md) to see if a firewall is blocking access to Azure Resource Manager. |

## Next steps

If you don't see your problem here or you can't resolve your issue, try one of the following channels for support:

- Get answers from Azure experts through [Microsoft Q&A](/answers/topics/azure-arc.html).
- Open a support request to get assistance. For more information, see [Create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request).
