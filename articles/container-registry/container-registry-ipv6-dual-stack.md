---
title: IPv6 dual-stack endpoints in Azure Container Registry (preview)
description: Learn how to enable IPv6 dual-stack endpoints on an Azure container registry so clients can reach the registry over both IPv4 and IPv6.
ms.topic: how-to
author: johnsonshi
ms.author: johsh
ms.service: azure-container-registry
ms.date: 06/15/2026
# Customer intent: As a platform engineer operating IPv6-only or dual-stack networks, I want to enable IPv6 on my Azure container registry, so that my clients can pull and push container images over IPv6.
---

# IPv6 dual-stack endpoints in Azure Container Registry (preview)

Azure Container Registry supports an IPv6 dual-stack endpoint protocol in preview. When the endpoint protocol of a registry is set to `IPv4AndIPv6`, the registry's endpoints are reachable over both IPv4 and IPv6, so clients on IPv4-only, dual-stack, and IPv6-capable networks can authenticate, push, and pull against the same registry.

> [!IMPORTANT]
> IPv6 dual-stack endpoints are currently in preview. This preview enables IPv6 for the registry's public endpoints only (the login server, dedicated data endpoints, and regional endpoints). IPv6 over [private endpoints](container-registry-private-endpoints.md) isn't supported; private endpoint traffic continues to use IPv4. [ACR Tasks](container-registry-tasks-overview.md) isn't supported on a registry that has IPv6 dual-stack enabled. Tasks don't work when the endpoint protocol is set to `IPv4AndIPv6`, including quick builds (with `az acr build`) and quick task runs (with `az acr run`).

## Why IPv6 dual-stack endpoints

Teams adopt IPv6 for their container registry traffic for several reasons:

* **IPv6-only and dual-stack networks.** Clients in networks that prefer or require IPv6 — including newer cloud network deployments, telco and IoT environments, and modernized corporate networks — need their container registry reachable over IPv6.
* **Guarding against IPv4 address exhaustion.** Organizations migrating toward IPv6 reduce their dependence on increasingly scarce IPv4 address space.
* **Regulatory and organizational mandates.** Some organizations operate under requirements to transition services and clients to IPv6.

The dual-stack model means you don't have to choose: the registry continues to serve IPv4 clients while also serving IPv6 clients.

## Endpoint protocol values

The endpoint protocol is a registry-level setting with two values:

| Endpoint protocol | Behavior |
|-------------------|----------|
| `IPv4` (default) | Registry endpoints are reachable over IPv4 only. |
| `IPv4AndIPv6` (preview) | Registry endpoints are reachable over both IPv4 and IPv6 (dual stack). |

There's no IPv6-only mode. Dual stack preserves compatibility with existing IPv4 clients.

## Prerequisites

* A registry in the **Premium** [SKU](container-registry-skus.md).
* [Dedicated data endpoints](container-registry-dedicated-data-endpoints.md) enabled on the registry. Setting the endpoint protocol to `IPv4AndIPv6` requires `dataEndpointEnabled` to be `true`. This requirement is enforced by the service.
* Azure CLI version **2.87.0** or later for `az acr update --endpoint-protocol`. Run `az version` to check your version and `az upgrade` to update.

## Enable dual-stack endpoints on an existing registry

Enable dedicated data endpoints and set the endpoint protocol in a single update:

```azurecli
az acr update --name myregistry --data-endpoint-enabled true --endpoint-protocol IPv4AndIPv6
```

If dedicated data endpoints are already enabled on the registry, you can set the endpoint protocol on its own:

```azurecli
az acr update --name myregistry --endpoint-protocol IPv4AndIPv6
```

Verify the configuration:

```azurecli
az acr show --name myregistry --query "{endpointProtocol:endpointProtocol, dataEndpointEnabled:dataEndpointEnabled}"
```

Example output:

```output
{
  "dataEndpointEnabled": true,
  "endpointProtocol": "IPv4AndIPv6"
}
```

## Revert to IPv4-only endpoints

To revert the registry to IPv4-only endpoints:

```azurecli
az acr update --name myregistry --endpoint-protocol IPv4
```

Reverting the endpoint protocol doesn't disable dedicated data endpoints. To disable them as well, run `az acr update --name myregistry --data-endpoint-enabled false` after reverting the endpoint protocol to `IPv4`.

## Firewall and network considerations

* **FQDN-based firewall rules continue to work unchanged.** Rules that allow the registry login server (`myregistry.azurecr.io`), dedicated data endpoints (`myregistry.<region>.data.azurecr.io`), and regional endpoints (`myregistry.<region>.geo.azurecr.io`, if enabled) apply regardless of protocol.
* **IP-based allowlists need to account for IPv6.** If your client firewall allows registry access by IP address ranges instead of FQDNs, IPv6 client traffic to the registry needs corresponding IPv6 rules.
* **Dedicated data endpoints are part of the dual-stack model.** Because dual stack requires dedicated data endpoints, layer blob downloads are served from `myregistry.<region>.data.azurecr.io` rather than `*.blob.core.windows.net`. If you're enabling dedicated data endpoints for the first time as part of dual-stack adoption, review [Dedicated data endpoints](container-registry-dedicated-data-endpoints.md#enable-dedicated-data-endpoints) for the client firewall impact before enabling.

For more information, see [Configure rules to access an Azure container registry behind a firewall](container-registry-firewall-rules.md).

## Interactions with other registry features

| Feature | Interaction with `IPv4AndIPv6` |
|---------|--------------------------------|
| [Dedicated data endpoints](container-registry-dedicated-data-endpoints.md) | Required. The service rejects `IPv4AndIPv6` unless `dataEndpointEnabled` is `true`. |
| [SKU](container-registry-skus.md) | Premium SKU is required, because dedicated data endpoints are a Premium feature. |
| [Geo-replication](container-registry-geo-replication.md) | The endpoint protocol is a registry-level setting. In a geo-replicated registry, dedicated data endpoints exist in every replica region. |
| [Private endpoints](container-registry-private-endpoints.md) | The `endpointProtocol` setting applies to the registry's public endpoints. IPv6 over private endpoints isn't part of this preview. |
| [ACR Tasks](container-registry-tasks-overview.md) | Not supported. Tasks don't work when the endpoint protocol is set to `IPv4AndIPv6`, including quick builds (with `az acr build`) and quick task runs (with `az acr run`). |

## Next steps

* Learn about [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), the prerequisite for dual-stack endpoints.
* See the [Azure Container Registry endpoint reference](container-registry-endpoint-reference.md) for all endpoint types, URL formats, and CLI flags.
* Review [firewall access rules](container-registry-firewall-rules.md) for clients behind a firewall.
