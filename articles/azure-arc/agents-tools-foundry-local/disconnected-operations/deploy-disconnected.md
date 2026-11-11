---
title: Install Agents and Tools with Foundry Local for disconnected operations
description: "Learn how to install Agents and Tools with Foundry Local on an Azure Local cluster that operates without internet connectivity."
ms.reviewer: assafakiva
ms.topic: how-to
ms.date: 11/08/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
ms.collection: ce-skilling-ai-copilot
#CustomerIntent: As a cloud administrator, I want to install Agents and Tools with Foundry Local on an Azure Local cluster that uses disconnected operations so that I can run an AI-powered chat solution in a disconnected environment.
---

# Agents and Tools with Foundry Local installation for disconnected operations

This article shows you how to install the [Agents and Tools with Foundry Local](../overview.md) extension on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview). Use this approach when you need to run an AI-powered chat solution that uses Foundry models without external internet connectivity.

In this article, you choose a model authentication method, configure Microsoft Entra sign-in, create fallback app roles, install the extension, assign Azure roles, and verify provisioning. For a standard connected deployment, see [Deploy the extension for Agents and Tools with Foundry Local](../deploy.md).

[!INCLUDE [disconnected-operations-preview-notice](../includes/disconnected-operations-preview-notice.md)]

## Prerequisites

Before you begin, verify these prerequisites for your environment. Throughout this article, **stamp** refers to your specific disconnected environment instance, including its domain and service endpoints.

### Disconnected environment preconditions

Ensure your disconnected environment is prepared with the required pack and extension type:

| Requirement | How to check |
|---|---|
| **Agents and Tools with Foundry Local expansion pack loaded** into your disconnected environment. | [Download and import the Agents and Tools with Foundry Local expansion pack](prepare-disconnected.md), and then confirm with your platform onboarding process for the **stamp** (your disconnected environment instance). |
| **Agents and Tools with Foundry Local extension type registered** (`microsoft.arc.rag`). | `az k8s-extension extension-types list --cluster-type connectedClusters --cluster-name <cluster> --resource-group <rg>` includes `microsoft.arc.rag`. |

### Tools and cluster readiness

On the disconnected management machine, ensure the following conditions are true before you begin:

| Requirement | How to check |
|---|---|
| **Tools**: PowerShell and Azure CLI signed in to your subscription on the disconnected stamp, and `kubectl` configured for the target cluster. | `az account show` returns your subscription; `kubectl get nodes` lists the cluster nodes. |
| **Foundry Local installed** with at least one model running. | `kubectl get mdep -n foundry-local-operator` shows a `ModelDeployment` with `phase=Running`. |
| **A policy-enforcing CNI** (Calico, Cilium, Azure NPM, Antrea, or kube-router) is installed. The Agents and Tools with Foundry Local extension uses NetworkPolicies by default and requires one. | `kubectl get ds -A` lists one of these CNI solutions. |

### Values to collect before installation

You need the Foundry **model deployment name** (for example, `phi4-cpu-demo`) and its **deployed model alias** (for example, `gpt-oss-20b`). Pass them at install time.

Substitute your own values for these placeholders throughout this article:

| Placeholder | Meaning | Example |
|---|---|---|
| `<rg>` | Resource group of the connected cluster | `developer` |
| `<cluster>` | Connected cluster name | `aldo-cluster` |
| `<ingress-domain>` | Ingress hostname on your stamp | `edgerag.autonomous.cloud.private` |
| `<stamp-domain>` | Your stamp's disconnected domain | `autonomous.cloud.private` |

## Choose how the extension authenticates to Foundry

