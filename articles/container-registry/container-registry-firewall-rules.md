---
title: Configure Firewall Access Rules to access an Azure Container Registry
description: Configure rules to access an Azure container registry from behind a firewall, allowing access to REST API and data endpoint domain names.
ms.topic: how-to
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.date: 02/24/2026
# Customer intent: As a network administrator, I want to configure firewall rules for Azure Container Registry access, so that I can ensure secure and reliable communication for pulling and pushing container images from devices behind a firewall.
---

# Configure rules to access an Azure container registry behind a firewall

This article explains how to configure rules on your firewall to allow access to an Azure container registry. For example, an Azure IoT Edge device behind a firewall or proxy server might need to access a container registry to pull a container image. Or, a locked-down server in an on-premises network might need access to push an image.

To configure inbound network access to a container registry only within an Azure virtual network, see [Configure Azure Private Link for an Azure container registry](container-registry-private-endpoints.md).

## About registry endpoints

To pull or push images or other artifacts to an Azure container registry, a client such as a Docker daemon needs to interact over HTTPS with two or more distinct endpoints. For clients that access a registry from behind a firewall, you need to configure access rules for each endpoint. All endpoints are reached over port 443.

* **Global endpoint (REST API)** — Authentication and registry management operations are handled through the registry's public REST API endpoint. This endpoint is the login server name of the registry. Example: `myregistry.azurecr.io`

  * **Registry REST API endpoint for certificates** — Azure container registry uses a wildcard SSL certificate for all subdomains. When connecting to the Azure container registry using SSL, the client must be able to download the certificate for the TLS handshake. In such cases, `azurecr.io` must also be accessible.

* **Storage (data) endpoint** — Azure [allocates blob storage](container-registry-storage.md) in Azure Storage accounts on behalf of each registry to manage the data for container images and other artifacts. When a client accesses image layers in an Azure container registry, it makes requests using a storage account endpoint provided by the registry.

If your registry is [geo-replicated](container-registry-geo-replication.md), a client might need to interact with the data endpoint in a specific region or in multiple replicated regions.

