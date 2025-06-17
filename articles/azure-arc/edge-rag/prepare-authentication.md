---
title: Configure Authentication for Edge RAG Deployment
description: "Learn how to configure authentication for Edge RAG deployment in Azure, including app registration, roles, and user assignments."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/21/2025
ai-usage: ai-assisted
#CustomerIntent: As a cloud administrator, I want to prepare and configure authentication for Edge RAG so that I can securely connect to and manage the chat solution.
---

# Configure authentication for an Edge RAG Deployment

In this article, configure authentication for your Edge RAG deployment by registering an application, creating app roles, and assigning users or groups. This article is part of the deployment prerequisites checklist.

## Configure authentication for the chat solution

Configure and control authentication to the Edge RAG for AI application developers and for end users of the chat endpoint. In this section, you create an application registration, create app roles, and assign users or groups to those roles. 

You might need to work with your Microsoft Entra or cloud administrator to configure authentication.

1. In the Azure portal, go to **Microsoft Entra ID**.
1. Go to the appropriate tenant and select **Manage** > **App registrations**.
1. Select **New registration** to create an application registration.

	:::image type="content" source="media/complete-prerequisites/new-app-registration.png" alt-text="Screenshot that shows the new registration option on the top of the application registration page.":::

1. Enter *EdgeRAG* for **Name**.

1. Select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant**).

1. Select **Register**.

	:::image type="content" source="media/complete-prerequisites/register-app.png" alt-text="Screenshot that shows the fields on the register an application page where you add an application name and select supported account types.":::

1. After the application is registered, go to the registration and select **Manage** > **Authentication**.

1. Select **Add a platform** > **Single-page application**.

1. Specify your domain name appended with */authorizing* (for example, `arcrag.contoso.com/authorizing`)  as the **Redirect URIs**.

	:::image type="content" source="media/complete-prerequisites/configure-application.png" alt-text="Screenshot that shows the single-page application page where you configure redirect URLs and more." lightbox="media/complete-prerequisites/configure-application.png":::

1. Select **Configure**.
1. For **Supported account types**, select **Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)**.

	:::image type="content" source="media/complete-prerequisites/supported-account-types.png" alt-text="Screenshot that shows the options for the supported account types with the last option selected.":::

1. Select **+ Add a platform** > **Mobile and desktop applications**.
1. For **Redirect URIs**, select `https://login.microsoftonline.com/common/oauth2/nativeclient`.

1. Select **Configure**.
1. On the left-hand side menu, under **Manage**, select **App roles**.
1. Create two app roles. One for *EdgeRAGDeveloper* and another for *EdgeRAGEndUser*. Use the appropriate values listed in the table that follows the image.

	:::image type="content" source="media/complete-prerequisites/app-roles.png" alt-text="Screenshot that shows the two app roles created for the developer and user.":::


   | Field  | Value |
   |--------|-------|
   | Display name     |   *EdgeRAGDeveloper* or *EdgeRAGEndUser*      |
   | Allowed member types   |  User/Groups       |
   | Value    |    *EdgeRAGDeveloper* or *EdgeRAGEndUser*        |
   | Description    |  *EdgeRAGDeveloper* or *EdgeRAGEndUser*          |
   | Do you want to enable this app role? | Checked |

1. When complete, close the **App roles** page.
1. To assign users or groups to the role you created, on the tenant's left-hand side menu, under **Manage**, select **Enterprise applications**.
1. Search for and select the *EdgeRag* application you created.
1. Go to **Manage** > **Properties**.
1. Disable **Assignment Required**.
1. On the left-hand side menu, select **Users and groups** > **Add user/group**.
1. Select users and/or groups and assign **EdgeRAGDeveloper** or **EdgeRAGEndUser** role as appropriate.

## Next step

> [!div class="nextstepaction"]
> [Install networking and observability components for Edge RAG deployment](prepare-networking-observability.md)