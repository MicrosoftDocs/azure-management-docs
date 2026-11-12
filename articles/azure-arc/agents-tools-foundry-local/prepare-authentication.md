---
title: Configure Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure authentication for Agents and Tools with Foundry Local deployment in Azure, including app registration, roles, and user assignments."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.custom:
  - sfi-ropc-nochange
  - sfi-image-nochange
#CustomerIntent: As a cloud administrator, I want to prepare and configure authentication for Agents and Tools with Foundry Local so that I can securely connect to and manage the chat solution.
---

# Configure authentication for Agents and Tools with Foundry Local

For your Agents and Tools with Foundry Local deployment, register an application in Microsoft Entra ID. After deployment, assign Azure roles to callers and the extension's managed identity. App roles provide fallback access for supported APIs. This article is part of the [deployment prerequisites checklist](complete-prerequisites.md) and also a prerequisite of [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md). If your environment uses disconnected operations, use the CLI. See [Install for disconnected operations on Azure Local](disconnected-operations/deploy-disconnected.md).

You might need to work with your Microsoft Entra or cloud administrator to configure authentication.

## Prerequisites

Before you begin, ensure you have:

- An active Azure subscription. If you don't have a service subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn).
-  Microsoft Entra ID  permissions:
   - Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).
   - Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application.
- Permission to create Azure role assignments at the extension scope, such as **Role Based Access Control Administrator**, **User Access Administrator**, or **Owner**.

## Register an application in Microsoft Entra ID

Create and configure an application registration for Agents and Tools with Foundry Local in your Microsoft Entra ID tenant.

