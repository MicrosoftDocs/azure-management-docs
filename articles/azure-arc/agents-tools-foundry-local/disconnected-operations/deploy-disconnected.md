---
title: Install Agents and Tools with Foundry Local for Disconnected Operations on Azure Local
description: "Learn how to install the Agentic Retrieval extension on an Azure Local cluster that uses disconnected operations, including authentication, app registration, and the install command."
author: cwatson-cat
ms.author: cwatson
ms.topic: how-to
ms.date: 06/18/2026
ai-usage: ai-assisted
ms.subservice: edge-rag
#CustomerIntent: As a cloud administrator, I want to install Agents and Tools with Foundry Local on an Azure Local cluster that uses disconnected operations so that I can run an AI-powered chat solution in a disconnected environment.
---

# Install Agents and Tools with Foundry Local for disconnected operations on Azure Local

This article shows you how to install the Agentic Retrieval extension, part of the [Agents and Tools with Foundry Local](../overview.md) platform, on an Azure Local cluster configured for [disconnected operations for Azure Local](/azure/azure-local/manage/disconnected-operations-overview). Use this approach when you need to run an AI-powered chat solution that uses Foundry models without external internet connectivity.

In this article, you choose an authentication method, configure Microsoft Entra app registration for end-user sign-in, assign required app roles, grant permissions for managed identity scenarios, prepare installation inputs, run the installation command, and verify successful provisioning. For a standard connected deployment, see [Deploy the extension for Agentic Retrieval in Foundry Local](../deploy.md).

[!INCLUDE [preview-notice](../includes/preview-notice.md)]

## Prerequisites

Before you begin, verify these prerequisites for your environment. Throughout this article, stamp refers to your specific disconnected environment instance, including its domain and service endpoints.

### Disconnected environment preconditions

Make sure your disconnected environment is prepared with the required pack and extension type:

| Requirement | How to check |
|---|---|
| **Agents and Tools with Foundry Local expansion pack loaded** into your disconnected environment. | [Download and import the Agentic Retrieval expansion pack](prepare-disconnected.md), then confirm with your platform onboarding process for the stamp (your disconnected environment instance). |
| **Agentic Retrieval extension type registered** (`microsoft.arc.rag`). | `az k8s-extension extension-types list --cluster-type connectedClusters --cluster-name <cluster> --resource-group <rg>` includes `microsoft.arc.rag`. |

### Tools and cluster readiness

On the disconnected management machine, ensure the following conditions are true before you begin:

| Requirement | How to check |
|---|---|
| **Tools**: PowerShell and `az` CLI, signed in to your subscription (`az login` against the disconnected stamp). `kubectl` configured against the target cluster. | `az account show` returns your subscription; `kubectl get nodes` lists the cluster nodes. |
| **Foundry Local installed** with at least one model running. | `kubectl get mdep -n foundry-local-operator` shows a `ModelDeployment` with `phase=Running`. |
| **A policy-enforcing CNI** (Calico, Cilium, Azure NPM, Antrea, or kube-router) is installed. Agentic Retrieval uses NetworkPolicies by default and requires one. | `kubectl get ds -A` lists one of these CNI solutions. |

### Values to collect before installation

You need the Foundry **model deployment name** (for example, `phi4-cpu-demo`) and its **catalog model ID** (for example, `Phi-4-mini-instruct-generic-cpu`). Pass them at install time.

Substitute your own values for these placeholders throughout this article:

| Placeholder | Meaning | Example |
|---|---|---|
| `<rg>` | Resource group of the connected cluster | `developer` |
| `<cluster>` | Connected cluster name | `aldo-cluster` |
| `<ingress-domain>` | Ingress hostname on your stamp | `edgerag.autonomous.cloud.private` |
| `<stamp-domain>` | Your stamp's disconnected domain | `autonomous.cloud.private` |

## Choose how Agentic Retrieval authenticates to Foundry

