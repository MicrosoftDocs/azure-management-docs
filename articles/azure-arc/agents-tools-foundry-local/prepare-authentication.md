---
title: Configure Authentication for Agents and Tools with Foundry Local
description: "Learn how to configure authentication for Agents and Tools with Foundry Local deployment in Azure, including app registration, roles, and user assignments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 5/18/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to prepare and configure authentication for Agents and Tools with Foundry Local so that I can securely connect to and manage the chat solution.
---

# Configure authentication for Agents and Tools with Foundry Local

For your Agents and Tools with Foundry Local deployment, register an application, create app roles, and assign users or groups in Microsoft Entra ID. This article is part of the [deployment prerequisites checklist](complete-prerequisites.md) and also a prerequisite of [Quickstart: Install Agents and Tools with Foundry Local](quickstart-edge-rag.md).

You might need to work with your Microsoft Entra or cloud administrator to configure authentication.

[!INCLUDE [preview-notice](includes/preview-notice.md)]

## Prerequisites

Before you begin, make sure you have:

- An active Azure subscription. If you don't have a service subscription, create a [free account](https://azure.microsoft.com/pricing/purchase-options/azure-account?cid=msft_learn) before you begin.
-  Microsoft Entra ID  permissions:
   - Permissions to create a Microsoft Enterprise Entra [application](/entra/identity/enterprise-apps/add-application-portal).
   - Ability to add new or existing Microsoft Entra [users and groups](/entra/identity/enterprise-apps/add-application-portal-assign-users) to the application.

## Register an application in Microsoft Entra ID

Create and configure an application registration for Agents and Tools with Foundry Local in your Microsoft Entra ID tenant.

1. In the [Azure portal](https://portal.azure.com/), go to **Microsoft Entra ID**.
1. Go to the appropriate tenant and select **Manage** > **App registrations**.
1. Select **New registration** to create an application registration.

	:::image type="content" source="media/prepare-authentication/new-app-registration.png" alt-text="Screenshot that shows the new registration option on the top of the application registration page.":::

1. Enter *EdgeRAG* for **Name**.

1. Select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant**).

1. Select **Register**.

	:::image type="content" source="media/prepare-authentication/register-app.png" alt-text="Screenshot that shows the fields on the register an application page where you add an application name and select supported account types.":::

1. After the application is registered, go to the registration and select **Manage** > **Authentication**.

1. Select **Add a platform** > **Single-page application**.

1. Specify your domain name (for example, `https://arcrag.contoso.com/`)  as the **Redirect URIs**.

	:::image type="content" source="media/prepare-authentication/configure-application.png" alt-text="Screenshot that shows the single-page application page where you configure redirect URLs and more." lightbox="media/prepare-authentication/configure-application.png":::

1. Select **Configure**.
1. For **Supported account types**, select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**.

	:::image type="content" source="media/prepare-authentication/supported-account-types.png" alt-text="Screenshot that shows the options for the supported account types with the last option selected.":::

1. Select **+ Add a platform** > **Mobile and desktop applications**.
1. For **Redirect URIs**, select `https://login.microsoftonline.com/common/oauth2/nativeclient`.

1. Select **Configure**.

## Create app roles for Agents and Tools with Foundry Local

Within the Agents and Tools with Foundry Local app registration, create app roles for AI application developers and end users of the chat endpoint.

1. In the app registration, on the left-hand side menu, under **Manage**, select **App roles**.
1. Create two app roles. One for *EdgeRAGDeveloper* and another for *EdgeRAGEndUser*. Use the appropriate values listed in the table that follows the image.

	:::image type="content" source="media/prepare-authentication/app-roles.png" alt-text="Screenshot that shows the two app roles created for the developer and user.":::


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
1. Search for and select the *EdgeRag* application you created.
1. Go to **Manage** > **Properties**.
1. Disable **Assignment Required**.
1. On the left-hand side menu, select **Users and groups** > **Add user/group**.
1. Select users and groups and assign the **EdgeRAGDeveloper** or **EdgeRAGEndUser** role as appropriate. Assign both roles to the developers working on the chat solution.
1. When complete, close the **Users and groups** page.

## Create app roles for collection access

The `EdgeRAGEndUser` role alone doesn't grant query access. Each end user also needs an app role whose **Value** exactly matches each collection name they can query.