1. In the [Azure portal](https://portal.azure.com/), go to **Microsoft Entra ID**.
1. Go to the appropriate tenant and select **Manage** > **App registrations**.
1. Select **New registration** to create an application registration.

	:::image type="content" source="media/prepare-authentication/new-app-registration.png" alt-text="Screenshot of the new registration option on the top of the application registration page.":::

1. Enter a name for the application, such as *agents-tools-foundry-local*.

1. Under **Supported account types**, select the **Single tenant only** option. The portal adds your tenant name, such as **Single tenant only - Contoso**. Some portal versions show this option as **Accounts in this organizational directory only (Single tenant)**. The extension accepts sign-ins and access tokens only from this tenant.

	:::image type="content" source="media/prepare-authentication/register-app.png" alt-text="Screenshot of the Register an application page with a name entered and the Single tenant only option selected under Supported account types.":::

1. Select **Register**.

1. After the application is registered, go to the registration and select **Manage** > **Authentication**.

1. Select **Add a platform** > **Single-page application**.

1. Specify your domain name (for example, `https://arcrag.contoso.com/`) as the **Redirect URIs**.

	:::image type="content" source="media/prepare-authentication/configure-application.png" alt-text="Screenshot of the single-page application page where you configure redirect URLs and more." lightbox="media/prepare-authentication/configure-application.png":::

1. Select **Configure**.

1. Select **+ Add a platform** > **Mobile and desktop applications**.
1. For **Redirect URIs**, select `https://login.microsoftonline.com/common/oauth2/nativeclient`.

1. Select **Configure**.

1. Select **Manage** > **Expose an API**. Next to **Application ID URI**, select **Add**, keep the default value `api://<application-client-id>`, and select **Save**. The commands in these articles request access tokens for this URI.

1. Select **Manage** > **Manifest**. In the `api` section, set `requestedAccessTokenVersion` to `2`, and then select **Save**. The extension accepts only version 2.0 access tokens. In the earlier Azure AD Graph manifest format, this property is named `accessTokenAcceptedVersion`.

## Assign Azure roles

Azure role-based access control (Azure RBAC) grants permissions on Azure resources. An administrator with role-assignment permission assigns a role to a user, group, service principal, or managed identity. A service principal is an application's identity in your tenant.

The APIs ask Azure Resource Manager (ARM), Azure's resource management service, to check Azure roles first. The check uses the Agents and Tools with Foundry Local **extension resource scope**. A scope is the resource where a role assignment applies. After you deploy the extension, get its resource ID:

```azurecli
az k8s-extension show \
  --resource-group "<resource-group>" \
  --cluster-name "<cluster-name>" \
  --cluster-type connectedClusters \
  --name "<extension-name>" \
  --query id -o tsv
```

Choose the roles that match each caller's tasks:

| Task | Azure role for a connected deployment | Azure role for a disconnected deployment |
|---|---|---|
| Read resource settings and list enabled agents | **Reader** | **Reader** |
| Create, change, or delete agents, connections, knowledge bases, sources, or collections | **Cognitive Services Contributor** | **Contributor** |
| Run agents, use the local chat UI, and manage your own Responses and Conversations | **Cognitive Services OpenAI User** | **Contributor** |
| Use classic chat completions or vector search | **Cognitive Services OpenAI User** | **Contributor** |

Assign more than one role when needed. For example, a connected chat user needs **Reader** to list agents and **Cognitive Services OpenAI User** to run them. In disconnected deployments, use **Reader** for read-only access and **Contributor** for chat and changes. **Contributor** and **Owner** also grant management access in connected deployments, but don't replace the data permissions in **Cognitive Services OpenAI User**.

For example, assign a role at the extension scope:

```azurecli
az role assignment create \
  --assignee-object-id "<caller-object-id>" \
  --assignee-principal-type User \
  --role "Cognitive Services OpenAI User" \
  --scope "<extension-resource-id>"
```

Replace `User` with `Group` or `ServicePrincipal` when needed. Use `ServicePrincipal` for managed identities. Use the caller's object ID, not its application (client) ID.

> [!IMPORTANT]
> Assign **Reader** at this same extension scope to the extension's system-assigned managed identity. Azure creates this identity for the extension. The extension uses it to read Azure role definitions, role assignments, and deny assignments. The Azure portal deployment doesn't create this role assignment. Without it, Responses and Conversations can return `503` with `Authorization service unavailable`. This step applies even when your model uses an API key. See [Allow the extension to check Azure roles](deploy.md#allow-the-extension-to-check-azure-roles).

Use the extension scope for least privilege. Assignments inherited from the subscription, resource group, or connected cluster also grant access. Group assignments count. An Azure role that allows collection access allows access to **every collection**; it doesn't apply a per-collection filter.

Azure role changes take time to propagate. The service also caches allowed decisions for 300 seconds and denied decisions for 30 seconds. Wait for propagation and the cache to expire before you test a changed assignment.

App-only tokens represent an application instead of a person. Tokens from service principals and managed identities don't need a `roles` claim when an Azure role allows the request. The token must still be issued for the extension's app registration.

## Create app roles for Agents and Tools with Foundry Local

App roles are permissions you define in your Microsoft Entra app registration. Supported APIs check them as a second option, or *fallback*, when Azure roles don't allow a request. Create them when you need fallback access, full agent definitions, or access through the developer portal, which still checks app roles during sign-in.

> [!IMPORTANT]
> Azure roles are the primary access model. If the Azure role check doesn't allow a request, most management and classic APIs can use `EdgeRAGDeveloper` or `EdgeRAGEndUser` as a fallback. Responses and Conversations have **no app-role fallback**. Running agents, including through the local chat UI, requires an Azure role.

You still need the `EdgeRAGDeveloper` app role to get full definitions from `GET /agents` and `GET /agents/{name}`. Callers without this app role get summaries of enabled agents, even if an Azure role allows the request.

1. In the app registration, on the left-hand side menu, under **Manage**, select **App roles**.
1. Create two app roles. One for *EdgeRAGDeveloper* and another for *EdgeRAGEndUser*. Use the appropriate values listed in the following table.

	:::image type="content" source="media/prepare-authentication/app-roles.png" alt-text="Screenshot of the two app roles created for the developer and user.":::


   | Field  | Value |
   |--------|-------|
   | Display name     |   *EdgeRAGDeveloper* or *EdgeRAGEndUser*      |
   | Allowed member types   |  User/Groups       |
   | Value    |    *EdgeRAGDeveloper* or *EdgeRAGEndUser*        |
   | Description    |  *EdgeRAGDeveloper* or *EdgeRAGEndUser*          |
   | Do you want to enable this app role? | Checked |

1. When complete, close the **App roles** page.

## Assign users or groups to roles

Next, in the Microsoft Entra ID tenant, assign users or groups to the roles you created for Agents and Tools with Foundry Local.

1. In the Microsoft Entra ID tenant, on the left-hand side menu under **Manage**, select **Enterprise applications**.
1. Search for and select the *agents-tools-foundry-local* application you created.
1. Go to **Manage** > **Properties**.
1. Disable **Assignment Required**.
1. On the left-hand side menu, select **Users and groups** > **Add user/group**.
1. Select users and groups and assign the **EdgeRAGDeveloper** or **EdgeRAGEndUser** role as appropriate. Assign both roles to the developers working on the chat solution.
1. When complete, close the **Users and groups** page.

## Create app roles for collection access

This section applies only when a classic query uses app-role fallback. In that case, `EdgeRAGEndUser` alone doesn't grant query access. Each end user also needs an app role whose **Value** exactly matches each collection name they can query. `EdgeRAGDeveloper` grants access to all collections.

Callers allowed by an Azure role can access every collection and don't need collection app roles. Collection app roles don't grant access to Responses or Conversations.

Start with the default `edgeragapp` collection that's created when you deploy the extension. If you don't specify a collection during ingestion, data is ingested into `edgeragapp`. After deployment, create and assign one matching app role for each additional collection you add.

For app-role fallback, create and assign collection app roles before end users query data. Data ingestion succeeds without these roles, but queries fail with `403 Forbidden`.

### Create a collection app role

1. In the [Azure portal](https://portal.azure.com), go to **Microsoft Entra ID** > **App registrations** and select your Agents and Tools with Foundry Local app registration.
1. On the left-hand side menu, under **Manage**, select **App roles**.
1. Select **Create app role**.
1. Create an app role for the default `edgeragapp` collection or a collection you create after you install the extension. Use the following values:

   | Field  | Value |
   |--------|-------|
    | Display name | A descriptive name (for example, *Default Collection* or *Finance Docs Collection*) |
   | Allowed member types | **Users/Groups** |
    | Value | The **exact collection name**. For the default collection, use `edgeragapp`. This value must match the collection name used in the API. |
    | Description | Description of the collection access (for example, *Grants query access to the default edgeragapp collection*) |
   | Do you want to enable this app role? | Checked |

1. Select **Apply**.
1. After deployment, if you create additional collections, repeat these steps for each collection (for example, `finance-docs`).

### Assign users to collection app roles

1. In the Microsoft Entra ID tenant, on the left-hand side menu under **Manage**, select **Enterprise applications**.
1. Search for and select the *agents-tools-foundry-local* application.
1. On the left-hand side menu, select **Users and groups** > **Add user/group**.
1. Select the users or groups who need access to the collection.
1. Select the collection app role (for example, `edgeragapp` for the default collection).
1. Complete the assignment.

### Example

The following table shows an example of collection app role assignments:

| Collection name | App role value | Assigned users |
|---|---|---|
| `edgeragapp` | `edgeragapp` | All chat end users |
| `finance-docs` | `finance-docs` | Finance team |
| `hr-data` | `hr-data` | HR team |

When a query uses app-role fallback, an end user querying `edgeragapp` must have the `edgeragapp` app role in their token. The same rule applies to each additional collection (for example, `finance-docs`). Otherwise, the request is denied with `403 Forbidden`.

For more information about Azure roles and app-role collection access, see [Collections and RBAC](collections-overview.md#collections-and-rbac).

## (Optional) Register a Foundry Local application

If you use Foundry Local as your model endpoint, you need a second app registration to identify the Foundry inference service. This registration provides the `foundryClientId` value used for managed identity token scope (`<client_id>/.default`).

| App registration | Purpose | Key value |
|---|---|---|
| **Agents and Tools app** | Identifies the Agents and Tools with Foundry Local extension for Microsoft Entra authentication (JWT validation on external endpoints). | `auth.clientId` - passed to the Agents and Tools with Foundry Local extension. |
| **Foundry Local app** | Identifies the Foundry inference service. Used as the managed identity token audience scope. | `foundryClientId` - passed to the inference operator and Agents and Tools with Foundry Local extension. |

For instructions on creating the Foundry Local app registration, see [Configure authentication for Foundry Local](/azure/azure-sovereign-clouds/private/foundry-local/how-to-configure-authentication).

> [!IMPORTANT]
> **For the indexed-source endpoint, `foundryClientId` and `byom-api-key` are mutually exclusive.** When `foundryClientId` is set, the indexed-source workloads use managed identity token authentication exclusively. No API key secret is needed, and if one exists it's ignored. When `foundryClientId` isn't set, those workloads require a `byom-api-key` Kubernetes secret. These workloads serve classic chat in the developer portal, which is deprecated. Agent model connections choose their own authentication type, but agents that call Foundry Local on this cluster still need `foundryClientId` to trust the Foundry Local certificate.

After you deploy the extension, configure Foundry Local managed identity role assignments. For steps, see [Configure Foundry Local inference authentication for Agents and Tools with Foundry Local](configure-foundry-inference-authentication.md).

## (Optional) Get app and tenant IDs

If you plan to use the [quickstart](quickstart-edge-rag.md) or want to deploy the extension by using the command line, get the application ID for the registration you created and the tenant ID.

1. In the [Azure portal](https://portal.azure.com/), search for **app registration**.
1. Select the registration that you created for the extension.
1. Copy the **Application (client) ID** and **Directory (tenant) ID**.
1. Paste the values to an app like Windows Notepad to use later.


## Next step

> [!div class="nextstepaction"]
> [Install networking and observability components](prepare-networking-observability.md)
