---
title: "Troubleshoot extension issues for Azure Arc-enabled Kubernetes clusters"
ms.date: 05/13/2025
ms.topic: how-to
ms.custom:
  - devx-track-azurecli
  - references_regions
  - build-2025
description: "Learn how to resolve common problems with Azure Arc-enabled Kubernetes cluster extensions."
# Customer intent: "As a Kubernetes administrator, I want to troubleshoot extension issues in Azure Arc-enabled clusters, so that I can ensure smooth operation and management of GitOps, Open Service Mesh, and other functionalities."
---

# Troubleshoot extension issues for Azure Arc-enabled Kubernetes clusters

This article describes troubleshooting tips for common problems related to [cluster extensions](extensions-release.md) like GitOps (Flux v2) in Azure or Open Service Mesh (OSM).

For help with troubleshooting Azure Arc-enabled Kubernetes problems in general, see [Troubleshoot Azure Arc-enabled Kubernetes issues](troubleshooting.md).

## GitOps (Flux v2)

> [!NOTE]
> You can use the Flux v2 extension in either an Azure Arc-enabled Kubernetes cluster or in an Azure Kubernetes Service (AKS) cluster. These tips generally apply to all cluster types.

For general help with troubleshooting problems when you use `fluxConfigurations` resources, run these Azure CLI commands with the `--debug` parameter:

```azurecli
az provider show -n Microsoft.KubernetesConfiguration --debug
az k8s-configuration flux create <parameters> --debug
```

### Helm chart config setting error

You may see an error message reading "`Unable to render the Helm chart with the provided config settings and config protected settings : Recommendation Please check if the values provided to the config settings and the config protected settings are valid for this extension type : InnerError [template: azure-k8s-flux/templates/source-controller.yaml:100:24: executing "azure-k8s-flux/templates/source-controller.yaml" at <index (lookup "v1" "ConfigMap" "kube-system" "extension-manager-config").data "AZURE_TENANT_ID">: error calling index: index of untyped nil]`".

To resolve this issue:

- If you're running `microsoft.flux` version 1.15.1 or earlier, upgrade to version 1.15.2 or later.
- If you're running `microsoft.flux` version 1.16.2 in the regions West Central US, France Central, UK South, or West Europe, upgrade to version 1.16.3 or later.

### Webhook dry run errors

Flux might fail and display an error similar to `dry-run failed, error: admission webhook "<webhook>" does not support dry run`. To resolve the problem, go to `ValidatingWebhookConfiguration` or `MutatingWebhookConfiguration`. In the configuration, set the value for `sideEffects` to `None` or `NoneOnDryRun`.

