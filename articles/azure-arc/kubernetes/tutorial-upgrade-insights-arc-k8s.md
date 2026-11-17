---
# Required metadata
# For more information, see https://learn.microsoft.com/en-us/help/platform/learn-editor-add-metadata
# For valid values of ms.service, ms.prod, and ms.topic, see https://learn.microsoft.com/en-us/help/platform/metadata-taxonomies

title: Get Upgrade Insights for Azure Arc-enabled Kubernetes clusters
description: Get Upgrade Insights for Azure Arc-enabled Kubernetes clusters
author:      ponatara # GitHub alias
ms.author:   ponatara # Microsoft alias
ms.service: azure-arc
ms.topic: tutorial
ms.date:     11/16/2026
---

# Tutorial: Assess Kubernetes upgrade readiness with Upgrade Insights

> [!IMPORTANT]
> Kubernetes Upgrade Insights is currently in **public preview**. Preview features are provided under the [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Kubernetes upgrades can expose application and cluster compatibility issues that are difficult to identify in advance. An API used by a workload might be removed in the target Kubernetes version, an installed component might not support the target version, or the cluster might have a health condition that could block the upgrade.

Kubernetes Upgrade Insights for Azure Arc-enabled Kubernetes provides a proactive, read-only assessment to help you identify these risks before you begin an upgrade. It evaluates your cluster for deprecated Kubernetes API usage, component compatibility, and upgrade readiness conditions. Upgrade Insights provides assessment findings and recommendations only. It doesn't initiate the Kubernetes upgrade or automatically remediate issues.

In this tutorial, you learn how to:

> [!div class="checklist"]
> - Verify the prerequisites for Upgrade Insights.
> - Enable Upgrade Insights from the Azure portal or Azure CLI.
> - Confirm that the extension is installed and producing assessments.
> - Review findings in the Azure portal.
> - Import and use the optional Azure Monitor dashboard with Grafana.
> - Query assessment results across clusters with Azure Resource Graph.
> - Interpret findings and prepare for a Kubernetes upgrade.

## How Upgrade Insights works

Upgrade Insights is deployed as an Azure Arc-enabled Kubernetes cluster extension. After installation, the extension collects signals relevant to upgrade readiness and assesses the cluster against the next applicable minor Kubernetes version identified for the cluster.

The extension performs an initial assessment after installation and refreshes supported assessments daily. You can review the results in the Azure portal or query them at scale by using Azure Resource Graph. The extension publishes the results as Azure resources in your tenant.

Upgrade Insights evaluates three areas:

| Assessment area | What it identifies |
|---|---|
| Deprecated Kubernetes APIs | API versions used by workloads, controllers, clients, Helm charts, or automation that are deprecated or removed in the target Kubernetes version. |
| Component compatibility | Installed component versions that might be incompatible with, untested against, or undetermined for the target Kubernetes version.|
| Upgrade readiness | Current cluster health or configuration conditions that might block or disrupt an upgrade, including Pod Disruption Budgets, node readiness, and API server health. |

The component compatibility assessment currently evaluates a defined set of common components listed in the following table:

| Component category | Components evaluated |
|---|---|
| Cluster autoscaling | Cluster Autoscaler |
| Service mesh | Istio, Linkerd |
| Monitoring | Prometheus |
| Ingress | NGINX, Traefik |
| Cluster datastore | etcd |
| Container runtimes (CRI) | containerd, CRI-O |
| Cluster DNS | CoreDNS |
| Container networking (CNI) | Calico, Cilium, Flannel |
| Certificate management | cert-manager |
| Azure Arc agents | Azure Arc agents |

> [!NOTE]
> Upgrade Insights doesn't replace your Kubernetes distribution's upgrade documentation or application-specific testing. The assessment covers a defined set of components and readiness conditions and might not identify every compatibility issue.

## Prerequisites

Before you begin, ensure that you have:

- An existing Kubernetes cluster connected to Azure Arc. To connect a cluster, see [Quickstart: Connect an existing Kubernetes cluster to Azure Arc](https://learn.microsoft.com/azure/azure-arc/kubernetes/quickstart-connect-cluster?tabs=azure-cli).
- Access to Azure Public Cloud.
- Azure RBAC permissions to install cluster extension on the target cluster and view assessment results.
- Azure CLI, if you plan to use the command-line steps.
- The Kubernetes Upgrade Insights extension version 1.0.2 or later.

If you use Azure CLI, sign in and select the subscription that contains the Arc-enabled Kubernetes resource:

```azurecli
az login \
az account set --subscription <subscription-id>
```

## Enable Upgrade Insights

You can enable Upgrade Insights from the Azure portal or by using Azure CLI.

### Option 1: Enable Upgrade Insights in the Azure portal

1. Sign in to the [Azure portal](https://portal.azure.com/).
1. Navigate to **Azure Arc**.
1. Under **Infrastructure**, select **Kubernetes clusters**.
1. Select the target cluster or clusters.
1. Verify that each cluster has a **Connected** status.
1. Select **Enable Upgrade Insights**.
1. Follow the portal prompts to deploy the extension.


### Option 2: Enable Upgrade Insights with Azure CLI

Run the following command:

```azurecli
az k8s-extension create \
  --cluster-type connectedClusters \
  --cluster-name <cluster-name> \
  --resource-group <resource-group> \
  --name <extension-name> \
  --extension-type microsoft.kubernetesupgradeinsights
```

The extension deploys Upgrade Insights components in the `k8s-upgrade-insights` namespace. These components collect the signals required for the read-only assessment and publish the assessment results to Azure.

## Verify the extension installation

Check the extension provisioning state:

```azurecli
az k8s-extension show \
  --cluster-type connectedClusters \
  --cluster-name <cluster-name> \
  --resource-group <resource-group> \
  --name <extension-name> \
  --query provisioningState \
  --output tsv
```

The expected value is `Succeeded`.

You can also verify that the extension pods are running:

```bash
kubectl get pods -n k8s-upgrade-insights
```

Allow time for the initial assessment to complete and publish results. After the initial assessment, supported findings are refreshed daily.

## Review Upgrade Insights in the Azure portal

1. In the Azure portal, go to **Azure Arc** > **Kubernetes clusters**.
1. Select the cluster where you enabled Upgrade Insights.
1. Open **Upgrade Insights**.
1. Review the current Kubernetes version, target Kubernetes version, assessment time, and summary of critical and warning findings.
1. Review each assessment category:
   - **Deprecated API usage**
   - **Component compatibility**
   - **Upgrade readiness**
1. Select a finding to view the affected resource or component, severity, details, and recommended remediation.


### Understand the findings

#### Deprecated API usage

For each deprecated API, Upgrade Insights shows information such as:

- Affected cluster.
- Current and target Kubernetes versions.
- API group, version, and resource.
- Kubernetes version in which the API is removed.
- Severity.
- Recently observed usage and request activity.

A **Critical** finding means that the API version is removed in the target Kubernetes version. A **Warning** means that the API is deprecated but remains available in the target version and is scheduled for removal in a later release.

Update workloads, controllers, manifests, Helm charts, and automation that use an API removed in the target Kubernetes version before you upgrade.

#### Component compatibility

For detected components, Upgrade Insights shows the installed version, compatibility status, severity, and a recommended action.

| Compatibility status | Meaning |
|---|---|
| Compatible | The detected component version is compatible with the target Kubernetes version. |
| Incompatible | The detected component version isn't compatible with the target version. Review the recommended minimum or supported version. |
| Not validated | Compatibility with the target version hasn't been validated. Investigate before upgrading. |
| Undetermined | Compatibility couldn't be determined from the available information. Review the component vendor's guidance. |

#### Upgrade readiness

Readiness checks evaluate the cluster's current health and configuration. Current checks include:

- **Pod Disruption Budgets:** Identifies configurations that might prevent pod eviction and block node drain.
- **Node readiness:** Identifies nodes that aren't reporting a ready status.
- **API server health:** Identifies API server health conditions that might prevent an upgrade from starting or completing successfully.

Resolve critical readiness findings before you begin the Kubernetes upgrade.

## Review fleet-level results in the Azure portal

Use the fleet-level multi-cluster Upgrade Insights experience to understand assessment coverage and findings across multiple Azure Arc-enabled Kubernetes clusters.

1. In the Azure portal, go to the Upgrade Insights fleet or multi-cluster experience.
1. Review the number of clusters with Upgrade Insights enabled and the number of clusters with available assessments.
1. Filter results by subscription, resource group, cluster, location, current Kubernetes version, target Kubernetes version, or severity.
1. Select a critical or warning summary to view the affected clusters and findings.


## Import the optional Grafana dashboard

You can keep the Azure Monitor dashboard with Grafana as an optional fleet visualization and troubleshooting experience. The dashboard uses Upgrade Insights assessment resources in Azure Resource Graph and provides summary metrics, filters, and detailed tables.

You don't need an Azure Managed Grafana workspace to use the provided dashboard with **Azure Monitor > Dashboards with Grafana**.

1. Download or locate the provided `ArcKubernetesUpgradeInsights.json` dashboard file.
1. In the [Azure portal](https://portal.azure.com/), go to **Azure Monitor**.
1. Select **Dashboards with Grafana**.
1. Select **New** > **Import**.
1. Upload `ArcKubernetesUpgradeInsights.json`.
1. Enter a dashboard name.
1. Select the subscription, resource group, and region where you want to save the dashboard resource.
1. Select **Import**.
1. Use the dashboard filters to select subscriptions, resource groups, clusters, locations, current Kubernetes versions, and target Kubernetes versions.


The dashboard includes views for:

- Total critical and warning issues.
- Clusters with assessment reports.
- Deprecated API summaries and details.
- Component compatibility summaries and recommendations.
- Pod Disruption Budget findings.
- Node readiness.
- API server health.

## Query results with Azure Resource Graph

Upgrade Insights publishes assessment resources with the following resource type:

`microsoft.kubernetesconfiguration/upgradeassessments`

### Query in Resource Graph Explorer

1. In the Azure portal, open **Resource Graph Explorer**.
1. Run the following query:

```kusto
KubernetesConfigurationResources
| where type =~ "microsoft.kubernetesconfiguration/upgradeassessments"
```
### Query with Azure CLI

Run the query:

```azurecli
az graph query -q "KubernetesConfigurationResources | where type =~ 'microsoft.kubernetesconfiguration/upgradeassessments'"
```

### Summarize critical and warning findings

The following query combines the three assessment categories and summarizes findings by severity:

```kusto
KubernetesConfigurationResources
| where type =~ "microsoft.kubernetesconfiguration/upgradeassessments"
| extend clusterName = tostring(split(id, "/")[8])
| extend findings = array_concat(
    properties.componentCompatibility,
    properties.deprecatedApis,
    properties.readinessChecks)
| mv-expand finding = findings
| summarize
    Critical = countif(tostring(finding.severity) =~ "Critical"),
    Warning = countif(tostring(finding.severity) =~ "Warning")
  by subscriptionId, resourceGroup, clusterName,
     currentVersion = tostring(properties.currentKubernetesVersion),
     targetVersion = tostring(properties.targetKubernetesVersion)
| order by Critical desc, Warning desc
```

## Remediate findings and prepare for the upgrade

Use the assessment as an input to your normal upgrade process:

1. Address deprecated APIs that are removed in the target Kubernetes version.
1. Upgrade or replace incompatible cluster components.
1. Investigate components with unvalidated or undetermined compatibility.
1. Resolve Pod Disruption Budget, node readiness, and API server health findings.
1. Recheck the portal or Azure Resource Graph after the next daily assessment.
1. Complete application, integration, backup, and recovery testing required by your organization.
1. Follow your Kubernetes distribution's upgrade guidance to perform the upgrade.

> [!IMPORTANT]
> Upgrade Insights is advisory. It doesn't schedule or perform upgrades, update APIs or components, change Pod Disruption Budgets, or repair unhealthy nodes.

## Troubleshooting

### No assessment results appear

If you don't see assessment results:

1. Verify that the extension provisioning state is `Succeeded`.
1. Verify that the pods in the `k8s-upgrade-insights` namespace are running.
1. Confirm that you have permission to read the Arc-enabled Kubernetes resource and Upgrade Insights assessment resources.
1. Allow time for the initial assessment to complete.
1. Query Azure Resource Graph directly to determine whether results are being published.

### The portal shows partial assessment coverage

Partial coverage can occur when Upgrade Insights isn't enabled on every selected cluster, the initial assessment hasn't completed, or a cluster isn't connected. Review the coverage information, verify the extension status on affected clusters, and check the assessment timestamp.

### A component is shown as not validated or undetermined

Review the component vendor's compatibility guidance for the target Kubernetes version. Upgrade Insights surfaces the available compatibility evidence but doesn't replace component-specific release notes and support policies.

## Current limitations

- You can't manually trigger an on-demand assessment.
- The assessment evaluates a defined set of components and readiness conditions.
- Findings don't replace distribution-specific compatibility checks, application testing, or upgrade documentation.
- Upgrade Insights doesn't guarantee that every application or component compatibility issue will be identified.

## Clean up resources

If you created the optional Grafana dashboard only for this tutorial, you can delete the dashboard resource from its resource group.

To remove the Upgrade Insights extension from a cluster, run:

```azurecli
az k8s-extension delete \
  --cluster-type connectedClusters \
  --cluster-name <cluster-name> \
  --resource-group <resource-group> \
  --name <extension-name> \
  --yes
```

Removing the extension stops future assessments for that cluster.

## Next steps

- [Deploy and manage Azure Arc-enabled Kubernetes cluster extensions](https://learn.microsoft.com/azure/azure-arc/kubernetes/extensions)
- [Run your first Azure Resource Graph query using Azure CLI](https://learn.microsoft.com/azure/governance/resource-graph/first-query-azurecli)
- Review your Kubernetes distribution's documentation for the target version and complete the upgrade after remediating identified risks.


