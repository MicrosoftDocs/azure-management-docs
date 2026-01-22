---
title: "Known issues: Connected Registry Arc Extension"
description: "Learn how to troubleshoot the most common problems for a Connected Registry Arc Extension and resolve issues with ease."
author: rayoef
ms.author: rayoflores
ms.service: azure-container-registry
ms.topic: troubleshooting-known-issue #Don't change.
ms.date: 05/09/2024
ms.custom: sfi-ropc-nochange
# Customer intent: "As an IT operator managing Arc-enabled Kubernetes clusters, I want to troubleshoot common issues with the connected registry extension, so that I can ensure successful installations and maintain proper functionality."
---

# Troubleshoot connected registry extension 

This article discusses some common error messages that you may receive when you install or update the connected registry extension for Arc-enabled Kubernetes clusters.

## Check extension and pod status

The connected registry extension is released as a Helm chart and installed by Helm V3. All components of the connected registry extension are installed in the _connected-registry_ namespace. Use the following commands to check the extension status.

```bash
# get the extension status 
az k8s-extension show --name <extension-name>  
# check status of all pods of connected registry extension 
kubectl get pod -n connected-registry    
# get events of the extension 
kubectl get events -n connected-registry   --sort-by='.lastTimestamp'
```

## Resolve common errors

Review this section to find tips on resolving errors you may encounter when installing or updating the connected registry extension.

### Error: can't reuse a name that is still in use

This error means the extension name you specified already exists. Use a different name for the extension.

The connected registry name must start with a letter and contain only alphanumeric characters. It must be 5 to 40 characters long.

### Error: unable to create new content in namespace _connected-registry_ because it's being terminated 

This error happens when an uninstallation operation isn't finished, and another installation operation is triggered. Run `az k8s-extension show` command to check the provisioning status of the extension and make sure the extension has been uninstalled before taking other actions.

### Error: failed in download the Chart path not found 

This error happens when you specify an extension version that doesn't exist. Update your command to use a valid version, or to use the latest extension version, don't specify `--version` at all.

## Common scenarios

This section describes common scenarios that may cause issues when installing or updating the connected registry extension, and how to resolve them.

### Installation fails without an error message

If the extension generates an error message when you create or update it, inspect where the creation failed by running the `az k8s-extension list` command:

```bash
az k8s-extension list \ 
--resource-group <my-resource-group-name> \ 
--cluster-name <my-cluster-name> \ 
--cluster-type connectedClusters
```
 
To resolve this issue, try restarting the cluster, and make sure the KubernetesConfiguration service provider is registered. Alternately, delete and reinstall the connected registry extension.

### Extension creation stuck in running state

**Possibility 1:** Issue with Persistent Volume Claim (PVC)

- Check status of connected registry PVC 
```bash
kubectl get pvc -n connected-registry -o yaml connected-registry-pvc
``` 

The value of _phase_ under _status_ should be _bound_. If it doesn’t change from _pending_, delete the extension. 

- Check whether the desired storage class is in your list of storage classes: 

```bash
kubectl get storageclass --all-namespaces
``` 

- If not, recreate the extension and add
   
```bash
--config pvc.storageClassName=”standard”` 
``` 

- Alternatively, it could be an issue with not having enough space for the PVC. Recreate the extension with the parameter  

```bash
--config pvc.storageRequest=”250Gi”` 
``` 

**Possibility 2:** Connection String is bad 

- Check the logs for the connected registry Pod: 

```bash
kubectl get pod -n connected-registry
``` 

- Copy the name of the connected registry pod (e.g.: “connected-registry-8d886cf7f-w4prp") and paste it into the following command: 

```bash
kubectl logs -n connected-registry connected-registry-8d886cf7f-w4prp
```  

- If you see the following error message, the connected registry's connection string is bad: 

```bash
Response: '{"errors":[{"code":"UNAUTHORIZED","message":"Incorrect Password","detail":"Please visit https://aka.ms/acr#UNAUTHORIZED for more information."}]}' 
``` 

- Ensure that a _protected-settings-extension.json_ file has been created 

```bash
cat protected-settings-extension.json
``` 

- If needed, regenerate _protected-settings-extension.json_ 

```bash
cat << EOF > protected-settings-extension.json  
{ 
"connectionString": "$(az acr connected-registry get-settings \ 
--name myconnectedregistry \ 
--registry myacrregistry \ 
--parent-protocol https \ 
--generate-password 1 \ 
--query ACR_REGISTRY_CONNECTION_STRING --output tsv --yes)" 
} 
EOF
``` 

- Update the extension to include the new connection string 

```bash
az k8s-extension update \ 
--cluster-name <myarck8scluster> \ 
--cluster-type connectedClusters \ 
--name <myconnectedregistry> \ 
-g <myresourcegroup> \ 
--config-protected-file protected-settings-extension.json
```

### Extension created, but connected registry is not in 'Online' state 

**Possibility 1:** Previous connected registry has not been deactivated 

This scenario commonly happens when a previous connected registry extension has been deleted and a new one has been created for the same connected registry. 

- Check the logs for the connected registry Pod: 

```bash
kubectl get pod -n connected-registry
```

- Copy the name of the connected registry pod (e.g.: “connected-registry-xxxxxxxxx-xxxxx") and paste it into the following command: 

```bash
kubectl logs -n connected-registry connected-registry-xxxxxxxxx-xxxxx
```

- If you see the following error message, the connected registry needs to be deactivated:  

`Response: '{"errors":[{"code":"ALREADY_ACTIVATED","message":"Failed to activate the connected registry as it is already activated by another instance. Only one instance is supported at any time.","detail":"Please visit https://aka.ms/acr#ALREADY_ACTIVATED for more information."}]}'` 

- Run the following command to deactivate:  

```azurecli
az acr connected-registry deactivate -n <myconnectedregistry> -r <mycontainerregistry>
```

After a few minutes, the connected registry pod should be recreated, and the error should disappear. 

## Enable logging

- Run the `az acr connected-registry update` command to update the connected registry extension with the debug log level:

```azurecli
az acr connected-registry update --registry mycloudregistry --name myacrregistry --log-level debug
```

- The following log levels can be applied to aid in troubleshooting:

  - **Debug** provides detailed information for debugging purposes.

  - **Information** provides general information for debugging purposes.

  - **Warning** indicates potential problems that aren't yet errors but might become one if no action is taken.

  - **Error** logs errors that prevent an operation from completing.

  - **None** turns off logging, so no log messages are written.

- Adjust the log level as needed to troubleshoot the issue.

The active selection provides more options to adjust the verbosity of logs when debugging issues with a connected registry. The following options are available:

The connected registry log level is specific to the connected registry's operations and determines the severity of messages that the connected registry handles. This setting is used to manage the logging behavior of the connected registry itself.

**--log-level** set the log level on the instance. The log level determines the severity of messages that the logger handle. By setting the log level, you can filter out messages that are below a certain severity. For example, if you set the log level to "warning" the logger handles warnings, errors, and critical messages, but it ignores information and debug messages.

The az cli log level controls the verbosity of the output messages during the operation of the Azure CLI. The Azure CLI (az) provides several verbosity options for log levels, which can be adjusted to control the amount of output information during its operation:

**--verbose** increases the verbosity of the logs. It provides more detailed information than the default setting, which can be useful for identifying issues.

**--debug** enables full debug logs. Debug logs provide the most detailed information, including all the information provided at the "verbose" level plus more details intended for diagnosing problems.

## Glossary of terms

Review this information to help understand the terminology used in connected registry extension deployment and management.

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