For more information, see [How do I resolve "webhook does not support dry run" errors?](https://fluxcd.io/flux/cheatsheets/troubleshooting/#how-do-i-resolve-webhook-does-not-support-dry-run-errors).

### Errors installing the microsoft.flux extension

The `microsoft.flux` extension installs Flux controllers and Azure GitOps agents in an Azure Arc-enabled Kubernetes cluster or AKS cluster. If the extension isn't already installed in a cluster and you [create a GitOps configuration resource](tutorial-use-gitops-flux2.md) for the cluster, the extension is installed automatically.

If you experience an error during installation or if the extension shows a `Failed` state, make sure that the cluster doesn't have any policies that restrict creating the `flux-system` namespace or any of the resources in that namespace.

For an AKS cluster, ensure that the `Microsoft.ContainerService/AKS-ExtensionManager` feature flag is enabled in the Azure subscription:

```azurecli
az feature register --namespace Microsoft.ContainerService --name AKS-ExtensionManager
```

Next, run the following command to determine if there are other problems. Set the cluster type parameter (`-t`) to `connectedClusters` for an Azure Arc-enabled cluster, or to `managedClusters` for an AKS cluster. If the extension was installed automatically when you created your GitOps configuration, the name of the `microsoft.flux` extension is `flux`.

```azurecli
az k8s-extension show -g <RESOURCE_GROUP> -c <CLUSTER_NAME> -n flux -t <connectedClusters or managedClusters>
```

The output can help you identify the problem and how to fix it. Possible remediation actions include:

- Force-delete the extension by running `az k8s-extension delete --force -g <RESOURCE_GROUP> -c <CLUSTER_NAME> -n flux -t <managedClusters OR connectedClusters>`.
- Uninstall the Helm release by running `helm uninstall flux -n flux-system`.
- Delete the `flux-system` namespace from the cluster by running `kubectl delete namespaces flux-system`.

Then, you can either [create a new Flux configuration](./tutorial-use-gitops-flux2.md), which installs the `microsoft.flux` extension automatically, or you can [install the Flux extension manually](extensions.md).

### Errors installing the microsoft.flux extension in a cluster that has a Microsoft Entra pod-managed identity

If you attempt to install the Flux extension in a cluster that has a Microsoft Entra pod-managed identity, an error might occur in the `extension-agent` pod. The output looks similar to this example:

```console
{"Message":"2021/12/02 10:24:56 Error: in getting auth header : error {adal: Refresh request failed. Status Code = '404'. Response body: no azure identity found for request clientID <REDACTED>\n}","LogType":"ConfigAgentTrace","LogLevel":"Information","Environment":"prod","Role":"ClusterConfigAgent","Location":"westeurope","ArmId":"/subscriptions/<REDACTED>/resourceGroups/<REDACTED>/providers/Microsoft.Kubernetes/managedclusters/<REDACTED>","CorrelationId":"","AgentName":"FluxConfigAgent","AgentVersion":"0.4.2","AgentTimestamp":"2021/12/02 10:24:56"}
```

The extension status returns as `Failed`:

```console
"{\"status\":\"Failed\",\"error\":{\"code\":\"ResourceOperationFailure\",\"message\":\"The resource operation completed with terminal provisioning state 'Failed'.\",\"details\":[{\"code\":\"ExtensionCreationFailed\",\"message\":\" error: Unable to get the status from the local CRD with the error : {Error : Retry for given duration didn't get any results with err {status not populated}}\"}]}}",
```

In this case, the `extension-agent` pod tries to get its token from Azure Instance Metadata Service on the cluster, but the token request is intercepted by the [pod identity](/azure/aks/use-azure-ad-pod-identity). To fix this problem, [upgrade to the latest version](extensions.md#upgrade-an-extension-instance) of the `microsoft.flux` extension.

### Memory and CPU resource requirements for installing the microsoft.flux extension

The controllers that are installed in your Kubernetes cluster when you install the `microsoft.flux` extension must have enough CPU and memory resources to properly schedule on a Kubernetes cluster node. Be sure that your cluster meets the minimum memory and CPU resource requirements.

The following table lists the minimum and maximum limits for potential CPU and memory resource requirements for this scenario:

| Container name | Minimum CPU | Minimum memory | Maximum CPU | Maximum memory |
| -------------- | ----------- | -------- |
| `fluxconfig-agent` | 5 m | 30 Mi | 50 m | 150 Mi |
| `fluxconfig-controller` | 5 m | 30 Mi | 100 m | 150 Mi |
| `fluent-bit` | 5 m | 30 Mi | 20 m | 150 Mi |
| `helm-controller` | 100 m | 64 Mi | 1,000 m | 1 Gi |
| `source-controller` | 50 m | 64 Mi | 1,000 m | 1 Gi |
| `kustomize-controller` | 100 m | 64 Mi | 1,000 m | 1 Gi |
| `notification-controller` | 100 m | 64 Mi | 1,000 m | 1 Gi |
| `image-automation-controller` | 100 m | 64 Mi | 1,000 m | 1 Gi |
| `image-reflector-controller` | 100 m | 64 Mi | 1,000 m | 1 Gi |

If you enabled a custom or built-in Azure Policy Gatekeeper policy that limits the resources for containers on Kubernetes clusters, ensure that either the resource limits on the policy are greater than the limits shown in the preceding table or that the `flux-system` namespace is part of the `excludedNamespaces` parameter in the policy assignment. An example of a policy in this scenario is `Kubernetes cluster containers CPU and memory resource limits should not exceed the specified limits`.

## Azure Monitor Container Insights

This section provides help with troubleshooting problems with [Container insights in Azure Monitor for Azure Arc-enabled Kubernetes clusters](/azure/azure-monitor/containers/container-insights-enable-arc-enabled-clusters?toc=%2Fazure%2Fazure-arc%2Fkubernetes%2Ftoc.json&bc=%2Fazure%2Fazure-arc%2Fkubernetes%2Fbreadcrumb%2Ftoc.json&tabs=create-cli%2Cverify-portal).

### Enable privileged mode for a Canonical Charmed Kubernetes cluster

Azure Monitor Container Insights requires its Kubernetes DaemonSet to run in privileged mode. To successfully set up a Canonical Charmed Kubernetes cluster for monitoring, run the following command:

```console
juju config kubernetes-worker allow-privileged=true
```

### Can't install AMA pods on Oracle Linux 9.x

If you try to install Azure Monitor Agent (AMA) on an Oracle Linux (Red Hat Enterprise Linux (RHEL)) 9.x Kubernetes cluster, the AMA pods and the AMA-RS pod might not work correctly due to the `addon-token-adapter` container in the pod. When you check the logs of the `ama-logs-rs` pod, in `addon-token-adapter container`, you see output that's similar to the following example:

```output
Command: kubectl -n kube-system logs ama-logs-rs-xxxxxxxxxx-xxxxx -c addon-token-adapter
 
Error displayed: error modifying iptable rules: error adding rules to custom chain: running [/sbin/iptables -t nat -N aad-metadata --wait]: exit status 3: modprobe: can't change directory to '/lib/modules': No such file or directory

iptables v1.8.9 (legacy): can't initialize iptables table `nat': Table does not exist (do you need to insmod?)

Perhaps iptables or your kernel needs to be upgraded.
```

This error occurs because installing the extension requires the `iptable_nat` module, but this module isn't automatically loaded in Oracle Linux (RHEL) 9.x distributions.

To fix this problem, you must explicitly load the `iptables_nat` module on each node in the cluster. Use the `modprobe` command `sudo modprobe iptables_nat`. After you sign into each node and manually add the `iptable_nat` module, retry the AMA installation.

> [!NOTE]
> Performing this step does not make the `iptables_nat` module persistent.  

## Azure Arc-enabled Open Service Mesh

This section demonstrates commands you can use to validate and troubleshoot the deployment of [Open Service Mesh (OSM)](tutorial-arc-enabled-open-service-mesh.md) extension components on your cluster.

### Check the OSM controller deployment

```bash
kubectl get deployment -n arc-osm-system --selector app=osm-controller
```

If the OSM controller is healthy, you see output similar to:

```output
NAME             READY   UP-TO-DATE   AVAILABLE   AGE
osm-controller   1/1     1            1           59m
```

### Check OSM controller pods

```bash
kubectl get pods -n arc-osm-system --selector app=osm-controller
```

If the OSM controller is healthy, output that looks similar to the following example appears:

```output
NAME                            READY   STATUS    RESTARTS   AGE
osm-controller-b5bd66db-wglzl   0/1     Evicted   0          61m
osm-controller-b5bd66db-wvl9w   1/1     Running   0          31m
```

Even though one controller has a status of `Evicted` at one point, another controller has the `READY` status `1/1` and `Running` with `0` restarts. If the `READY` status is anything other than `1/1`, the service mesh is in a broken state. If `READY` is `0/1`, the control plane container is crashing.

Use the following command to inspect controller logs:

```bash
kubectl logs -n arc-osm-system -l app=osm-controller
```

If the `READY` status is a number greater than `1` after the forward slash (`/`), sidecars are installed. The OSM controller generally doesn't work correctly when sidecars are attached.

### Check the OSM controller service

To check the OSM controller service, run this command:

```bash
kubectl get service -n arc-osm-system osm-controller
```

If the OSM controller is healthy, output that looks similar to the following example appears:

```output
NAME             TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)              AGE
osm-controller   ClusterIP   10.0.31.254   <none>        15128/TCP,9092/TCP   67m
```

> [!NOTE]
> The actual value for `CLUSTER-IP` will be different from this example. The values for `NAME` and `PORT(S)` should match what is shown in this example.

### Check OSM controller endpoints

```bash
kubectl get endpoints -n arc-osm-system osm-controller
```

If the OSM controller is healthy, output that looks similar to the following example appears:

```output
NAME             ENDPOINTS                              AGE
osm-controller   10.240.1.115:9092,10.240.1.115:15128   69m
```

If the cluster has no `ENDPOINTS` that have the value `osm-controller`, the control plane is unhealthy. This unhealthy state means that the controller pod crashed or that it was never deployed correctly.

### Check the OSM injector deployment

```bash
kubectl get deployments -n arc-osm-system osm-injector
```

If the OSM injector is healthy, output that looks similar to the following example appears:

```output
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
osm-injector   1/1     1            1           73m
```

### Check the OSM injector pod

```bash
kubectl get pod -n arc-osm-system --selector app=osm-injector
```

If the OSM injector is healthy, output that looks similar to the following example appears:

```output
NAME                            READY   STATUS    RESTARTS   AGE
osm-injector-5986c57765-vlsdk   1/1     Running   0          73m
```

The `READY` status must be `1/1`. Any other value indicates an unhealthy OSM injector pod.

### Check the OSM injector service

```bash
kubectl get service -n arc-osm-system osm-injector
```

If the OSM injector is healthy, output that looks similar to the following example appears:

```output
NAME           TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
osm-injector   ClusterIP   10.0.39.54   <none>        9090/TCP   75m
```

Ensure that the IP address listed for `osm-injector` service is `9090`. There should be no value listed for `EXTERNAL-IP`.

### Check OSM injector endpoints

```bash
kubectl get endpoints -n arc-osm-system osm-injector
```

If the OSM injector is healthy, output that looks similar to the following example appears:

```output
NAME           ENDPOINTS           AGE
osm-injector   10.240.1.172:9090   75m
```

For OSM to function, there must be at least one endpoint for `osm-injector`. The IP address of your OSM injector endpoints varies, but the port value `9090` must be the same.

### Check webhooks: Validating and Mutating

Check the **Validating** webhook by running the following command:

```bash
kubectl get ValidatingWebhookConfiguration --selector app=osm-controller
```

If the **Validating** webhook is healthy, output that looks similar to the following example appears:

```output
NAME                     WEBHOOKS   AGE
osm-validator-mesh-osm   1          81m
```

Check the **Mutating** webhook by running the following command:

```bash
kubectl get MutatingWebhookConfiguration --selector app=osm-injector
```

If the **Mutating** webhook is healthy, output that looks similar to the following example appears:

```output
NAME                  WEBHOOKS   AGE
arc-osm-webhook-osm   1          102m
```

Check for the service and the Certificate Authority bundle (CA bundle) of the **Validating** webhook by using this command:

```bash
kubectl get ValidatingWebhookConfiguration osm-validator-mesh-osm -o json | jq '.webhooks[0].clientConfig.service'
```

A well-configured **Validating** webhook has output that looks similar to this example:

```json
{
  "name": "osm-config-validator",
  "namespace": "arc-osm-system",
  "path": "/validate",
  "port": 9093
}
```

Check for the service and the CA bundle of the **Mutating** webhook by using the following command:

```bash
kubectl get MutatingWebhookConfiguration arc-osm-webhook-osm -o json | jq '.webhooks[0].clientConfig.service'
```

A well-configured **Mutating** webhook has output that looks similar to this example:

```output
{
  "name": "osm-injector",
  "namespace": "arc-osm-system",
  "path": "/mutate-pod-creation",
  "port": 9090
}
```

Check whether the OSM controller has given the **Validating** (or **Mutating**) webhook a CA bundle by using the following command:

```bash
kubectl get ValidatingWebhookConfiguration osm-validator-mesh-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

```bash
kubectl get MutatingWebhookConfiguration arc-osm-webhook-osm -o json | jq -r '.webhooks[0].clientConfig.caBundle' | wc -c
```

Example output:

```bash
1845
```

The number in the output indicates the number of bytes, or the size of the CA bundle. If the output is empty, 0, or a number under 1,000, the CA bundle isn't correctly provisioned. Without a correct CA bundle, `ValidatingWebhook` throws an error.

### Check the `osm-mesh-config` resource

Check for the existence of the resource:

```azurecli-interactive
kubectl get meshconfig osm-mesh-config -n arc-osm-system
```

Check the value of the OSM `meshconfig` setting:

```azurecli-interactive
kubectl get meshconfig osm-mesh-config -n arc-osm-system -o yaml
```

Look for output that looks similar to this example:

```yaml
apiVersion: config.openservicemesh.io/v1alpha1
kind: MeshConfig
metadata:
  creationTimestamp: "0000-00-00A00:00:00A"
  generation: 1
  name: osm-mesh-config
  namespace: arc-osm-system
  resourceVersion: "2494"
  uid: 6c4d67f3-c241-4aeb-bf4f-b029b08faa31
spec:
  certificate:
    certKeyBitSize: 2048
    serviceCertValidityDuration: 24h
  featureFlags:
    enableAsyncProxyServiceMapping: false
    enableEgressPolicy: true
    enableEnvoyActiveHealthChecks: false
    enableIngressBackendPolicy: true
    enableMulticlusterMode: false
    enableRetryPolicy: false
    enableSnapshotCacheMode: false
    enableWASMStats: true
  observability:
    enableDebugServer: false
    osmLogLevel: info
    tracing:
      enable: false
  sidecar:
    configResyncInterval: 0s
    enablePrivilegedInitContainer: false
    logLevel: error
    resources: {}
  traffic:
    enableEgress: false
    enablePermissiveTrafficPolicyMode: true
    inboundExternalAuthorization:
      enable: false
      failureModeAllow: false
      statPrefix: inboundExtAuthz
      timeout: 1s
    inboundPortExclusionList: []
    outboundIPRangeExclusionList: []
    outboundPortExclusionList: []
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
```

The following table lists `osm-mesh-config` resource values:

| Key | Type | Default value | Kubectl patch command examples |
|-----|------|---------------|--------------------------------|
| `spec.traffic.enableEgress` | bool | `false` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"traffic":{"enableEgress":false}}}'  --type=merge` |
| `spec.traffic.enablePermissiveTrafficPolicyMode` | bool | `true` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"traffic":{"enablePermissiveTrafficPolicyMode":true}}}'  --type=merge` |
| `spec.traffic.outboundPortExclusionList` | array | `[]` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"traffic":{"outboundPortExclusionList":[6379,8080]}}}'  --type=merge` |
| `spec.traffic.outboundIPRangeExclusionList` | array | `[]` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"traffic":{"outboundIPRangeExclusionList":["10.0.0.0/32","1.1.1.1/24"]}}}'  --type=merge` |
| `spec.traffic.inboundPortExclusionList` | array | `[]` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"traffic":{"inboundPortExclusionList":[6379,8080]}}}'  --type=merge` |
| `spec.certificate.serviceCertValidityDuration` | string | `"24h"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"certificate":{"serviceCertValidityDuration":"24h"}}}'  --type=merge` |
| `spec.observability.enableDebugServer` | bool | `false` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"observability":{"enableDebugServer":false}}}'  --type=merge` |
| `spec.observability.osmLogLevel` | string | `"info"`| `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"observability":{"tracing":{"osmLogLevel": "info"}}}}'  --type=merge` |
| `spec.observability.tracing.enable` | bool | `false` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"observability":{"tracing":{"enable":true}}}}'  --type=merge` |
| `spec.sidecar.enablePrivilegedInitContainer` | bool | `false` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"sidecar":{"enablePrivilegedInitContainer":true}}}'  --type=merge` |
| `spec.sidecar.logLevel` | string | `"error"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"sidecar":{"logLevel":"error"}}}'  --type=merge` |
| `spec.featureFlags.enableWASMStats` | bool | `"true"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableWASMStats":"true"}}}'  --type=merge` |
| `spec.featureFlags.enableEgressPolicy` | bool | `"true"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableEgressPolicy":"true"}}}'  --type=merge` |
| `spec.featureFlags.enableMulticlusterMode` | bool | `"false"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableMulticlusterMode":"false"}}}'  --type=merge` |
| `spec.featureFlags.enableSnapshotCacheMode` | bool | `"false"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableSnapshotCacheMode":"false"}}}'  --type=merge` |
| `spec.featureFlags.enableAsyncProxyServiceMapping` | bool | `"false"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableAsyncProxyServiceMapping":"false"}}}'  --type=merge` |
| `spec.featureFlags.enableIngressBackendPolicy` | bool | `"true"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableIngressBackendPolicy":"true"}}}'  --type=merge` |
| `spec.featureFlags.enableEnvoyActiveHealthChecks` | bool | `"false"` | `kubectl patch meshconfig osm-mesh-config -n arc-osm-system -p '{"spec":{"featureFlags":{"enableEnvoyActiveHealthChecks":"false"}}}'  --type=merge` |