Start with the default `edgeragapp` collection that's created when you deploy the extension. If you don't specify a collection during ingestion, data is ingested into `edgeragapp`. After deployment, create and assign one matching app role for each additional collection you add.

Create and assign collection app roles for `edgeragapp` and every additional collection before end users query data. Data ingestion succeeds without these roles, but queries fail with `403 Forbidden`.

### Create a collection app role

1. In the [Azure portal](https://portal.azure.com), go to **Microsoft Entra ID** > **App registrations** and select your Agents and Tools app registration.
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
1. Search for and select the *EdgeRag* application.
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

At query time, a user querying `edgeragapp` must have the `edgeragapp` app role in their token. The same rule applies to each additional collection (for example, `finance-docs`). Otherwise, the request is denied with `403 Forbidden`.

For more information about how collections use RBAC, see [Collections and RBAC](collections-overview.md#collections-and-rbac).

## (Optional) Register a Foundry Local application

If you use Foundry Local as your model endpoint, you need a second app registration to identify the Foundry inference service. This registration provides the `foundryClientId` value used for managed identity token scope (`<client_id>/.default`).

| App registration | Purpose | Key value |
|---|---|---|
| **Agents and Tools app** (EdgeRAG) | Identifies the Agents and Tools extension for Microsoft Entra authentication (JWT validation on external endpoints). | `auth.clientId` - passed to the Agents and Tools extension. |
| **Foundry Local app** | Identifies the Foundry inference service. Used as the managed identity token audience scope. | `foundryClientId` - passed to the inference operator and Agents and Tools extension. |

For instructions on creating the Foundry Local app registration, see [Configure authentication for Foundry Local](/azure/azure-sovereign-clouds/private/foundry-local/how-to-configure-authentication).

## Configure Azure RBAC role assignments

Configure Azure role assignments so that the Agents and Tools extension's managed identity can call the Foundry inference endpoint, and the Foundry operator can perform ARM RBAC validation.

Assign these roles at the **subscription or resource group level**. Azure RBAC inherits permissions from parent scopes, so a subscription-level assignment automatically covers all clusters within it. The principal IDs for the role assignments come from the Microsoft Entra ID app registrations you created earlier.

```azurecli
# Assign at subscription or resource group scope
SCOPE="/subscriptions/<subscription_id>"

# 1. Reader role for Foundry operator identity (required for ARM RBAC validation)
az role assignment create \
    --assignee-object-id <foundry_app_principal_id> \
    --role "Reader" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal

# 2. Cognitive Services OpenAI User role for Agents and Tools identity (for model inference)
az role assignment create \
    --assignee-object-id <agents_and_tools_app_principal_id> \
    --role "Cognitive Services OpenAI User" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal

# 3. Reader role for Agents and Tools identity (for managed identity token ARM RBAC checks)
az role assignment create \
    --assignee-object-id <agents_and_tools_app_principal_id> \
    --role "Reader" \
    --scope $SCOPE \
    --assignee-principal-type ServicePrincipal
```

If you prefer to follow the principle of least privilege, scope these roles to individual connected cluster resources instead. In that case, run the role assignments after each cluster deployment by using the cluster resource ID as the scope and the per-cluster extension principal IDs:

```azurecli
SCOPE=$(az connectedk8s show -g <resource_group> -n <cluster_name> --query "id" -o tsv)
FOUNDRY_PRINCIPAL_ID=$(az k8s-extension show -g <resource_group> -c <cluster_name> \
    -t connectedClusters --name inference-operator --query "identity.principalId" -o tsv)
EXTENSION_PRINCIPAL_ID=$(az k8s-extension show -g <resource_group> -c <cluster_name> \
    -t connectedClusters --name <extension_name> --query "identity.principalId" -o tsv)
```

## (Optional) Get app and tenant IDs

If you plan to use the [quickstart](quickstart-edge-rag.md) or want to deploy Agents and Tools with Foundry Local by using the command line, get the application ID for the registration you created and the tenant ID.

1. In the [Azure portal](https://portal.azure.com/), search for **app registration**.
1. Select the Agents and Tools with Foundry Local registration you created.
1. Copy the **Application (client) ID** and **Directory (tenant) ID**.
1. Paste the values to an app like Windows Notepad to use later.


## Next step

> [!div class="nextstepaction"]
> [Install networking and observability components](prepare-networking-observability.md)
