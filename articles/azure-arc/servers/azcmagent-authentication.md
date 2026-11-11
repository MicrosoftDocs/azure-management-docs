---
title: Authenticate the azcmagent CLI
titleSuffix: Azure Connected Machine agent
description: Choose an Azure sign-in method for azcmagent commands, including interactive login, service principals, access tokens, and Azure CLI credentials.
ms.topic: how-to
ms.date: 11/10/2026
ai-usage: ai-assisted
---

# Authenticate the azcmagent CLI

This article describes the authentication methods and flags available for signing in to Azure with the `azcmagent` CLI. The following commands support these options:

- [`azcmagent connect`](azcmagent-connect.md#authentication-options).
- [`azcmagent reconnect`](azcmagent-reconnect.md#authentication-options).
- [`azcmagent disconnect`](azcmagent-disconnect.md#authentication-options).

Each command reference explains when authentication is required and which Azure permissions you need.

## Choose an authentication method

Choose any authentication method supported by your environment and use its flags in the `[authentication]` portion of the command's usage syntax. The following table suggests common uses for each method.

| Method | Suggested use |
| --- | --- |
| [Interactive browser login](#interactive-browser-login-windows-only) | Sign in interactively on Windows with a desktop experience. |
| [Device code authentication](#device-code-authentication) | Sign in by using a browser on another device, including for servers without a desktop. |
| [Service principal with secret](#service-principal-with-secret) | Authenticate noninteractively for scripted operations by using a client secret. |
| [Service principal with certificate](#service-principal-with-certificate) | Authenticate noninteractively for scripted operations by using a certificate instead of a client secret. |
| [Access token](#access-token) | Authenticate noninteractively by using a short-lived token obtained by another tool or automation process. |
| [Azure CLI credentials](#azure-cli-credentials) | Reuse an existing Azure CLI session on the server. |

For any authentication method, provide the tenant information required by the command, as described in its command reference.

### Interactive browser login (Windows only)

Interactive browser authentication is the default on Windows with a desktop experience. You don't need to specify an authentication flag. The sign-in page opens in your default browser. Complete browser authentication within five minutes. If browser authentication fails or times out, the agent falls back to device code authentication.

### Device code authentication

Device code authentication is the default interactive sign-in method on Linux. On Windows, use `--use-device-code` to select this method. This option lets you sign in through a browser on another device. The agent displays a URL and code. Open the URL on an internet-connected device, enter the code, and complete sign-in within five minutes.

### Service principal with secret

Use `--service-principal-id <application-id>` with `--service-principal-secret <secret>`.

Microsoft recommends supplying secrets through a [configuration file](#protect-credentials) rather than command-line arguments.

### Service principal with certificate

Use `--service-principal-id <application-id>` with `--service-principal-cert <path>`.

The agent accepts PKCS #12 (.PFX) files and ASCII-encoded files, such as .PEM, containing both the certificate and its private key. Password-protected PFX files and encrypted PEM files aren't supported.

On Windows, you can instead use `--service-principal-cert-thumbprint <thumbprint>` to select a certificate from a Windows certificate store. The agent searches the following stores under Current User and Local Machine:

- Personal (`MY`).
- Intermediate Certification Authorities (`CA`).
- Trusted Publishers (`TrustedPublisher`).
- Third-Party Root Certification Authorities (`AuthRoot`).

Exportable and nonexportable certificates are supported.

For information about creating the service principal, see [Use an Azure service principal with certificate-based authentication](/cli/azure/azure-cli-sp-tutorial-3).

### Access token

Use `--access-token <token>` to authenticate with a Microsoft Entra ID access token issued for the Azure Resource Manager API. You must obtain a valid token before running the command. The agent doesn't renew a token supplied through this flag.

You can obtain an Azure Resource Manager access token with Azure CLI ([`az account get-access-token`](/cli/azure/account#az-account-get-access-token)) or Azure PowerShell ([`Get-AzAccessToken`](/powershell/module/az.accounts/get-azaccesstoken)).

### Azure CLI credentials

Use `--use-azcli` to authenticate with an existing Azure CLI session on the server. This method requires Azure Connected Machine agent version 1.59 or later and [Azure CLI installed on the server](/cli/azure/install-azure-cli).

If you aren't already signed in, run:

```azurecli
az login
```

Then run the required `azcmagent` command with `--use-azcli`.

## Authentication flags

These flags are shared by the commands listed in this article. Resource flags, including command-specific requirements for `--tenant-id`, remain documented in each command reference.

| Flag | Description |
| --- | --- |
| `--access-token` | Supplies a Microsoft Entra access token. See [Access token](#access-token). |
| `-i`, `--service-principal-id` | Specifies the service principal's application ID. Use with a secret, certificate file, or certificate thumbprint. Follow the command reference's tenant requirements. |
| `-p`, `--service-principal-secret` | Specifies the service principal secret. Requires `--service-principal-id`. See [Service principal with secret](#service-principal-with-secret). |
| `--service-principal-cert` | Specifies the path to a certificate file containing the private key. Requires `--service-principal-id`. See [Service principal with certificate](#service-principal-with-certificate). |
| `--service-principal-cert-thumbprint` | Specifies a certificate thumbprint on Windows. Requires `--service-principal-id`. See [Service principal with certificate](#service-principal-with-certificate). |
| `-c`, `--use-device-code` | Uses device code authentication. |
| `--use-azcli` | Uses credentials from an Azure CLI session on the server. Requires agent version 1.59 or later. |
| `--user-tenant-id` | Specifies the tenant used for authentication, which can differ from the resource tenant, such as for a guest account or [Azure Lighthouse](/azure/lighthouse). Must be a GUID. |

## Protect credentials

To follow the principle of least privilege, use a service principal dedicated to agent operations and grant it only the permissions needed for the command. Permission to connect a new server doesn't necessarily include permission to disconnect or reset an existing server's identity.

Don't include secrets, access tokens, or private certificate material in logs or source control. For service principal secrets, use a JSON or YAML configuration file instead of command-line arguments. For example:

```json
{
  "service-principal-id": "<application-id>",
  "service-principal-secret": "<secret>"
}
```

Pass the file to the command with `--config <path>`, along with any required resource and tenant flags. Restrict access to the file and remove it when it's no longer needed. For configuration file syntax and command-line precedence, see [Common flags](azcmagent.md#common-flags).