### Check namespaces

> [!NOTE]
> The `arc-osm-system` namespace never participates in a service mesh and is never labeled or annotated with the key/value pairs shown here.

You can use the `osm namespace add` command to join namespaces to a specific service mesh. When a Kubernetes namespace is part of the mesh, complete the following steps to confirm that requirements are met.

View the annotations of the `bookbuyer` namespace:

```bash
kubectl get namespace bookbuyer -o json | jq '.metadata.annotations'
```

The following annotation must be present:

```bash
{
  "openservicemesh.io/sidecar-injection": "enabled"
}
```

View the labels of the `bookbuyer` namespace:

```bash
kubectl get namespace bookbuyer -o json | jq '.metadata.labels'
```

The following label must be present:

```bash
{
  "openservicemesh.io/monitored-by": "osm"
}
```

If you aren't using the `osm` CLI, you can manually add these annotations to your namespaces. If a namespace isn't annotated with `"openservicemesh.io/sidecar-injection": "enabled"`, or if it isn't labeled with `"openservicemesh.io/monitored-by": "osm"`, the OSM injector doesn't add Envoy sidecars.

> [!NOTE]
> After `osm namespace add` is called, only *new* pods are injected with an Envoy sidecar. Existing pods must be restarted by using the `kubectl rollout restart deployment` command.

