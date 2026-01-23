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

# Troubleshoot connected registry problems 

This article describes common error messages that you might receive when you install or update the connected registry extension for Arc-enabled Kubernetes clusters.

## Check extension and pod status

The connected registry extension is a Helm chart. All components of the connected registry extension are installed in the _connected-registry_ namespace. Use the following commands to check the extension status.

```bash
# get the extension status 
az k8s-extension show --name <extension-name>  
# check status of all pods of connected registry extension 
kubectl get pod -n connected-registry    
# get events of the extension 
kubectl get events -n connected-registry   --sort-by='.lastTimestamp'
```

## Troubleshoot extension installation or updates

Use the following suggestions to resolve errors you might encounter when installing or updating the connected registry extension.

### Error: can't reuse a name that is still in use

This error message means the extension name you specified already exists. Use a different name for the extension.

The connected registry name must start with a letter and contain only alphanumeric characters. It must be 5 to 40 characters long.

### Error: unable to create new content in namespace _connected-registry_ because it's being terminated

This error message occurs when an uninstallation operation isn't finished, and another installation operation starts. Run the `az k8s-extension show` command to check the provisioning status of the extension. Make sure the extension is uninstalled before taking other actions.

### Error: failed in download the Chart path not found

This error message occurs when you specify an extension version that doesn't exist. Update your command to use a valid version. To use the latest extension version, don't specify `--version`.

### Installation fails without an error message

If the extension generates an error message when you create or update it, check where the creation failed by running the `az k8s-extension list` command.

```bash
az k8s-extension list \ 
--resource-group <my-resource-group-name> \ 
--cluster-name <my-cluster-name> \ 
--cluster-type connectedClusters
```
 
To resolve this problem, try restarting the cluster, and make sure the KubernetesConfiguration service provider is registered. Or, delete and reinstall the connected registry extension.

### Extension creation stuck in running state

One reason for this problem is due to issues with Persistent Volume Claim (PVC). Check the status of the connected registry PVC by running the following command:

```bash
kubectl get pvc -n connected-registry -o yaml connected-registry-pvc
```

The value of `phase` under `status` should be `bound`. If it doesn't change from `pending, delete the extension and try again.

Check whether the desired storage class is in your list of storage classes:

```bash
kubectl get storageclass --all-namespaces
```

If it's not, recreate the extension and add:

```bash
--config pvc.storageClassName=”standard”` 
```

If you need more space for the PVC, try recreating the extension with this parameter:

```bash
--config pvc.storageRequest="250Gi"` 
``` 

The extension can also get stuck due to a bad connection string. Check the logs for the connected registry pod:

```bash
kubectl get pod -n connected-registry
```

Copy the name of the connected registry pod (for example, `connected-registry-xxxxxxxxx-xxxxx`) and paste it into the following command:

```bash
kubectl logs -n connected-registry connected-registry-xxxxxxxxx-xxxxx
```  

If you see the following error message, the connected registry's connection string is bad:

```bash
Response: '{"errors":[{"code":"UNAUTHORIZED","message":"Incorrect Password","detail":"Please visit https://aka.ms/acr#UNAUTHORIZED for more information."}]}' 
```

To resolve this problem, ensure that a protected-settings-extension.json file is created:

```bash
cat protected-settings-extension.json
```

If needed, regenerate protected-settings-extension.json.

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

Then update the extension to include the new connection string.

```bash
az k8s-extension update \ 
--cluster-name <myarck8scluster> \ 
--cluster-type connectedClusters \ 
--name <myconnectedregistry> \ 
-g <myresourcegroup> \ 
--config-protected-file protected-settings-extension.json
```

### Extension created, but connected registry isn't in **Online** state

This scenario commonly happens when you delete a previous connected registry extension and create a new one for the same connected registry.

Check the logs for the connected registry pod:

```bash
kubectl get pod -n connected-registry
```

Copy the name of the connected registry pod (for example, `connected-registry-xxxxxxxxx-xxxxx`) and paste it into the following command:

```bash
kubectl logs -n connected-registry connected-registry-xxxxxxxxx-xxxxx
```

If you see the following error message, deactivate the connected registry:  

`Response: '{"errors":[{"code":"ALREADY_ACTIVATED","message":"Failed to activate the connected registry as it is already activated by another instance. Only one instance is supported at any time.","detail":"Please visit https://aka.ms/acr#ALREADY_ACTIVATED for more information."}]}'`

Run the following command to deactivate:  

```azurecli
az acr connected-registry deactivate -n <myconnectedregistry> -r <mycontainerregistry>
```

If successful, fter a few minutes, the connected registry pod is recreated, and the error should no longer occur.

## Enable logging

Run the `az acr connected-registry update` command to update the connected registry extension with the debug log level:

```azurecli
az acr connected-registry update --registry mycloudregistry --name myacrregistry --log-level debug
```

Apply the following log levels to help troubleshoot problems:

- **Debug** provides detailed information for debugging purposes.

- **Information** provides general information for debugging purposes.

- **Warning** indicates potential problems that aren't yet errors but might become errors if no action is taken.

- **Error** logs errors that prevent an operation from completing.

- **None** turns off logging, so no log messages are written.

Adjust the log levels as needed to troubleshoot the problem further.

The connected registry log level is specific to the connected registry's operations and determines the severity of messages that the connected registry handles. Use this setting to manage the logging behavior of the connected registry itself.

**--log-level** sets the log level on the instance. The log level determines the severity of messages that the logger handles. By setting the log level, you can filter out messages that are below a certain severity. For example, if you set the log level to `warning`, the logger handles warnings, errors, and critical messages, but it ignores information and debug messages.

The Azure CLI log level controls the verbosity of the output messages during the operation of the Azure CLI. The Azure CLI (`az`) provides several verbosity options for log levels, which you can adjust to control the amount of output information during its operation:

**--verbose** increases the verbosity of the logs. It provides more detailed information than the default setting, which can be useful for identifying problems.

**--debug** enables full debug logs. Debug logs provide the most detailed information, including all the information provided at the `verbose` level plus more details intended for diagnosing problems.