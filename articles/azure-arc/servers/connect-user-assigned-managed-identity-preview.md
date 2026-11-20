---
title: Connect an Azure Arc-enabled server with a user-assigned managed identity (preview)
description: Connect an Azure Arc-enabled server with user-assigned managed identities and safely manage identity assignments during the preview.
ms.service: azure-arc
ms.topic: how-to
ms.date: 11/10/2026
---

# Connect an Azure Arc-enabled server with a user-assigned managed identity (preview)

You can connect an Azure Arc-enabled server with one or more existing user-assigned managed identities (UAMIs). During onboarding, choose whether to use only user-assigned identities or to retain a system-assigned managed identity (SAMI) alongside them. This article also explains how to manage identity assignments on an existing, supported TPM-backed server.

Attaching a managed identity doesn't grant it access to other Azure resources. To authorize access, assign the identity the roles required by your application at the target resource's scope. The account that connects the server, the identities attached to the server, and the permissions those identities have on other resources are separate.

> [!IMPORTANT]
> This experience is in preview. No preview feature registration is required.

## Prerequisites

Before you begin, ensure you have:

- A server that meets the [Azure Arc-enabled servers prerequisites](prerequisites.md).
- Azure Connected Machine agent version **1.69 or later**. If needed, [upgrade the agent](manage-agent.md?tabs=windows).
- Administrator or root access on the server to run agent commands.
- The server must be onboarded to Azure Arc using [TPM-backed identity](tpm-backed-identity.md).
- Registration of the `Microsoft.HybridCompute` resource provider, as described in [Azure resource providers](prerequisites.md#azure-resource-providers).
- One or more existing [user-assigned managed identities](/entra/identity/managed-identities-azure-resources/manage-user-assigned-managed-identities-azure-portal) in the same Microsoft Entra tenant as the Arc-enabled server, and the full Azure Resource Manager resource ID of each identity.
- The target subscription ID, resource group, Azure region, and tenant ID for onboarding.
- Azure CLI for the Azure commands in this article, and credentials for the subscription and resources you use.

### Required permissions

Use an Azure account or service principal with permissions for each operation you perform. Access to the Arc-enabled server doesn't automatically grant access to the UAMI resources.

| Operation | Role | Assignment scope |
| --- | --- | --- |
| Onboard a server. | **Azure Connected Machine Onboarding** or **Contributor**. | The resource group where you create the Arc-enabled server. |
| Update an existing server's identity configuration. | **Azure Connected Machine Resource Administrator**. | The resource group that contains the Arc-enabled server. |
| Read and assign an existing UAMI. | [Managed Identity Operator](/azure/role-based-access-control/built-in-roles/identity#managed-identity-operator). | Each UAMI resource you attach, or a parent scope that contains those identities. |

For the server roles, see [Required permissions](prerequisites.md#required-permissions). Managed Identity Operator grants permission to read and assign a UAMI; it doesn't grant the UAMI access to application resources.

Attaching a UAMI doesn't replace onboarding authentication. Authenticate the connect operation with a supported method, such as a service principal, an access token, device-code authentication, or Azure CLI authentication. For onboarding approaches, see [Deployment options](deployment-options.md). For authentication parameters, see [azcmagent connect](azcmagent-connect.md).

### Command conventions

The multiline shell examples in this article use **Bash** syntax. A backslash (`\`) continues a command on the next line; don't add spaces after it. For another shell, adapt the line continuations or enter the command on one line. Run `azcmagent` commands on the server with administrator or root privileges. Replace all angle-bracketed values before running a sample.

## Choose an identity configuration

Choose the identity types before you onboard the server. The Azure Connected Machine agent supports up to 10 user-assigned managed identities per server.

> [!IMPORTANT]
> **Known issue:** [Azure Arc gateway](arc-gateway.md) doesn't currently support UAMI-only servers. If you use Azure Arc gateway, choose `SystemAssigned,UserAssigned` and keep the system-assigned managed identity associated with the server.

| Desired configuration | Onboarding setting | Resulting identity type |
| --- | --- | --- |
| Use only user-assigned identities. | Set `--system-assigned-identity=false` and specify the UAMIs. | `UserAssigned`. |
| Keep a system-assigned identity alongside user-assigned identities. | Specify the UAMIs and omit `--system-assigned-identity` to retain its default value of `true`. | `SystemAssigned,UserAssigned`. |

You can add a system-assigned managed identity after onboarding, including to a server onboarded with only user-assigned identities.



## Prepare the server

1. On the server, check the agent version and verify the TPM:

   ```console
   azcmagent --version
   azcmagent check tpm
   ```

   Confirm that the agent is version 1.69 or later and that the TPM is ready before you connect. If the server doesn't have a working TPM 2.0, resolve that issue first. The agent checks the TPM requirement locally and rejects UAMI onboarding before it calls Azure when the requirement isn't met.

1. Get the resource ID of an existing user-assigned identity. Repeat this command for each identity you want to attach:

   ```azurecli
   az identity show --name "<uami-name>" --resource-group "<uami-resource-group>" --subscription "<uami-subscription-id>" --query id -o tsv
   ```

   Use the complete returned ID, in this format:

   ```text
   /subscriptions/<uami-subscription-id>/resourceGroups/<uami-resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name>
   ```

## Connect the server

Run the command for the [identity configuration you chose](#choose-an-identity-configuration).

Both configurations require `--identity-key-store tpm`. The preview parameters `--user-assigned-identities` and `--system-assigned-identity` don't appear in `azcmagent connect` help; their absence from help is expected during this preview.

The following commands use the default interactive onboarding flow. Add the supported authentication arguments for your chosen method if you don't use that flow. The examples specify `--tenant-id` explicitly; you can omit it when your credentials infer the tenant.

### Option A: Use only user-assigned identities

Run the following command to disable the system-assigned identity and attach a UAMI. The resulting identity type is `UserAssigned`.

```bash
azcmagent connect \
  --subscription-id "<subscription-id>" \
  --resource-group "<resource-group>" \
  --location "<region>" \
  --identity-key-store tpm \
  --system-assigned-identity=false \
  --user-assigned-identities "<uami-resource-id>" \
  --tenant-id "<tenant-id>"
```

### Option B: Keep the system-assigned identity and add user-assigned identities

Run the following command to keep the default system-assigned identity and attach a UAMI. Omitting `--system-assigned-identity=false` retains the default value of `true`. The resulting identity type is `SystemAssigned,UserAssigned`.

```bash
azcmagent connect \
  --subscription-id "<subscription-id>" \
  --resource-group "<resource-group>" \
  --location "<region>" \
  --identity-key-store tpm \
  --user-assigned-identities "<uami-resource-id>" \
  --tenant-id "<tenant-id>"
```

### Attach multiple user-assigned identities

In either command, replace the `--user-assigned-identities` argument with a comma-separated list of full resource IDs. You don't need spaces around the commas. The following is an argument fragment, not a standalone command:

```text
--user-assigned-identities "<uami-resource-id-1>,<uami-resource-id-2>"
```

## Verify the connection and identity credentials

1. On the server, check the connection:

   ```console
   azcmagent show
   ```

   Confirm that the agent reports `Connected`.

1. Check the identity assignments in Azure. Open the [preview Azure portal experience](https://aka.ms/arc-identity-toc), go to the Arc-enabled server, and select **Identity**. Confirm that the expected UAMIs are listed and that the system-assigned identity is present if you retained it. Use this preview entry point; the Identity experience might not be visible through the standard portal entry point.

   Alternatively, use Azure CLI to read the server's identity configuration:

   ```bash
   az rest \
     --method get \
     --url "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.HybridCompute/machines/<machine-name>?api-version=2026-06-16-preview" \
     --query identity
   ```

   Confirm that `type` matches your chosen configuration and that `userAssignedIdentities` contains each expected full UAMI resource ID as a key. This response confirms the assignments in Azure, not whether credentials are ready on the server.

1. On the server, check the local identity credentials:

   ```console
   azcmagent identity list
   ```

   The following example output shows two user-assigned identities. Resource IDs and identity IDs are placeholders.

   ```text
   UserAssigned
     /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name-1>
       Client Id    : <client-id-1>
       Principal Id : <principal-id-1>
       Renew After  : 2026-12-12T18:22:00Z
       Expires      : 2027-01-26T18:22:00Z
       State        : Active
     /subscriptions/<subscription-id>/resourcegroups/<resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name-2>
       Client Id    : <client-id-2>
       Principal Id : <principal-id-2>
       Renew After  : 2026-12-12T18:27:00Z
       Expires      : 2027-01-26T18:27:00Z
       State        : Active
   ```

   Under `UserAssigned`, match each full resource ID to the intended UAMI assignment in Azure and confirm that its `State` is `Active`. Don't check only the number of entries. Each entry also includes `Client Id`, `Principal Id`, `Renew After`, and `Expires`; the output doesn't include a separate sealed-credential field.

   If a newly assigned identity isn't listed yet, allow the next agent heartbeat to complete and run the command again. When you add a UAMI through the Azure portal, replication to the agent typically takes up to 5 minutes. Keep the agent connected during verification. For the planned recovery behavior when a TPM-backed server has no identity, see [Identity type None on TPM-backed servers](#identity-type-none-on-tpm-backed-servers). To avoid interruptions, don't detach an existing identity until the replacement is listed with `State` set to `Active`.

   If the identity is still missing or isn't active after reconciliation, follow [An assigned identity has no local credential yet](#an-assigned-identity-has-no-local-credential-yet).

## Acquire an access token for a user-assigned identity

After you [verify the local identity credentials](#verify-the-connection-and-identity-credentials), applications on the server can request access tokens for an attached UAMI. Token acquisition has the following tool and library requirements, separate from the agent version required for onboarding:

| Tool or library | Requirement for Arc UAMI token acquisition |
| --- | --- |
| Azure CLI. | Version **2.91.0 or later**. |
| Microsoft Authentication Library for .NET (MSAL.NET). | Version **4.88.0 or later** for .NET applications and SDKs that use MSAL.NET. |
| Azure PowerShell. | Support and the minimum version are pending confirmation for this preview. |

### Use Azure CLI

Run the following commands on the Arc-enabled server. Use the UAMI's `Client Id` from `azcmagent identity list`, not its principal ID or Azure resource ID, for `--client-id`.

```azurecli
az login --identity --client-id "<uami-client-id>"
az account get-access-token --resource "https://management.azure.com/"
```

This example requests a token for Azure Resource Manager. For another service, set `--resource` to that service's resource URI. The UAMI must have the required role assignments on the target resource to use the token to access it. Signing in with a managed identity here is separate from authenticating the onboarding operation.

For other ways to select a UAMI, see [Sign into Azure with a managed identity using Azure CLI](/cli/azure/authenticate-azure-cli-managed-identity).

### Use an application or SDK

Use the [managed identity token-acquisition guidance](/entra/identity/managed-identities-azure-resources/how-to-use-vm-token) for identity selection and token handling. Arc-enabled servers follow the same UAMI token-request contract. Explicitly select the intended identity, especially when multiple UAMIs are attached, rather than relying on a default identity. For REST requests, select one identity with `client_id`, `object_id`, or `msi_res_id`.

> [!IMPORTANT]
> The linked guidance includes Azure VM examples. On an Arc-enabled server, use the local Arc identity endpoint exposed by `IDENTITY_ENDPOINT`, not the VM endpoint at `169.254.169.254`. Arc also requires local authorization through its challenge-response flow. For the Arc endpoint and local access requirements, see [Access Azure resources with managed identity](managed-identity-authentication.md). That article's existing token examples use the system-assigned identity; select the UAMI explicitly when adapting them.

## Manage identities after onboarding

Use this procedure only for an existing connected server that uses the TPM identity key store and meets the supported preview prerequisites. This procedure doesn't cover migration from a file-based identity key store to TPM.

Verify the existing server's identity key store in Azure:

```azurecli
az resource show --ids "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.HybridCompute/machines/<machine-name>" --query "properties.identityKeyStore" --output tsv
```

Confirm that the returned value is `Tpm` before changing identities. A successful `azcmagent check tpm` only establishes TPM readiness; it doesn't establish that the server uses the TPM identity key store.

For identity changes, use the Azure portal, Azure CLI identity commands, or an identity-only HTTP `PATCH` request to the `Microsoft.HybridCompute/machines` resource. Don't reconnect the server to update its identity assignments, and don't use a full `PUT` request or a template that replaces unrelated resource properties. The agent reconciles the changes and seals identity certificates at its next heartbeat.

### Identity type None on TPM-backed servers

When `identityType` is `None`, a TPM-backed server enters the `Degraded` state. While the server is in this state, the agent can't issue managed identity access tokens. Extension installation, guest policy, and extensions are unavailable until an identity is associated with the Arc-enabled server.

While in `Degraded`, the server continues polling the Azure Arc data plane for new identity associations. After you associate an identity, the server can recover to `Connected` without reonboarding.

### Add a user-assigned identity

Before you begin, verify that the existing server is connected and uses the TPM identity key store. Follow [Verify the connection and identity credentials](#verify-the-connection-and-identity-credentials) to check its Azure assignments and local credentials. Then choose a method to add a UAMI.

# [Azure portal](#tab/azure-portal)

1. Sign in to the [Azure portal](https://portal.azure.com).
1. Go to **Azure Arc** > **Machines**, and select your Arc-enabled server.
1. In the service menu, under **Security**, select **Identity (Preview)**.
1. On the **User assigned** tab, select **Add**.

   :::image type="content" source="media/user-assigned-managed-identity/add-user-assigned-identity-portal.png" alt-text="Screenshot of the Azure Arc server Identity (Preview) page with the User assigned tab selected." lightbox="media/user-assigned-managed-identity/add-user-assigned-identity-portal.png":::

1. Select the subscription that contains the UAMI, select the identity, and then select **Add**.

# [Azure CLI](#tab/azure-cli)

Use [az connectedmachine identity assign](/cli/azure/connectedmachine/identity#az-connectedmachine-identity-assign) to add the UAMI. This command requires Azure CLI version 2.75.0 or later. The `connectedmachine` extension installs automatically the first time you run the command.

```azurecli
az connectedmachine identity assign \
  --name "<machine-name>" \
  --resource-group "<resource-group>" \
  --subscription "<subscription-id>" \
  --user-assigned "/subscriptions/<uami-subscription-id>/resourceGroups/<uami-resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name>"
```

Use the subscription and resource group of the Arc-enabled server for `--subscription` and `--resource-group`. Use the full resource ID of the UAMI for `--user-assigned`.

# [REST API](#tab/rest-api)

Send the following identity-only `PATCH` request to add a UAMI and retain or add a system-assigned identity. This example uses API version `2026-06-16-preview`.

To use only user-assigned identities, change the `type` value to `UserAssigned` before you run the request.

```bash
az rest \
  --method patch \
  --url "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.HybridCompute/machines/<machine-name>?api-version=2026-06-16-preview" \
  --headers "Content-Type=application/json" \
  --body '{
    "identity": {
      "type": "SystemAssigned,UserAssigned",
      "userAssignedIdentities": {
        "/subscriptions/<uami-subscription-id>/resourceGroups/<uami-resource-group>/providers/Microsoft.ManagedIdentity/userAssignedIdentities/<uami-name>": {}
      }
    }
  }'
```

In this preview API, `userAssignedIdentities` uses merge semantics. An ID with `{}` adds the identity; an omitted identity remains attached. You can add multiple identities by listing each full resource ID as a separate key with `{}` as its value. An ID with `null` detaches that identity.

---

After you add the identity, allow the next heartbeat to complete. Repeat the [verification steps](#verify-the-connection-and-identity-credentials) to confirm that the server is still connected, Azure shows the intended assignments, and the newly added identity is listed locally with `State` set to `Active`.

### Replace or detach an identity safely

To avoid interruptions when replacing a user-assigned identity, add and verify the replacement before detaching the existing identity. Use separate updates. If you only need to detach an identity, first confirm that another identity is listed locally with `State` set to `Active`.

You can replace or detach a UAMI by using the Azure portal or Azure CLI:

- **Azure portal:** On your Arc-enabled server, go to **Security** > **Identity (Preview)** > **User assigned**. Use **Add** to attach the replacement identity. After you verify it, select the old identity and select **Remove**.
- **Azure CLI:** Use [az connectedmachine identity assign](/cli/azure/connectedmachine/identity#az-connectedmachine-identity-assign) to add the replacement identity. After you verify it, use [az connectedmachine identity remove](/cli/azure/connectedmachine/identity#az-connectedmachine-identity-remove) with `--user-assigned` set to the old UAMI's full resource ID to detach it.

With either method, allow the next heartbeat to complete and repeat the [verification steps](#verify-the-connection-and-identity-credentials) after each update.

Before you delete the old UAMI resource, confirm that the server remains connected, the identities you're keeping are listed locally with `State` set to `Active`, and the old assignment is absent in Azure and from the local identity list.

This staged approach lets you verify replacement credentials before removing the old identity.

## Troubleshoot the preview

### UAMI onboarding is rejected

Run `azcmagent check tpm`, confirm that the TPM is ready, and include `--identity-key-store tpm` in the connect command. Check that the agent is version 1.69 or later. Missing preview parameters in help don't, by themselves, indicate that the feature is unavailable.

Confirm that the credential you use for onboarding has **Managed Identity Operator**, or equivalent permissions to read and assign identities, on each UAMI or a parent scope. This check is especially important when you onboard with a service principal or an account other than the one that created the UAMI. Permissions held by the UAMI's creator don't automatically apply to the onboarding credential. See [Required permissions](#required-permissions).

### An assigned identity has no local credential yet

Run `azcmagent show` and confirm that the agent reports `Connected`. For a UAMI added through the Azure portal, allow up to 5 minutes for replication while the agent remains connected, then rerun `azcmagent identity list`. For other identity changes, allow the next agent heartbeat to complete before rerunning the command. If the expected identity is missing or its `State` isn't `Active`, verify the assignment in the preview portal or with the GET request in [Verify the connection and identity credentials](#verify-the-connection-and-identity-credentials). If the assignment is correct but the local identity remains missing or inactive, [create an Azure support request](/azure/azure-portal/supportability/how-to-create-azure-support-request). Include the agent version, the identity-change operation, and the connection and credential status. Don't remove the server's remaining working identity while investigating.

### A deleted UAMI is still referenced

When you delete a UAMI resource, Azure removes its identity reference from the Arc-enabled server asynchronously. The reference might remain visible while cleanup is in progress.

If cleanup doesn't complete or the identity association still doesn't work, manually remove the affected identity reference from the Arc resource. If you need that identity association, re-add an existing UAMI after removing the reference, then follow [Verify the connection and identity credentials](#verify-the-connection-and-identity-credentials). You can't re-add a deleted UAMI resource; use an existing or replacement UAMI.

## Next steps

- [Assign Azure roles](/azure/role-based-access-control/role-assignments-portal) to grant the attached identity access to the resources your application needs.
- Review [managed identity concepts](/entra/identity/managed-identities-azure-resources/overview) to understand identity lifecycles and access to Azure services.
- [Acquire an access token for an attached UAMI](#acquire-an-access-token-for-a-user-assigned-identity), or, if you retained a system-assigned identity, learn how to [access Azure resources with that identity](managed-identity-authentication.md).
- Keep the [Azure Connected Machine agent up to date](manage-agent.md?tabs=windows).