Choose the authentication method that the extension uses for Foundry model calls. End-user sign-in for the extension endpoints is always enabled and configured separately in [Create the app registration](#create-the-app-registration). Compare the following options to choose the authentication method that best fits your environment.

| Mode | What it is | Trade-off |
|---|---|---|
| **Managed identity (recommended)** | The Agents and Tools with Foundry Local extension identity authenticates to Foundry through Microsoft Entra ID and ARM RBAC. Set with `--config foundryClientId=<...>`. | No API key to rotate. Requires the one-time grants in [Grant inference permission](#grant-inference-permission-managed-identity-auth-only). |
| **API key (bring your own model [BYOM])** | You pre-create a Kubernetes secret that holds Foundry's API key. | Simplest to start. The key changes every time Foundry redeploys the model, so you must refresh the secret and restart the Agents and Tools with Foundry Local pods when that happens. |

The install command differs slightly between the two. Both are shown in [Install the extension](#install-the-extension).

The `byom.*` settings and the `byom-api-key` Secret are used only by classic chat in the developer portal. Classic chat is deprecated, and its APIs accept requests only from the developer portal. Setting `foundryClientId` (or a `*.foundry-local-operator.svc.cluster.local` `byom.apiEndpoint`) also enables Foundry Local CA trust for Agent Service. Agents don't use `byom.*` or `byom-api-key` for model calls; after installation, create a [model connection](../create-manage-agents.md#create-a-model-connection) (and approve the model endpoint in `mi-destinations` when you use managed identity).

## Create the app registration

The extension requires Microsoft Entra application credentials for end-user sign-in. On a disconnected stamp there's no portal UI for this, so you create the app with the `az` CLI. Run the commands on the disconnected management machine, logged in via `az login`.

### Create the app and service principal

```powershell
az login   # authenticates against the disconnected stamp

$app = az ad app create `
  --display-name "agents-tools-foundry-local" `
  --sign-in-audience AzureADMyOrg `
  --requested-access-token-version 2 | ConvertFrom-Json
$appId = $app.appId
$tenantId = az account show --query tenantId -o tsv

az ad app update --id $appId --identifier-uris "api://$appId" | Out-Null
az ad sp create --id $appId | Out-Null

Write-Host "App ID : $appId"
Write-Host "Tenant : $tenantId"
```

Save `$appId` and `$tenantId`. You pass them as `auth.clientId` and `auth.tenantId` at install time.

### Register the sign-in redirect URIs

The disconnected portal has no UI for this, so use `az rest` against the disconnected Graph endpoint.

```powershell
# The --% stop-parsing operator is required so PowerShell/cmd don't mangle the (appId='...') parens.

$payload = @{ spa = @{ redirectUris = @(
  "https://<ingress-domain>:8443",
  "https://<ingress-domain>:8443/chat/",
  "https://<ingress-domain>",
  "https://<ingress-domain>/chat/"
) } } | ConvertTo-Json -Depth 5 -Compress
$payload | Out-File -Encoding ascii spa-body.json

# Replace <appId> with your $appId value
az --% rest --method PATCH `
  --uri "https://graph.<stamp-domain>/v1.0/applications(appId='<appId>')" `
  --headers "Content-Type=application/json" `
  --body "@spa-body.json"
```

The redirect hostnames must match the `<ingress-domain>` that you pass at install time.

### Create the app roles

Create the following app roles for fallback access. The script generates role IDs and reuses them when it assigns the roles. Save `app-roles.json` if you need to reproduce the same app registration later.

> [!IMPORTANT]
> Azure roles are the primary access model. App roles remain a fallback for most management and classic APIs, but not for Responses or Conversations. After installation, assign **Contributor** to callers who run agents or use the local chat UI. You still need `EdgeRAGDeveloper` for full definitions from agent list and get operations.

```powershell
$developerRoleId = [guid]::NewGuid().ToString()
$endUserRoleId   = [guid]::NewGuid().ToString()
$appRoleId       = [guid]::NewGuid().ToString()

$roles = @(
  @{ allowedMemberTypes = @("User"); description = "Developer (full access)"; displayName = "EdgeRAGDeveloper"; id = $developerRoleId; isEnabled = $true; value = "EdgeRAGDeveloper" },
  @{ allowedMemberTypes = @("User"); description = "Fallback read and classic query access"; displayName = "EdgeRAGEndUser"; id = $endUserRoleId; isEnabled = $true; value = "EdgeRAGEndUser" },
  @{ allowedMemberTypes = @("User"); description = "Default collection access";        displayName = "edgeragapp";       id = $appRoleId;       isEnabled = $true; value = "edgeragapp" }
) | ConvertTo-Json -Depth 5 -Compress
$roles | Out-File -Encoding ascii app-roles.json

az ad app update --id $appId --app-roles "@app-roles.json"
```

### Assign roles to your users

Assign the app roles to your users by using the following script. You can repeat these steps to grant access to additional users later.

On disconnected Microsoft Entra, `/users/{id}/appRoleAssignments` returns `Not Found`. Use `/servicePrincipals/{spObjectId}/appRoleAssignedTo` instead.

```powershell
$myObjectId  = az ad signed-in-user show --query id -o tsv
$spObjectId  = az ad sp show --id $appId --query id -o tsv

foreach ($roleId in @($developerRoleId, $endUserRoleId, $appRoleId)) {
  $body = @{ principalId = $myObjectId; resourceId = $spObjectId; appRoleId = $roleId } | ConvertTo-Json -Compress
  $body | Out-File -Encoding ascii role-body.json

  # Replace <spObjectId> with the service principal object ID from the earlier command
  az --% rest --method POST `
    --uri "https://graph.<stamp-domain>/v1.0/servicePrincipals/<spObjectId>/appRoleAssignedTo" `
    --headers "Content-Type=application/json" `
    --body "@role-body.json"
}
```

To grant access to more users later, repeat just this role-assignment step with their object IDs.

Get the **Foundry app registration's `appId`** (you pass it as `foundryClientId`):

```powershell
$foundryClientId = az ad app list `
  --display-name "FoundryOnArc-Disconnected" `
  --query "[0].appId" -o tsv
```

## Prepare for installation

Complete the following steps to prepare your environment for installation. These steps gather the values and resources needed by the install command.

### Pre-create the byom-api-key secret (API-key auth only)

Skip this step if you chose managed identity auth. Create the secret **before** you install so the install never waits on it:

```powershell
$MODEL_NAME = "phi4-cpu-demo"   # your ModelDeployment name

$BYOM_API_KEY = kubectl get secret "$MODEL_NAME-api-keys" -n foundry-local-operator -o jsonpath='{.data.primary-key}' | %{[Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($_))}

kubectl create namespace arc-rag --dry-run=client -o yaml | kubectl apply -f -
kubectl delete secret byom-api-key -n arc-rag --ignore-not-found
kubectl create secret generic byom-api-key --from-literal=BYOM_API_KEY=$BYOM_API_KEY -n arc-rag
```

### Pick the load balancer IP

The extension's ingress needs a load balancer IP from your cluster's ingress subnet, supplied at install time (it can't be baked into the pack).

1. On the disconnected management machine, find the ingress network interface card (NIC) and its subnet:

   ```powershell
   Get-NetIPAddress -AddressFamily IPv4 | Where-Object PrefixOrigin -ne 'WellKnown' | Format-Table IPAddress, PrefixLength, InterfaceAlias
   ```

   The ingress-subnet NIC's IP and prefix tell you the subnet. For example, `10.0.60.1/24` gives a host range from `.1` to `.254`.
1. Pick a free IP high in the range (avoid the `.1` gateway and the node DHCP pool). A common pattern is the **last IP of the node pool**.
1. Confirm it's free:

   ```powershell
   Test-Connection -ComputerName <candidate-IP> -Count 2 -Quiet   # expect $False
   ```

1. Pass it as `--config metallb.ipRange=<candidate-IP>/32` at install time.

## Install the extension

Pick the option that matches your [Choose how the extension authenticates to Foundry](#choose-how-the-extension-authenticates-to-foundry) choice. Install typically takes 10 to 25 minutes.

Substitute the following values: `<appId>` and `<tenantId>` from [Create the app and service principal](#create-the-app-and-service-principal); `<lb-IP>` from [Pick the load balancer IP](#pick-the-load-balancer-ip); `<modelName>` your `ModelDeployment` name; `<model-alias>` the deployed model alias (such as `gpt-oss-20b`).

### Option 1: Managed identity authentication (recommended)

Pass the Foundry application ID from the preceding command as `foundryClientId`. Complete [Grant inference permission](#grant-inference-permission-managed-identity-auth-only) after installation, when the Agents and Tools with Foundry Local extension identity exists.

```powershell
az k8s-extension create `
  --cluster-type connectedClusters `
  --cluster-name <cluster> `
  --resource-group <rg> `
  --name edgerag `
  --extension-type microsoft.arc.rag `
  --auto-upgrade-minor-version false `
  --target-namespace arc-rag `
  --config isManagedIdentityRequired=true `
  --config AgentOperationTimeoutInMinutes=30 `
  --config auth.featureEnable=true `
  --config auth.clientId=<appId> `
  --config auth.tenantId=<tenantId> `
  --config foundryClientId=$foundryClientId `
  --config networking.mode=envoy `
  --config networking.domainName=<ingress-domain> `
  --config metallb.ipRange=<lb-IP>/32 `
  --config byom.apiEndpoint=https://<modelName>.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions `
  --config byom.apiModel=<model-alias> `
  --config gpu_enabled=false
```

### Option 2: API key (BYOM) authentication

Use the same command **without** `foundryClientId`. The extension uses the `byom-api-key` secret from [Pre-create the byom-api-key secret (API-key auth only)](#pre-create-the-byom-api-key-secret-api-key-auth-only).

```powershell
az k8s-extension create `
  --cluster-type connectedClusters `
  --cluster-name <cluster> `
  --resource-group <rg> `
  --name edgerag `
  --extension-type microsoft.arc.rag `
  --auto-upgrade-minor-version false `
  --target-namespace arc-rag `
  --config isManagedIdentityRequired=true `
  --config AgentOperationTimeoutInMinutes=30 `
  --config auth.featureEnable=true `
  --config auth.clientId=<appId> `
  --config auth.tenantId=<tenantId> `
  --config networking.mode=envoy `
  --config networking.domainName=<ingress-domain> `
  --config metallb.ipRange=<lb-IP>/32 `
  --config byom.apiEndpoint=https://<modelName>.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions `
  --config byom.apiModel=<model-alias> `
  --config gpu_enabled=false
```

`gpu_enabled=false` is correct for CPU-only stamps. Set it to `true` only if your stamp has GPU nodes labeled for Agents and Tools with Foundry Local.

## Assign Azure roles

Complete this step for **both** model authentication methods. Run the commands with Azure CLI signed in to the disconnected stamp. An administrator with role-assignment permission assigns the roles.

The extension checks Azure roles at its extension resource scope through the stamp's local Azure Resource Manager (ARM) endpoint. The expansion pack configures this endpoint. The extension's system-assigned managed identity needs **Reader** at that scope to read role definitions, role assignments, and deny assignments.

Get the resource ID and identity's principal ID:

```azurecli
az k8s-extension show --cluster-type connectedClusters --cluster-name "<cluster>" --resource-group "<rg>" --name edgerag --query id -o tsv

az k8s-extension show --cluster-type connectedClusters --cluster-name "<cluster>" --resource-group "<rg>" --name edgerag --query identity.principalId -o tsv
```

Use the returned values to grant Reader:

```azurecli
az role assignment create --assignee-object-id "<extension-principal-id>" --assignee-principal-type ServicePrincipal --role "Reader" --scope "<extension-resource-id>"
```

> [!IMPORTANT]
> If this assignment or an inherited Reader assignment is missing, the ARM check is unavailable. Responses and Conversations return `503` with `Authorization service unavailable`. Other APIs depend on app-role fallback.

Assign caller roles at the same extension scope:

| Caller task | Role |
|---|---|
| Read resource settings and list enabled agent summaries | **Reader** |
| Create, change, or delete resources | **Contributor** |
| Run agents, use chat, or manage your Responses and Conversations | **Contributor** |

Disconnected deployments map data permissions to control-plane permissions, so **Reader** alone can't run chat. For example:

```azurecli
az role assignment create --assignee-object-id "<user-object-id>" --assignee-principal-type User --role "Contributor" --scope "<extension-resource-id>"
```

Use `Group` or `ServicePrincipal` instead of `User` when needed. Inherited subscription, resource-group, and cluster assignments also apply. Azure-role-authorized callers can access every collection without a per-collection app-role filter. Allow time for role propagation; the service also caches allows for 300 seconds and denies for 30 seconds.

## Grant inference permission (managed identity auth only)

Complete this separate model-access step after you install the extension and [assign Azure roles](#assign-azure-roles). Skip only this model-access step if you chose API-key authentication.

Managed identity inference uses two authorization layers:

1. Grant the connected-cluster and Foundry extension identities permission to read ARM role assignments.
1. Grant the Agents and Tools with Foundry Local extension identity inference permission on the Foundry extension resource. In disconnected deployments, Foundry Local requires **Contributor** on the Foundry extension resource for inference.

The current disconnected caller is the Agents and Tools with Foundry Local extension identity. Don't assign the inference role to the connected-cluster identity.

Set the resource scopes and principal IDs:

```powershell
$RG = "<rg>"
$CLUSTER = "<cluster>"
$EXTENSION_NAME = "edgerag"

$cluster = az connectedk8s show `
  --name $CLUSTER `
  --resource-group $RG | ConvertFrom-Json

$foundryExtensionName = az k8s-extension list `
  --cluster-type connectedClusters `
  --cluster-name $CLUSTER `
  --resource-group $RG | ConvertFrom-Json |
  Where-Object { $_.extensionType -ieq "Microsoft.Foundry" } |
  Select-Object -First 1 -ExpandProperty name

if (-not $foundryExtensionName) {
  throw "The Foundry extension wasn't found."
}

$foundryExtension = az k8s-extension show `
  --cluster-type connectedClusters `
  --cluster-name $CLUSTER `
  --resource-group $RG `
  --name $foundryExtensionName | ConvertFrom-Json

$extension = az k8s-extension show `
  --cluster-type connectedClusters `
  --cluster-name $CLUSTER `
  --resource-group $RG `
  --name $EXTENSION_NAME | ConvertFrom-Json

$clusterScope = $cluster.id
$foundryScope = $foundryExtension.id
$clusterPrincipalId = $cluster.identity.principalId
$foundryPrincipalId = $foundryExtension.identity.principalId
$extensionPrincipalId = $extension.identity.principalId
```

Assign the ARM roles:

```powershell
# Connected-cluster identity: Reader on the connected-cluster resource.
az role assignment create `
  --assignee-object-id $clusterPrincipalId `
  --assignee-principal-type ServicePrincipal `
  --role Reader `
  --scope $clusterScope

# Foundry extension identity: Reader for ARM RBAC validation.
az role assignment create `
  --assignee-object-id $foundryPrincipalId `
  --assignee-principal-type ServicePrincipal `
  --role Reader `
  --scope $clusterScope

# Agents and Tools with Foundry Local extension identity: Reader for ARM checks.
az role assignment create `
  --assignee-object-id $extensionPrincipalId `
  --assignee-principal-type ServicePrincipal `
  --role Reader `
  --scope $clusterScope

# Agents and Tools with Foundry Local extension identity: Contributor for inference on the Foundry extension.
az role assignment create `
  --assignee-object-id $extensionPrincipalId `
  --assignee-principal-type ServicePrincipal `
  --role Contributor `
  --scope $foundryScope
```

If Foundry returns `500 rbac_check_unavailable` and ARM reports a `roleAssignments/read` 403, grant the Foundry extension identity `Reader` at the resource-group scope instead of the connected-cluster scope.

Role assignments can take 5 to 30 minutes to propagate. Wait before retrying, and restart the Foundry model deployment and the current model-consuming deployments after propagation:

```powershell
kubectl rollout restart deployment/<modelName> --namespace foundry-local-operator
kubectl rollout status deployment/<modelName> --namespace foundry-local-operator

$deployments = @(
  "inferencingflow-deployment",
  "vectordb-api-server-deployment",
  "agents-management-deployment",
  "agents-execution-deployment",
  "kb-execution-deployment"
)

foreach ($deployment in $deployments) {
  if (kubectl get deployment $deployment --namespace arc-rag --ignore-not-found) {
    kubectl rollout restart deployment/$deployment --namespace arc-rag
    kubectl rollout status deployment/$deployment --namespace arc-rag
  }
}
```

Don't repeat the role assignments while propagation is in progress.

## Verify the installation

Confirm that the extension finished provisioning:

```powershell
az k8s-extension show `
  --cluster-type connectedClusters `
  --cluster-name <cluster> `
  --resource-group <rg> `
  --name edgerag `
  --query provisioningState -o tsv
```

Expect `Succeeded`.

## Related content

- [Disconnected operations for Agents and Tools with Foundry Local overview](overview-disconnected.md)
- [Deployment overview for Agents and Tools with Foundry Local](../deploy-overview.md)
- [Deploy the extension for Agents and Tools with Foundry Local](../deploy.md)
- [Configure authentication for Agents and Tools with Foundry Local](../prepare-authentication.md)
- [Deployment parameter reference and troubleshooting](../deploy-reference.md)
- [Configure "BYOM" endpoint authentication for Agents and Tools with Foundry Local](../configure-endpoint-authentication.md)