Choose the authentication method that Agentic Retrieval uses for Foundry model calls. End-user sign-in for the extension endpoints is always enabled and configured separately in [Create the Agentic Retrieval app registration](#create-the-agentic-retrieval-app-registration). Compare the following options to choose the authentication method that best fits your environment.

| Mode | What it is | Trade-off |
|---|---|---|
| **Managed identity (recommended)** | The cluster managed identity authenticates to Foundry via ARM RBAC. Set with `--config foundryClientId=<...>`. | Production-grade, no secrets to rotate. Requires the one-time grant in [Grant Contributor on the Foundry extension scope (managed identity auth only)](#grant-contributor-on-the-foundry-extension-scope-managed-identity-auth-only). |
| **API key (bring your own model [BYOM])** | You pre-create a Kubernetes secret that holds Foundry's API key. | Simplest to start. The key changes every time Foundry redeploys the model, so you must refresh the secret and restart the Agents and Tools pods when that happens. |

The install command differs slightly between the two. Both are shown in [Install the extension](#install-the-extension).

## Create the Agentic Retrieval app registration

Agentic Retrieval requires real Microsoft Entra application credentials for end-user sign-in. On a disconnected stamp there's no portal UI for this, so you create the app with the `az` CLI. Run the commands on the disconnected management machine, logged in via `az login`.

### Create the app and service principal

```powershell
az login   # authenticates against the disconnected stamp

$app = az ad app create `
  --display-name "EdgeRAG" `
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

Agentic Retrieval defines three roles. The role IDs in the following list are arbitrary identifiers (any GUIDs work). Reuse them across installs so role assignments stay consistent.

```powershell
$roles = @(
  @{ allowedMemberTypes = @("User"); description = "Edge-RAG Developer (full access)"; displayName = "EdgeRAGDeveloper"; id = "11111111-1111-1111-1111-111111111111"; isEnabled = $true; value = "EdgeRAGDeveloper" },
  @{ allowedMemberTypes = @("User"); description = "Edge-RAG End User (chat-only)";    displayName = "EdgeRAGEndUser";   id = "22222222-2222-2222-2222-222222222222"; isEnabled = $true; value = "EdgeRAGEndUser" },
  @{ allowedMemberTypes = @("User"); description = "Default collection access";        displayName = "edgeragapp";       id = "33333333-3333-3333-3333-333333333333"; isEnabled = $true; value = "edgeragapp" }
) | ConvertTo-Json -Depth 5 -Compress
$roles | Out-File -Encoding ascii app-roles.json

az ad app update --id $appId --app-roles "@app-roles.json"
```

### Assign roles to your users

Assign the app roles to your users by using the following script. You can repeat these steps to grant access to additional users later.

If disconnected Microsoft Entra, `/users/{id}/appRoleAssignments` returns `Not Found`. Use `/servicePrincipals/{spObjectId}/appRoleAssignedTo` instead.

```powershell
$myObjectId  = az ad signed-in-user show --query id -o tsv
$spObjectId  = az ad sp show --id $appId --query id -o tsv

foreach ($roleId in @(
    "11111111-1111-1111-1111-111111111111",
    "22222222-2222-2222-2222-222222222222",
    "33333333-3333-3333-3333-333333333333"
)) {
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

## Grant Contributor on the Foundry extension scope (managed identity auth only)

Skip this step if you chose API-key (BYOM) auth.

Disconnected Foundry authorizes inference via ARM RBAC. Grant the cluster managed identity the built-in **Contributor** role on the Foundry extension's ARM scope.

```powershell
$RG  = "<rg>"
$CLUSTER = "<cluster>"
$SUB = az account show --query id -o tsv

# Cluster managed identity principal
$EDGERAG_MI_OID = az resource show `
  --ids "/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Kubernetes/connectedClusters/$CLUSTER" `
  --query identity.principalId -o tsv

az role assignment create `
  --assignee-object-id $EDGERAG_MI_OID `
  --assignee-principal-type ServicePrincipal `
  --role Contributor `
  --scope "/subscriptions/$SUB/resourceGroups/$RG/providers/Microsoft.Kubernetes/connectedClusters/$CLUSTER/providers/Microsoft.KubernetesConfiguration/extensions/foundry"
```

Get the **Foundry app registration's `appId`** (you pass it as `foundryClientId`):

```powershell
az ad app list --display-name "FoundryOnArc-Disconnected" --query "[0].appId" -o tsv
```

Role propagation takes 5 to 30 minutes on disconnected Azure. The assignment is recorded immediately, but verification of the assignment takes time. If you get a `401` authentication error after installation, wait a few minutes, restart the Agentic Retrieval pods, and try again. Don't re-run the role assignment command.

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

### Pick the load-balancer IP

The extension's ingress needs a load-balancer IP from your cluster's ingress subnet, supplied at install time (it can't be baked into the pack).

1. On the disconnected management machine, find the ingress network interface card (NIC) and its subnet:

   ```powershell
   Get-NetIPAddress -AddressFamily IPv4 | Where-Object PrefixOrigin -ne 'WellKnown' | Format-Table IPAddress, PrefixLength, InterfaceAlias
   ```

   The ingress-subnet NIC's IP and prefix tell you the subnet (for example, `10.0.60.1/24` → host range `.1`–`.254`).
1. Pick a free IP high in the range (avoid the `.1` gateway and the node DHCP pool). A common pattern is the **last IP of the node pool**.
1. Confirm it's free:

   ```powershell
   Test-Connection -ComputerName <candidate-IP> -Count 2 -Quiet   # expect $False
   ```

1. Pass it as `--config metallb.ipRange=<candidate-IP>/32` at install time.

## Install the extension

Pick the option that matches your [Choose how Agentic Retrieval authenticates to Foundry](#choose-how-agentic-retrieval-authenticates-to-foundry) choice. Install typically takes 10 to 25 minutes.

Substitute the following values: `<edgerag-appId>` and `<tenantId>` from [Create the app and service principal](#create-the-app-and-service-principal); `<lb-IP>` from [Pick the load-balancer IP](#pick-the-load-balancer-ip); `<modelName>` your `ModelDeployment` name; `<catalog-model-id>` the Foundry catalog model name.

### Option 1: Managed identity auth (recommended)

Add `--config foundryClientId=<foundry-appId>` from [Grant Contributor on the Foundry extension scope (managed identity auth only)](#grant-contributor-on-the-foundry-extension-scope-managed-identity-auth-only).

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
  --config auth.clientId=<edgerag-appId> `
  --config auth.tenantId=<tenantId> `
  --config foundryClientId=<foundry-appId> `
  --config ingress.domainname=<ingress-domain> `
  --config metallb.ipRange=<lb-IP>/32 `
  --config byom.apiEndpoint=https://<modelName>.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions `
  --config byom.apiModel=<catalog-model-id> `
  --config gpu_enabled=false
```

### Option 2: API key (BYOM) auth

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
  --config auth.clientId=<edgerag-appId> `
  --config auth.tenantId=<tenantId> `
  --config ingress.domainname=<ingress-domain> `
  --config metallb.ipRange=<lb-IP>/32 `
  --config byom.apiEndpoint=https://<modelName>.foundry-local-operator.svc.cluster.local:5000/v1/chat/completions `
  --config byom.apiModel=<catalog-model-id> `
  --config gpu_enabled=false
```

`gpu_enabled=false` is correct for CPU-only stamps. Set it to `true` only if your stamp has GPU nodes labeled for Agents and Tools with Foundry Local.

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

- [Deployment overview for Agentic Retrieval in Foundry Local](../deploy-overview.md)
- [Deploy the extension for Agentic Retrieval in Foundry Local](../deploy.md)
- [Configure authentication for Agentic Retrieval in Foundry Local](../prepare-authentication.md)
- [Deployment parameter reference and troubleshooting](../deploy-reference.md)
- [Configure "BYOM" endpoint authentication for Agentic Retrieval in Foundry Local](../configure-endpoint-authentication.md)