---
title: "Glossary for Connected Registry with Azure Arc"
description: "Learn the terms and definitions for the connected registry extension with Azure Arc for a seamless extension deployment."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: glossary #Don't change
ms.date: 02/28/2025

# Customer intent: As an IT professional deploying the connected registry extension with Azure Arc, I want to understand key terms and definitions, so that I can ensure a smooth and successful deployment process.
---

# Glossary for Connected registry with Azure Arc

This glossary provides terms and definitions for the connected registry extension with Azure Arc for a seamless extension deployment.

## Glossary of terms

### Auto-upgrade-version

- **Definition:** Automatically upgrade the version of the extension instance.
- **Accepted Values:** `true`, `false`
- **Default Value:** `false`
- **Note:** [Azure Connected Machine agent](/azure/azure-arc/servers/agent-overview) manages the upgrade process and automatic rollback.

### Bring Your Own Certificate (BYOC)

- **Definition:** Allows customers to use their own certificate management service.
- **Accepted Values:** Kubernetes Secret or Public Certificate + Private Key pair
- **Note:** Customers must specify.

### Cert-manager.enabled

- **Definition:** Enables cert-manager service for use with the connected registry, handling the TLS certificate management lifecycle.
- **Accepted Values:** `true`, `false`
- **Default Value:** `true`
- **Note:** Customers can either use the provided cert-manager service at deployment or use theirs (must already be installed).

### Cert-manager.install

- **Definition:** Installs the cert-manager tool as part of the extension deployment.
- **Accepted Values:** `true`, `false`
- **Default Value:** `true`
- **Note:** Set this value to `false` if you're using your own cert-manager service.

### Child Registry

- **Description:** A registry that synchronizes with its parent (top-level) registry. The modes of the parent and child registries must match to ensure compatibility.

### Client Token

- **Definition:** Manages client access to a connected registry, allowing for actions on one or more repositories.
- **Accepted Values:** Token name
- **Note:** After creating a token, configure the connected registry to accept it using the `az acr connected-registry update` command.

### Cloud Registry

- **Description:** The ACR registry from which the connected registry syncs artifacts.

### Cluster-name

- **Definition:** The name of the Arc cluster for which you deploy the extension.
- **Accepted Values:** Alphanumeric value

### Cluster-type

- **Definition:** The type of Arc cluster for the extension deployment.
- **Accepted Values:** `connectedCluster`
- **Default Value:** `connectedCluster`

### Single configuration value (--config)

- **Definition:** The configuration parameters and values for deploying the connected registry extension on the Arc Kubernetes cluster.
- **Accepted Values:** Alphanumeric value

### Connection String

- **Value Type:** Alphanumeric
- **Customer Action:** Must generate and specify
- **Description:** The connection string contains authorization details necessary for the connected registry to securely connect and sync data with the cloud registry using Shared Key authorization. It includes the connected registry name, sync token name, sync token password, parent gateway endpoint, and parent endpoint protocol.

### Connected Registry

- **Description:** The on-premises or remote registry replica that facilitates local access to containerized workloads synchronized from the ACR registry.

### Data-endpoint-enabled

- **Definition:** Enables a [dedicated data endpoint](/azure/container-registry/container-registry-dedicated-data-endpoints) for client firewall configuration.
- **Accepted Values:** `true`, `false`
- **Default Value:** `false`
- **Note:** You must enable this setting for a successful creation of a connected registry.

### Extension-type

- **Definition:** Specifies the extension provider unique name for the extension deployment.
- **Accepted Values:** `Microsoft.ContainerRegistry.ConnectedRegistry`
- **Default Value:** `Microsoft.ContainerRegistry.ConnectedRegistry`

### Kubernetes Secret

- **Definition:** A Kubernetes managed secret for securely accessing data across pods within a cluster.
- **Accepted Values:** Secret name
- **Note:** You must specify this value.

### Message TTL (Time To Live)

- **Value Type:** Numerical
- **Default Value/Behavior:** Every two days
- **Description:** Message TTL defines the duration sync messages are retained in the cloud. This value isn't applicable when the sync schedule is continuous.

### Modes

- **Accepted Values:** `ReadOnly` and `ReadWrite`
- **Default Value/Behavior:** `ReadOnly`
- **Description:** Defines the operational permissions for client access to the connected registry. In `ReadOnly` mode, clients can only pull (read) artifacts, which is also suitable for nested scenarios. In `ReadWrite` mode, clients can pull (read) and push (write) artifacts, which is ideal for local development environments.

### Parent Registry

- **Description:** The primary registry that synchronizes with its child connected registries. A single parent registry can have multiple child registries connected to it. In a nested scenario, there can be multiple layers of registries within the hierarchy.

### Protected Settings File (--config-protected-file)

- **Definition:** The file containing the connection string for deploying the connected registry extension on the Kubernetes cluster. This file also includes the Kubernetes Secret or Public Cert + Private Key values pair for BYOC scenarios.
- **Accepted Values:** Alphanumeric value
- **Note:** You must specify this file.

### Public Certificate + Private Key

- **Value Type:** Alphanumeric base64-encoded
- **Customer Action:** Must specify
- **Description:** The public key certificate comprises a pair of keys: a public key available to anyone for identity verification of the certificate holder, and a private key, a unique secret key.

### Pvc.storageClassName

- **Definition:** Specifies the storage class in use on the cluster.
- **Accepted Values:** `standard`, `azurefile`

### Pvc.storageRequest

- **Definition:** Specifies the storage size that the connected registry claims in the cluster.
- **Accepted Values:** Alphanumeric value (for example, `500Gi`)
- **Default Value:** `500Gi`

### Service.ClusterIP

- **Definition:** The IP address within the Kubernetes service cluster IP range.
- **Accepted Values:** IPv4 or IPv6 format
- **Note:** You must specify. An incorrect IP not within the range results in a failed extension deployment.

### Sync Token

- **Definition:** A token used by each connected registry to authenticate with its immediate parent for content synchronization and updates.
- **Accepted Values:** Token name
- **Action:** Your action required.

### Synchronization Schedule

- **Value Type:** Numerical
- **Default Value/Behavior:** Every minute
- **Description:** The synchronization schedule, set using a cron expression, determines the cadence for when the registry syncs with its parent.

### Synchronization Window

- **Value Type:** Alphanumeric
- **Default Value/Behavior:** Hourly
- **Description:** The synchronization window specifies the sync duration. This parameter is disregarded if the sync schedule is continuous.

### TrustDistribution.enabled

- **Definition:** Trust distribution refers to the process of securely distributing trust between the connected registry and all client nodes within a Kubernetes cluster. When you enable trust distribution, all nodes are configured with trust distribution.
- **Accepted Values:** `true`, `false`
- **Note:** You must choose `true` or `false`.

### TrustDistribution.useNodeSelector

- **Definition:** By default, the trust distribution daemonsets, which are responsible for configuring the container runtime environment (containerd), run on all nodes in the cluster. However, with this setting enabled, trust distribution is limited to only those nodes that you specifically label with `containerd-configured-by: connected-registry`.
- **Accepted Values:** `true`, `false`
- **Label:** `containerd-configured-by=connected-registry`
- **Command to specify nodes for trust distribution:** `kubectl label node/[node name] containerd-configured-by=connected-registry`


### Registry Hierarchy

- **Description:** The structure of connected registries, where each connected registry is linked to a parent registry. The top parent in this hierarchy is the ACR registry.