* **Regional endpoint** (if enabled) — When [regional endpoints](container-registry-geo-replication.md#regional-endpoints-of-a-geo-replicated-registry-preview) are enabled on a **Premium** SKU registry, each geo-replica gets a regional endpoint of the form `<registry-name>.<region>.geo.azurecr.io`. Regional endpoints allow clients to connect directly to a specific geo-replica for authentication and push/pull/delete operations, bypassing the global endpoint. If you use firewall rules, you need to allow access to the regional endpoint for each geo-replica that clients connect to.

For the full list of endpoint types and FQDN patterns, see the [endpoint reference](container-registry-endpoint-reference.md).

## Allow access to REST and data endpoints

* **REST endpoint** - Allow access to the fully qualified registry login server name, `<registry-name>.azurecr.io`, or an associated IP address range
* **Storage (data) endpoint** - Allow access to all Azure blob storage accounts using the wildcard `*.blob.core.windows.net`, or an associated IP address range.

> [!NOTE]
> Azure Container Registry supports [dedicated data endpoints](container-registry-dedicated-data-endpoints.md), allowing you to tightly scope client firewall rules for your registry storage. Optionally enable data endpoints in all regions where the registry is located or replicated, using the form `<registry-name>.<region>.data.azurecr.io`.
>
> If [regional endpoints](container-registry-geo-replication.md#regional-endpoints-of-a-geo-replicated-registry-preview) are also enabled, allow access to `<registry-name>.<region>.geo.azurecr.io` for each geo-replica that clients connect to.

## About Azure Container Registry FQDNs

Azure Container Registry uses two FQDNs: the login URL and the data endpoint.

* Both the login URL and the data endpoint are accessible from within the virtual network, using private IPs by enabling a private link.
* A registry that doesn't use data endpoints must access data from an endpoint of the form `*.blob.core.windows.net`. This doesn't provide the required isolation when configuring firewall rules.
* A registry with a private link enabled gets the dedicated data endpoint automatically.
* One dedicated data endpoint is created per region for a registry.
* The login URL remains the same whether dedicated data endpoint is enabled or disabled.

## Allow access by IP address range

If your organization has policies to allow access only to specific IP addresses or address ranges, download the latest version of [Azure IP Ranges and Service Tags – Public Cloud](https://www.microsoft.com/download/details.aspx?id=56519).

To find the ACR REST endpoint IP ranges for which you need to allow access, search for **AzureContainerRegistry** in the JSON file.

> [!IMPORTANT]
> IP address ranges for Azure services can change, and updates are published weekly. Download the JSON file regularly, and make necessary updates in your access rules. If your scenario involves configuring network security group rules in an Azure virtual network or you use Azure Firewall, use the **AzureContainerRegistry** [service tag](#allow-access-by-service-tag) instead.
>

### REST IP addresses for all regions

```json
{
  "name": "AzureContainerRegistry",
  "id": "AzureContainerRegistry",
  "properties": {
    "changeNumber": 10,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.66.140.72/29",
    [...]
```

### REST IP addresses for a specific region

Search for the specific region, such as **AzureContainerRegistry.AustraliaEast**.

```json
{
  "name": "AzureContainerRegistry.AustraliaEast",
  "id": "AzureContainerRegistry.AustraliaEast",
  "properties": {
    "changeNumber": 1,
    "region": "australiaeast",
    "platform": "Azure",
    "systemService": "AzureContainerRegistry",
    "addressPrefixes": [
      "13.70.72.136/29",
    [...]
```

### Storage IP addresses for all regions

```json
{
  "name": "Storage",
  "id": "Storage",
  "properties": {
    "changeNumber": 19,
    "region": "",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "13.65.107.32/28",
    [...]
```

### Storage IP addresses for specific regions

Search for the specific region, such as **Storage.AustraliaCentral**.

```json
{
  "name": "Storage.AustraliaCentral",
  "id": "Storage.AustraliaCentral",
  "properties": {
    "changeNumber": 1,
    "region": "australiacentral",
    "platform": "Azure",
    "systemService": "AzureStorage",
    "addressPrefixes": [
      "52.239.216.0/23"
    [...]
```

## Allow access by service tag

In an Azure virtual network, use network security rules to filter traffic from a resource such as a virtual machine to a container registry. To simplify the creation of Azure network rules, use the **AzureContainerRegistry** [service tag](/azure/virtual-network/network-security-groups-overview#service-tags). A service tag represents a group of IP address prefixes to access an Azure service globally or per Azure region. The tag is automatically updated when addresses change.

For example, create an outbound network security group rule with destination **AzureContainerRegistry** to allow traffic to an Azure container registry. To allow access to the service tag only in a specific region, specify the region in the following format: `AzureContainerRegistry.[region name]`.

## Enable dedicated data endpoints

For steps to enable dedicated data endpoints using the Azure portal or Azure CLI, see [Dedicated data endpoints in Azure Container Registry](container-registry-dedicated-data-endpoints.md#enable-dedicated-data-endpoints).

After you set up dedicated data endpoints for your registry, you can enable client firewall access rules for the data endpoints. Enable data endpoint access rules for all required registry regions.

## Configure client firewall rules for MCR

To access Microsoft Container Registry (MCR) from behind a firewall, see the guidance to configure [MCR client firewall rules](https://github.com/microsoft/containerregistry/blob/main/docs/client-firewall-rules.md). MCR is the primary registry for all Microsoft-published docker images, such as Windows Server images.

## Next steps

* Learn about [Azure best practices for network security](/azure/security/fundamentals/network-best-practices).
* Learn more about [security groups](/azure/virtual-network/network-security-groups-overview) in an Azure virtual network.
* Learn more about setting up [Private Link](container-registry-private-endpoints.md) for a container registry.
* Learn more about [dedicated data endpoints](https://azure.microsoft.com/blog/azure-container-registry-mitigating-data-exfiltration-with-dedicated-data-endpoints/) for Azure Container Registry.
* See the [endpoint reference](container-registry-endpoint-reference.md) for a complete list of registry endpoint types and FQDN patterns.

<!-- LINKS - Internal -->

[az-acr-update]: /cli/azure/acr#az-acr-update
[az-acr-show-endpoints]: /cli/azure/acr#az-acr-show-endpoints
