---
title: Authenticate against Azure resources with Azure Arc-enabled servers
description: This article describes Azure Instance Metadata Service support for Azure Arc-enabled servers and how you can authenticate against Azure resources and local using a secret.
ms.topic: concept-article
ms.date: 12/05/2024
---

# Authenticate against Azure resources with Azure Arc-enabled servers

Applications or processes running directly on an Azure Arc-enabled server can use managed identities to access other Azure resources that support Microsoft Entra ID-based authentication. An application can obtain an [access token](/azure/active-directory/develop/developer-glossary#access-token) representing its identity, which is system-assigned for Azure Arc-enabled servers, and use it as a 'bearer' token to authenticate itself to another service.

Refer to the [managed identity overview](/azure/active-directory/managed-identities-azure-resources/overview) documentation for a detailed description of managed identities, and understand the distinction between system-assigned and user-assigned identities.

In this article, you'll learn how a server can use a system-assigned managed identity to access Azure [Key Vault](/azure/key-vault/general/overview). Key Vault makes it possible for your client application to use a secret to access resources not secured by Microsoft Entra ID. For example, TLS/SSL certificates used by your IIS web servers can be stored in Azure Key Vault and securely deploy the certificates to Windows or Linux servers outside of Azure.

## Security overview

Several actions occur when onboarding your server to Azure Arc-enabled servers to configure using a managed identity (similar to what happens for an Azure VM):

- Azure Resource Manager receives a request to enable the system-assigned managed identity on the Azure Arc-enabled server.

- Azure Resource Manager creates a service principal in Microsoft Entra ID for the identity of the server. The service principal is created in the Microsoft Entra tenant that's trusted by the subscription.

- Azure Resource Manager configures the identity on the server by updating the Azure Instance Metadata Service (IMDS) identity endpoint for [Windows](/azure/virtual-machines/windows/instance-metadata-service) or [Linux](/azure/virtual-machines/linux/instance-metadata-service) with the service principal client ID and certificate. The endpoint is a REST endpoint accessible only from within the server using a well-known, non-routable IP address. This service provides a subset of metadata information about the Azure Arc-enabled server to help manage and configure it.

The environment of a managed-identity-enabled server is configured with the following variables on a Azure Arc-enabled server:

- **IMDS_ENDPOINT**: The IMDS endpoint IP address `http://localhost:40342` for Azure Arc-enabled servers.

- **IDENTITY_ENDPOINT**: the localhost endpoint corresponding to service's managed identity `http://localhost:40342/metadata/identity/oauth2/token`.

Your code that's running on the server can request a token from the Azure Instance Metadata service endpoint, accessible only from within the server.

The system environment variable **IDENTITY_ENDPOINT** is used to discover the identity endpoint by applications. Applications should try to retrieve **IDENTITY_ENDPOINT** and **IMDS_ENDPOINT** values and use them. Applications with any access level are allowed to make requests to the endpoints. Metadata responses are handled as normal and given to any process on the machine. However, when a request is made that would expose a token, we require the client to provide a secret to attest that they're able to access data only available to higher-privileged users.

## Prerequisites

- An understanding of Managed identities.
- On Windows, you must be a member of the local **Administrators** group or the **Hybrid Agent Extension Applications** group.
- On Linux, you must be a member of the **himds** group.
- A server connected and registered with Azure Arc-enabled servers.
- You're a member of the [Owner group](/azure/role-based-access-control/built-in-roles#owner) in the subscription or resource group (in order to perform required resource creation and role management steps).
- An Azure Key Vault to store and retrieve your credential, and assign the Azure Arc identity access to the KeyVault.

    - If you don't have a Key Vault created, see [Create Key Vault](/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad#create-a-key-vault-).
    - To configure access by the managed identity used by the server, see [Grant access for Linux](/azure/active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad#grant-access) or [Grant access for Windows](/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad#grant-access). For step number 5, you're going to enter the name of the Azure Arc-enabled server. To complete this using PowerShell, see [Assign an access policy using PowerShell](/azure/key-vault/general/assign-access-policy-powershell).

## Acquiring an access token using REST API

The method to obtain and use a system-assigned managed identity to authenticate with Azure resources is similar to how it's performed with an Azure VM.

For an Azure Arc-enabled Windows server, using PowerShell, invoke the web request to get the token from the local host in the specific port. Specify the request using the IP address or the environmental variable **IDENTITY_ENDPOINT**.

```powershell
$apiVersion = "2020-06-01"
$resource = "https://management.azure.com/"
$endpoint = "{0}?resource={1}&api-version={2}" -f $env:IDENTITY_ENDPOINT,$resource,$apiVersion
$secretFile = ""
try
{
    Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'} -UseBasicParsing
}
catch
{
    $wwwAuthHeader = $_.Exception.Response.Headers["WWW-Authenticate"]
    if ($wwwAuthHeader -match "Basic realm=.+")
    {
        $secretFile = ($wwwAuthHeader -split "Basic realm=")[1]
    }
}
Write-Host "Secret file path: " $secretFile`n
$secret = cat -Raw $secretFile
$response = Invoke-WebRequest -Method GET -Uri $endpoint -Headers @{Metadata='True'; Authorization="Basic $secret"} -UseBasicParsing
if ($response)
{
    $token = (ConvertFrom-Json -InputObject $response.Content).access_token
    Write-Host "Access token: " $token
}
```

The following response is an example that is returned:

:::image type="content" source="media/managed-identity-authentication/powershell-token-output-example.png" alt-text="A successful retrieval of the access token using PowerShell.":::

For an Azure Arc-enabled Linux server, using Bash, you invoke the web request to get the token from the local host in the specific port. Specify the following request using the IP address or the environmental variable **IDENTITY_ENDPOINT**. To complete this step, you need an SSH client.

```bash
CHALLENGE_TOKEN_PATH=$(curl -s -D - -H Metadata:true "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com" | grep Www-Authenticate | cut -d "=" -f 2 | tr -d "[:cntrl:]")
CHALLENGE_TOKEN=$(cat $CHALLENGE_TOKEN_PATH)
if [ $? -ne 0 ]; then
    echo "Could not retrieve challenge token, double check that this command is run with root privileges."
else
    curl -s -H Metadata:true -H "Authorization: Basic $CHALLENGE_TOKEN" "http://127.0.0.1:40342/metadata/identity/oauth2/token?api-version=2019-11-01&resource=https%3A%2F%2Fmanagement.azure.com"
fi
```

The following response is an example that is returned:

:::image type="content" source="media/managed-identity-authentication/bash-token-output-example.png" alt-text="A successful retrieval of the access token using Bash.":::

> [!NOTE]
> The above example is for requesting an access token for use with ARM REST APIs when the resource variable is set to `https://management.azure.com`. If you need an access token for a different Azure service, replace the resource variable in the script with the correct resource value. To authenticate with Azure Storage, see [Using OAuth Token with Azure Storage](/azure/storage/blobs/authorize-access-azure-active-directory#microsoft-authentication-library-msal). To complete the configuration to authenticate to Azure Key Vault, see [Access Key Vault with Windows](/azure/active-directory/managed-identities-azure-resources/tutorial-windows-vm-access-nonaad#access-data) or [Access Key Vault with Linux](/azure/active-directory/managed-identities-azure-resources/tutorial-linux-vm-access-nonaad#access-data).

## Next steps

- To learn more about Azure Key Vault, see [Key Vault overview](/azure/key-vault/general/overview).

- Learn how to assign a managed identity access to a resource [using PowerShell](/entra/identity/managed-identities-azure-resources/how-to-assign-access-azure-resource?pivots=identity-mi-access-powershell) or using [the Azure CLI](/entra/identity/managed-identities-azure-resources/how-to-assign-access-azure-resource?pivots=identity-mi-access-cli).