### Verify the SMI CRDs

For the OSM Service Mesh Interface (SMI), check whether the cluster has the required custom resource definitions (CRDs):

```bash
kubectl get crds
```

Ensure that the CRDs correspond to the versions that are available in the release branch. To confirm which CRD versions are in use, see [SMI supported versions](https://docs.openservicemesh.io/docs/overview/smi/) and select your version in the **Releases** menu.

Get the versions of the installed CRDs by using the following command:

```bash
for x in $(kubectl get crds --no-headers | awk '{print $1}' | grep 'smi-spec.io'); do
    kubectl get crd $x -o json | jq -r '(.metadata.name, "----" , .spec.versions[].name, "\n")'
done
```

If CRDs are missing, use the following commands to install them on the cluster. Replace the version in these commands as needed (for example, instead of **v1.1.0**, you might use **release-v1.1**).

```bash
kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v1.0/cmd/osm-bootstrap/crds/smi_http_route_group.yaml

kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v1.0/cmd/osm-bootstrap/crds/smi_tcp_route.yaml

kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v1.0/cmd/osm-bootstrap/crds/smi_traffic_access.yaml

kubectl apply -f https://raw.githubusercontent.com/openservicemesh/osm/release-v1.0/cmd/osm-bootstrap/crds/smi_traffic_split.yaml
```

To see how CRD versions change between releases, see the [OSM release notes](https://github.com/openservicemesh/osm/releases).

### Troubleshoot certificate management

For information on how OSM issues and manages certificates to Envoy proxies running on application pods, see the [OSM documentation](https://docs.openservicemesh.io/docs/guides/certificates/).

### Upgrade Envoy

When a new pod is created in a namespace that's monitored by the add-on, OSM injects an [Envoy proxy sidecar](https://docs.openservicemesh.io/docs/guides/app_onboarding/sidecar_injection/) in that pod. If the Envoy version needs to be updated, follow the steps in the [Upgrade Guide](https://docs.openservicemesh.io/docs/guides/upgrade/#envoy) in the OSM documentation.

## Azure Fleet Manager

Depending upon your cluster configuration, and or environment, you may experience connection issues between the Azure Fleet Manager's dataplane components (agents) and the Fleet's hub cluster. Review Azure Fleet Manager's [important considerations for Arc-Enabled Kubernetes cluster members](/azure/kubernetes-fleet/concepts-member-cluster-types#arc-enabled-kubernetes-clusters-important-considerations) for a complete list of requirements and considerations.

## Related content

- Learn more about [cluster extensions](conceptual-extensions.md).
- View general [troubleshooting tips for Arc-enabled Kubernetes clusters](extensions-troubleshooting.md).
