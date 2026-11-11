---
title: Connect Microsoft 365 Local to Agents and Tools with Foundry Local extension
description: "Create managed SharePoint and Exchange knowledge sources, check readiness, and use them in Agentic Retrieval through a knowledge base."
author: Lea1702
ms.author: leabenero
ms.topic: how-to
ms.date: 11/09/2026
ms.service: azure-arc
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
ms.update-cycle: 180-days
ai-usage: ai-assisted
---

# Connect Microsoft 365 Local to Agents and Tools with Foundry Local extension

Microsoft 365 Local connects Agentic Retrieval in Agents and Tools with Foundry Local to on-premises SharePoint Server and Exchange Server. *Agentic Retrieval* is the extension component that finds relevant information in your knowledge sources and returns it with citations. You create SharePoint and Exchange knowledge sources, and then add them to a *knowledge base*, which groups knowledge sources for retrieval. Agents that use the knowledge base retrieve current documents, email, and calendar information from the source systems, using the signed-in user's permissions.

> [!IMPORTANT]
> Microsoft 365 Local is in limited preview for approved customers. To request access, complete the [preview registration form](https://aka.ms/AgenticRetrievalM365LocalPuP).
>
> Microsoft 365 Local is supported only with Agentic Retrieval, through knowledge sources that you use in a knowledge base. It isn't supported as a general Agent Service integration, so don't add SharePoint or Exchange tools directly to an agent.
>
> After your access is approved, complete [Preview entitlement and offline installation](prepare-microsoft-365-local.md#preview-entitlement-and-offline-installation) for the target subscription. Use version **1.0.0 or later** of the Agents and Tools with Foundry Local extension; see [Supported releases and server versions](prepare-microsoft-365-local.md#supported-releases-and-server-versions) and [Release notes](release-notes.md).

## How provisioning works

Follow the [deployment overview](deploy-overview.md) and [installation instructions](deploy.md) to prepare the platform once. After completing the [Microsoft 365 Local administrator prerequisites](prepare-microsoft-365-local.md), create each managed MCP knowledge source through the Knowledge Sources API:

```http
POST /knowledgesources
```

> [!NOTE]
> These examples target the `/knowledgesources` route in version **1.0.0 or later** of the Agents and Tools with Foundry Local extension. To confirm that your installed release includes this API and its custom resource definition, see [Supported releases and server versions](prepare-microsoft-365-local.md#supported-releases-and-server-versions).

The service creates a Kubernetes-backed knowledge source. The platform reconciles its MCP workload and managed Connection asynchronously. You don't supply a private MCP endpoint, deploy an MCP server yourself, or reinstall the extension for each source.

> [!IMPORTANT]
> Each provider that you use needs an administrator-prepared RWX cache claim, set in `sharepointMcp.cache.claimName` or `exchangeMcp.cache.claimName` during installation or an extension update. The extension doesn't create these claims. If the provider's cache setting is empty, source creation returns `400 Bad Request` with a message that names that setting, and the source isn't created. A missing or unsuitable claim also returns `400 Bad Request`; a claim that's still binding returns `503 Service Unavailable`. Creating a PVC or knowledge source doesn't attach storage to existing workloads. See [Shared file-cache requirements](prepare-microsoft-365-local.md#shared-file-cache-requirements).

| Responsibility | When | What to prepare |
|---|---|---|
| Platform administrator | Install or upgrade the platform | Managed-source capability, identity, TLS, networking, admission controls, model access, and shared cache storage |
| Identity and source-system administrators | Before source creation | AD FS clients, callbacks, CORS, claims, source permissions, SharePoint farm trust, and administrator-published SharePoint configuration |
| Application developer | After the prerequisites are ready | Create sources through the API, poll readiness, add them to a knowledge base, then add the knowledge base to an agent |
| End user | When using a configured agent | Sign in to the applicable connector |

Complete [Microsoft 365 Local administrator prerequisites](prepare-microsoft-365-local.md) before following this article. Those prerequisites apply to connected and disconnected environments.

## Supported capabilities

| Capability | SharePoint Server | Exchange Server |
|---|---|---|
| Live retrieval | Search, list libraries and documents, read metadata and supported document content | Search and read mail, attachments, calendar events, and supported scheduling information |
| Source authentication | AD FS user token plus certificate-signed high-trust user-and-add-in authentication (s2s_user) | Delegated AD FS user token accepted by EWS (delegated_connector) |

Knowledge-base retrieval uses read-only tools. It doesn't create, change, send, or delete content in SharePoint or Exchange. Retrieval doesn't bypass SharePoint or Exchange permissions: each user sees only the content that the source system allows.

## Before you begin

You need:

- Version **1.0.0 or later** of the Agents and Tools with Foundry Local extension with managed SharePoint and Exchange source support. See [Supported releases and server versions](prepare-microsoft-365-local.md#supported-releases-and-server-versions).
- Completed [administrator prerequisites](prepare-microsoft-365-local.md), including the `m365-preview-entitlement` Secret from [Preview entitlement and offline installation](prepare-microsoft-365-local.md#preview-entitlement-and-offline-installation).
- The existing Agents and Tools with Foundry Local extension application namespace (`arc-rag` by default), confirmed by the platform administrator, with [shared cache storage configured and verified](prepare-microsoft-365-local.md#configure-shared-cache-storage-during-installation-or-update).
- The gateway HTTPS origin, such as <https://agentic.contoso.com>.
- An access token for the Agents and Tools with Foundry Local extension application audience. Assign **Cognitive Services Contributor** at the extension scope, or **Contributor** in disconnected deployments. The `EdgeRAGDeveloper` app role also works as a fallback. See [Assign Azure roles](prepare-authentication.md#assign-azure-roles).
- A model connection for the knowledge base. See [Manage knowledge bases](knowledge-bases-guide.md#prerequisites).
- For SharePoint, completed [farm trust and user preparation](prepare-microsoft-365-local.md#5-prepare-sharepoint-farm-trust-and-users) and the exact immutable configuration binding name supplied by your administrator.
- For Exchange, completed [EWS OAuth trust and mailbox preparation](prepare-microsoft-365-local.md#6-prepare-exchange-ews-and-users) and [authenticated EWS OAuth validation](prepare-microsoft-365-local.md#validate-ews-oauth-before-developer-handoff), plus the approved EWS endpoint, public-client login settings, and TLS trust selection.
- Optional: for clickable email and calendar citations, get the approved Outlook on the web (OWA) base URL from the Exchange administrator, such as `https://mail.contoso.com/owa`. Users must be able to reach this hostname, trust its HTTPS certificate, and sign in to their mailbox in OWA. You set this URL once for each Exchange source in `exchange_parameters.owa_url`. It isn't a user credential, an app registration, or an extension setting, and the EWS endpoint alone doesn't set it.
- `curl` and `jq` for the command-line examples.

The examples use Bash and fictitious Contoso names. Replace them with your environment's approved values. Don't enable shell tracing or save access tokens in documentation, source control, or support logs.

```bash
set -euo pipefail
ENDPOINT="https://agentic.contoso.com"
TOKEN="$(az account get-access-token \
  --resource "api://<application-client-id>" \
  --query accessToken -o tsv)"
```

`ENDPOINT` is the gateway origin. Knowledge-source requests use `/knowledgesources`.

In a disconnected environment, complete the [disconnected environment checklist](prepare-microsoft-365-local.md#9-disconnected-environment-checklist), sign Azure CLI in to the local cloud, and use that environment's application audience. The primary application token and the user's AD FS connector token are different credentials.

## Create a SharePoint knowledge source

Your administrator publishes a binding that identifies the SharePoint Search entry point, farm trust, token validation, browser login, and TLS configuration. You reference that binding; you don't submit the PFX, its password, a user SID, or an access token.

Save this request as sharepoint-source.json:

```json
{
  "name": "corp-sharepoint",
  "kind": "sharepoint",
  "auth_type": "s2s_user",
  "description": "Corporate SharePoint documents",
  "sharepoint_parameters": {
    "credential_binding_ref": "sp-auth-corp-sharepoint.0000000000000001"
  }
}
```

Create the source:

```bash
curl --fail-with-body --silent --show-error \
  --request POST \
  --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/json" \
  --data-binary @sharepoint-source.json \
  "$ENDPOINT/knowledgesources"
```

The API returns 202 Accepted after validating and pinning the administrator binding. This response isn't proof of successful access to the farm. It doesn't perform an end-user sign-in or validate that a particular user can read a document.

One source represents a Search entry point with one authentication context, not a separate deployment for every user or document library. The configured URL isn't a security boundary limiting results to that one web; SharePoint applies the user's permissions.

## Create an Exchange knowledge source

Save this request as exchange-source.json:

```json
{
  "name": "corp-exchange",
  "kind": "exchange",
  "auth_type": "delegated_connector",
  "description": "Corporate email and calendar",
  "exchange_parameters": {
    "ews_url": "https://mail.contoso.com/EWS/Exchange.asmx",
    "owa_url": "https://mail.contoso.com/owa",
    "login": {
      "authority": "https://fs.contoso.com/adfs",
      "client_id": "22223333-cccc-4444-dddd-5555eeee6666",
      "redirect_uri": "https://agentic.contoso.com/chat/connector-callback.html",
      "resource": "https://mail.contoso.com",
      "scope": "openid profile EWS.AccessAsUser.All"
    },
    "tls": {
      "ca_source": "source",
      "ca_config_map_name": "ex-ca-corp-exchange.0000000000000001"
    }
  }
}
```

This example uses the source-specific CA ConfigMap prepared by the administrator. If the endpoint is trusted through the installation's platform trust, use `"tls": {"ca_source": "platform"}` and omit `ca_config_map_name`.

`owa_url` is optional. It's the browser-facing Outlook on the web base URL for this source, and it's separate from the EWS endpoint. Use `https://` with no query string or fragment. When you set it, email and calendar citations include a link that opens the item in the browser. If you omit it, Exchange retrieval still works, but citations don't include a browser link. To open a link, users need network access to the OWA host, a trusted HTTPS certificate, and their own OWA sign-in to the mailbox. To remove the URL later, update the source and set `owa_url` to `null`.

`owa_url` requires the extension release listed under **Exchange citation links** in the [November 2026 release notes](release-notes.md#november-2026), or a later release. Earlier releases reject the field. Before you set it, update the extension to that release. The update installs both the updated KnowledgeSource custom resource definition and the operator that applies the setting. Citations that were already stored don't change; retrieve the items again to get links.

The API stores `exchange_parameters.owa_url` as `spec.exchange.owaUrl` on the managed KnowledgeSource resource. Keep creating and updating sources through the API; don't edit the resource directly. Earlier releases used one extension-wide setting, `exchangeMcp.owaUrl`. That value isn't copied to managed Exchange sources. Set `owa_url` on each Exchange source instead.

```bash
curl --fail-with-body --silent --show-error \
  --request POST \
  --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/json" \
  --data-binary @exchange-source.json \
  "$ENDPOINT/knowledgesources"
```

The API returns `202 Accepted`. Wait for reconciliation before you add the source to a knowledge base. This flow uses on-premises Exchange EWS and AD FS; it isn't an Exchange Online or Microsoft Graph setup procedure.

## Add more SharePoint or Exchange servers

Repeat source creation for each additional approved endpoint or authentication context. Each successful POST creates a separate KnowledgeSource custom resource, and the platform provisions the source's MCP workload and managed Connection. Use the API rather than directly applying a custom-resource manifest, so source validation and identity pinning are preserved.

All sources use the existing Agents and Tools with Foundry Local extension application namespace. You don't reinstall the extension, create a new namespace per source, or replace the first source's configuration.

### Another SharePoint farm or Search entry point

- Complete the farm, AD FS, and user prerequisites for the additional farm. For a different farm trust, register its signing certificate, issuer, realm, and app principal separately.
- Publish a distinct binding configuration in the existing application namespace, such as sp-auth-branch-sharepoint.0000000000000001, together with its matching signing Secret and optional CA ConfigMap.
- Set that binding's configurationName to branch-sharepoint, its querySiteUrl to the approved additional Search entry point, and its farm/login fields to the corresponding administrator-approved values. Don't point the original farm's binding at an unrelated server.
- Save the following as additional-sharepoint-source.json:

```json
{
  "name": "branch-sharepoint",
  "kind": "sharepoint",
  "auth_type": "s2s_user",
  "description": "Branch SharePoint documents",
  "sharepoint_parameters": {
    "credential_binding_ref": "sp-auth-branch-sharepoint.0000000000000001"
  }
}
```

```bash
curl --fail-with-body --silent --show-error \
  --request POST \
  --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/json" \
  --data-binary @additional-sharepoint-source.json \
  "$ENDPOINT/knowledgesources"
```

Create a new source when you need a separate Search entry point or authentication context, not for every user or library. Multiple content servers or webs can already be covered by one farm's Search entry point; a source URL isn't an authorization boundary. SharePoint still enforces each user's permissions.

### Another Exchange EWS endpoint

- Have the Exchange and AD FS administrators approve the additional HTTPS EWS endpoint, audience, public client, scopes, callback, and mailbox permissions.
- If source-specific CA trust is needed, publish ex-ca-branch-exchange.0000000000000001 in the existing application namespace.
- Save the following as additional-exchange-source.json, replacing the generic values with the approved values for that endpoint:

```json
{
  "name": "branch-exchange",
  "kind": "exchange",
  "auth_type": "delegated_connector",
  "description": "Branch email and calendar",
  "exchange_parameters": {
    "ews_url": "https://mail-branch.contoso.com/EWS/Exchange.asmx",
    "owa_url": "https://mail-branch.contoso.com/owa",
    "login": {
      "authority": "https://fs-branch.contoso.com/adfs",
      "client_id": "33334444-dddd-5555-eeee-6666ffff7777",
      "redirect_uri": "https://agentic.contoso.com/chat/connector-callback.html",
      "resource": "https://mail-branch.contoso.com",
      "scope": "openid profile EWS.AccessAsUser.All"
    },
    "tls": {
      "ca_source": "source",
      "ca_config_map_name": "ex-ca-branch-exchange.0000000000000001"
    }
  }
}
```

```bash
curl --fail-with-body --silent --show-error \
  --request POST \
  --header "Authorization: Bearer $TOKEN" \
  --header "Content-Type: application/json" \
  --data-binary @additional-exchange-source.json \
  "$ENDPOINT/knowledgesources"
```

Use platform trust instead when appropriate, as in the first-source procedure. Set `owa_url` to the OWA address that the administrator approved for this endpoint. Don't reuse the first server's OWA URL. The setting is optional, as described in [Create an Exchange knowledge source](#create-an-exchange-knowledge-source). The EWS URL and login authority of an existing Exchange source are immutable: a different endpoint or authority requires a new source. A different mailbox accessed through the same endpoint doesn't require a source per user.

After each create, poll the new source by its own name and wait for current-generation readiness. Add the source to a knowledge base by its own name. Users sign in to each source's connector separately.

## Check current-generation readiness

Read each source by its name, not by a legacy database ID:

```bash
curl --fail-with-body --silent --show-error \
  --header "Authorization: Bearer $TOKEN" \
  "$ENDPOINT/knowledgesources/corp-sharepoint" \
  | jq '{name, provisioning_state, generation, observed_generation,
         provisioning_message, connection}'
curl --fail-with-body --silent --show-error \
  --header "Authorization: Bearer $TOKEN" \
  "$ENDPOINT/knowledgesources/corp-exchange" \
  | jq '{name, provisioning_state, generation, observed_generation, connection_id}'
```

| State | Meaning | Next step |
|---|---|---|
| Provisioning | The requested configuration isn't ready yet | Poll GET again; don't create a duplicate source |
| Succeeded, with matching generation and observed_generation | Reconciliation completed for the current configuration | Verify the managed Connection and test an authenticated end-user request |
| Failed | Provisioning or validation failed | Inspect the returned diagnostic information and ask the administrator to correct the prerequisite or configuration |

Source readiness doesn't prove that a user is licensed, has mailbox access, or has permissions on a SharePoint document. End-to-end testing remains required.

SharePoint responses return `connection.name` and `connection.id`. Exchange's managed Connection is named ex-\<source UID\>, using the `uid` returned by GET. The platform creates and manages this source-owned Connection, and it might not exist immediately after create. Don't create a Connection yourself to bypass that wait. To use a source, you reference it by name in a knowledge base, not by its Connection.

Don't confuse the Exchange `connection_id`, which is used for the user's delegated connector sign-in, with the managed Connection. Keep the values returned for the same source together; don't substitute a static exchange or sharepoint identifier.

## Use the sources through a knowledge base

<a id="connect-m365-built-in-tools"></a>

You use Microsoft 365 Local sources through Agentic Retrieval. Add the ready sources to a knowledge base, and then add the knowledge base to an agent. When a user asks a question, the agent calls the knowledge base. The knowledge base then retrieves information from SharePoint and Exchange with that user's connector sign-in, and returns it with citations. This section covers only the steps specific to Microsoft 365 Local. For the full procedure, see [Manage knowledge bases](knowledge-bases-guide.md).

> [!NOTE]
> Don't add the SharePoint or Exchange built-in tools (tool types `sharepoint_local` and `exchange_local`) directly to an agent version, and don't register a generic MCP Connection that points to a managed source. In this preview, Microsoft 365 Local is supported only through a knowledge base.

Complete source creation and current-generation readiness checks before you continue.

### 1. Add the sources to a knowledge base

Save the following JSON as `m365-knowledge-base.json`. Replace `local-model` with your model connection name, and replace `gpt-oss-20b` with the model name in that connection. List only the ready sources that this knowledge base should search.

```json
{
  "name": "workplace",
  "description": "Corporate SharePoint documents, email, and calendar information.",
  "knowledgeSources": [
    { "name": "corp-sharepoint" },
    { "name": "corp-exchange" }
  ],
  "models": [
    {
      "kind": "azureOpenAI",
      "azureOpenAIParameters": {
        "resourceUri": "connection://local-model",
        "deploymentId": "gpt-oss-20b"
      }
    }
  ]
}
```

```bash
curl --fail-with-body --silent --show-error \
  --request PUT \
  --header "Authorization: ******" \
  --header "Content-Type: application/json" \
  --header "If-None-Match: *" \
  --data-binary @m365-knowledge-base.json \
  "$ENDPOINT/knowledgebases('workplace')?api-version=2026-08-01-preview"
```

For multiple SharePoint farms or EWS endpoints, list each source by its own name. The agent uses the knowledge base's `description` to decide when to search, so make it clear and specific.

### 2. Add the knowledge base to an agent

The platform creates an MCP connection named `kb-<knowledge-base-name>` for the knowledge base. Add that connection to a new agent version as an `mcp` tool. Replace `local-model/gpt-oss-20b` with your model connection and model name:

```bash
curl --fail-with-body --silent --show-error \
  --request POST \
  --header "Authorization: ******" \
  --header "Content-Type: application/json" \
  --data '{
    "definition": {
      "kind": "prompt",
      "model": "local-model/gpt-oss-20b",
      "instructions": "Answer questions about corporate documents, email, and calendars. Search the workplace knowledge base first, and cite your sources.",
      "tools": [
        {
          "type": "mcp",
          "server_label": "workplace",
          "project_connection_id": "kb-workplace"
        }
      ]
    }
  }' \
  "$ENDPOINT/agents/m365-assistant/versions"
```

For a knowledge base tool, set only `type`, `server_label`, and `project_connection_id`; the platform fills in the rest. The knowledge base must exist and be ready when you create the version. If you're updating an existing agent, add this tool to its complete definition and create a new version.

### 3. Sign in and test

Use the agent version in the local chat UI or with the Responses API. Sign in to each source connector as the intended user. Verify a permitted read and an access-denied case. A missing or expired token must request source-specific sign-in instead of using another source's credentials.

For multiple sources, each user signs in to each source's connector separately. Signing in to one source doesn't authorize fallback to a different source after a failure.

### End-user validation

- Open the local chat UI at the final trusted HTTPS hostname.
- Select the configured agent and sign in to each required AD FS connector.
- Test a SharePoint read and an Exchange read with two users who have different source permissions.
- Verify that each user receives only authorized data and that source citations are usable.
- Confirm that missing or expired connector credentials require sign-in again rather than falling back to a shared account.

## Update or rotate configuration

Read the source immediately before changing it. Both managed kinds require the opaque `resource_version` from that `GET` as `expected_resource_version`, including for a description-only change.

For example, after the administrator publishes a new SharePoint binding revision:

```json
{
  "expected_resource_version": "<resource-version-from-get>",
  "sharepoint_parameters": {
    "credential_binding_ref": "sp-auth-corp-sharepoint.0000000000000002"
  }
}
```

Send this to `PATCH /knowledgesources/corp-sharepoint`. A changed SharePoint configuration returns 202; description-only and unchanged-binding updates return 200. Poll current-generation readiness again before use.

For Exchange, supply a complete `exchange_parameters` object when changing its mutable login or TLS settings. Its EWS URL, login authority, and authentication type can't be changed in place. Create a new source for a different endpoint or authority.

On `409 Conflict`, reread the source and review the desired change before submitting another request. On `503 Service Unavailable` after a mutation, the write might already have reached storage: read current state before retrying. Don't blindly replay mutations.

Deleting and recreating a source produces a new identity and a new managed Connection. Keep the same source for a supported credential/configuration rotation.

## Delete a source

First remove the source from the knowledge bases that reference it. Read the source to obtain current preconditions.

For SharePoint:

```bash
curl --fail-with-body --silent --show-error \
  --request DELETE \
  --header "Authorization: Bearer $TOKEN" \
  --get \
  --data-urlencode "expected_source_uid=<source-uid-from-get>" \
  --data-urlencode "expected_resource_version=<resource-version-from-get>" \
  "$ENDPOINT/knowledgesources/corp-sharepoint"
```

For Exchange, include its required `expected_uid`:

```bash
curl --fail-with-body --silent --show-error \
  --request DELETE \
  --header "Authorization: Bearer $TOKEN" \
  --get \
  --data-urlencode "expected_uid=<uid-from-get>" \
  "$ENDPOINT/knowledgesources/corp-exchange"
```

`204 No Content` acknowledges deletion; workload and cache cleanup can continue asynchronously. Don't remove finalizers or force cleanup after a timeout. Administrator-published SharePoint bindings, signing Secrets, and CA material remain administrator-owned and aren't deleted with the source.

## Troubleshooting

| Symptom | Check |
|---|---|
| Source creation returns 403 with `M365 Local is not enabled for this subscription.` | Ask the administrator to install a valid `m365-preview-entitlement` Secret for this subscription |
| Source creation is denied for another reason | Confirm the managed-source capability, developer role, and supported API version |
| Source creation returns 400 or 503 about the cache | Ask the administrator to set the provider's `cache.claimName` and prepare a `Bound` RWX claim; see [Shared file-cache requirements](prepare-microsoft-365-local.md#shared-file-cache-requirements) |
| SharePoint binding is rejected | Confirm the exact revision-qualified name, immutable ConfigMap, strict JSON fields, publication permission, and matching resource names |
| Source remains Provisioning | Check workload scheduling, available capacity, RWX claim binding, admission prerequisites, and endpoint DNS/TLS |
| Connector sign-in fails | Check the exact callback, public-client registration, requested/granted scopes, AD FS CORS, browser reachability, and client certificate trust |
| SharePoint token is rejected | Check the exact issuer/audience, scalar SID claim, requested scope, HTTPS JWKS reachability, and issuing CA trust |
| SharePoint read returns access denied | Check farm high trust, app-principal permissions, User Profile, Windows-claims identity, and the user's permissions |
| Exchange read is unauthorized | Check EWS OAuth trust, audience/scope, the scalar upn claim, mailbox enablement, and user permissions |
| Source update returns 409 | Reread current resource_version; don't reuse a stale mutation |
| Knowledge base can't retrieve from a new source | Wait for current-generation readiness and managed Connection reconciliation |

## Related content

- [Microsoft 365 Local administrator prerequisites](prepare-microsoft-365-local.md)
- [Agentic Retrieval overview](knowledge-layer-overview.md)
- [Manage knowledge bases](knowledge-bases-guide.md)
- [Agents and Tools with Foundry Local extension deployment overview](deploy-overview.md)
- [Known issues](known-issues.md)
