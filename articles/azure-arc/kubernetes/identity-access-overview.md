---
title: "Azure Arc-enabled Kubernetes identity and access overview"
ms.date: 05/22/2024
ms.topic: concept-article
description: "Understand identity and access options for Arc-enabled Kubernetes clusters."
---

# Azure Arc-enabled Kubernetes identity and access overview

You can authenticate, authorize, and control access to your Azure Arc-enabled Kubernetes clusters. This topic provides an overview of the options for doing so with your Arc-enabled Kubernetes clusters.

This image shows the ways that these different options can be used:

:::image type="content" source="media/identity-access-overview/identity-access-options.png" alt-text="Diagram showing the different options for authenticating, authorizing, and controlling access to Arc-enabled Kubernetes clusters.":::

You can also use both cluster connect and Azure RBAC together if that is most appropriate for your needs.

## Connectivity options

When planning how users will authenticate and access Arc-enabled Kubernetes clusters, the first decision is whether or not you want to use the cluster connect feature.

### Cluster connect

The Azure Arc-enabled Kubernetes [cluster connect](conceptual-cluster-connect.md) feature provides connectivity to the `apiserver` of the cluster. This connectivity doesn't require any inbound port to be enabled on the firewall. A reverse proxy agent running on the cluster can securely start a session with the Azure Arc service in an outbound manner.

With cluster connect, your Arc-enabled clusters can be accessed either within Azure or from the internet. This feature can help enable interactive debugging and troubleshooting scenarios.  Cluster connect may also require less interaction for updates when permissions are needed for new users. All of the authorization and authentication options described in this article work with cluster connect.

Cluster connect is required if you want to use [custom locations](conceptual-custom-locations.md) or [viewing Kubernetes resources from Azure portal](kubernetes-resource-view.md).

For more information, see [Use cluster connect to securely connect to Azure Arc-enabled Kubernetes clusters](cluster-connect.md).

<a name='azure-ad-and-azure-rbac-without-cluster-connect'></a>

### Microsoft Entra ID and Azure RBAC without cluster connect

If you don't want to use cluster connect, you can authenticate and authorize users so they can access the connected cluster by using [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) and [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview). Using [Azure RBAC on Azure Arc-enabled Kubernetes](conceptual-azure-rbac.md) lets you control the access that's granted to users in your tenant, managing access directly from Azure using familiar Azure identity and access features. You can also configure roles at the subscription or resource group scope, letting them roll out to all connected clusters within that scope.

Azure RBAC supports [conditional access](azure-rbac.md#use-conditional-access-with-azure-ad), allowing you to enable [just-in-time cluster access](azure-rbac.md#configure-just-in-time-cluster-access-with-azure-ad) or limit access to approved clients or devices.

Azure RBAC also supports a [direct mode of communication](azure-rbac.md#use-a-shared-kubeconfig-file), using Microsoft Entra identities to access connected clusters directly from within the datacenter, rather than requiring all connections to go through Azure.

Azure RBAC on Arc-enabled Kubernetes is currently in public preview. For more information, see  [Use Azure RBAC on Azure Arc-enabled Kubernetes clusters](azure-rbac.md).

## Authentication options

Authentication is the process of verifying a user's identity. There are two options for authenticating to an Arc-enabled Kubernetes cluster: cluster connect and Azure RBAC.

<a name='azure-ad-authentication'></a>

### Microsoft Entra authentication

The [Azure RBAC on Arc-enabled Kubernetes](conceptual-azure-rbac.md) feature lets you use [Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) to allow users in your Azure tenant to access your connected Kubernetes clusters.

You can also use Microsoft Entra authentication with cluster connect. For more information, see [Microsoft Entra authentication option](cluster-connect.md#microsoft-entra-authentication-option).

### Service token authentication

With cluster connect, you can choose to authenticate via [service accounts](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#service-account-tokens).

For more information, see [Service account token authentication option](cluster-connect.md#service-account-token-authentication-option).

## Authorization options

Authorization grants an authenticated user the permission to perform specified actions. With Azure Arc-enabled Kubernetes, there are two authorization options, both of which use role-based access control (RBAC):

- [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview) uses Microsoft Entra ID and Azure Resource Manager to provide fine-grained access management to Azure resources. This allows the benefits of Azure role assignments, such as activity logs tracking all changes made, to be used with your Azure Arc-enabled Kubernetes clusters.
- [Kubernetes role-based access control (Kubernetes RBAC)](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) lets you dynamically configure policies through the Kubernetes API so that users, groups, and service accounts only have access to specific cluster resources.

While Kubernetes RBAC works only on Kubernetes resources within your cluster, Azure RBAC works on resources across your Azure subscription.

### Azure RBAC authorization

[Azure role-based access control (RBAC)](/azure/role-based-access-control/overview) is an authorization system built on Azure Resource Manager and Microsoft Entra ID that provides fine-grained access management of Azure resources. With Azure RBAC, role definitions outline the permissions to be applied. You assign these roles to users or groups via a role assignment for a particular scope. The scope can be across the entire subscription or limited to a resource group or to an individual resource such as a Kubernetes cluster.

If you're using Microsoft Entra authentication without cluster connect, then Azure RBAC authorization is your only option for authorization.

If you're using cluster connect with Microsoft Entra authentication, you have the option to use Azure RBAC for connectivity to the `apiserver` of the cluster. For more information, see [Microsoft Entra authentication option](cluster-connect.md#azure-active-directory-authentication-option).

### Kubernetes RBAC authorization

[Kubernetes RBAC](https://kubernetes.io/docs/reference/access-authn-authz/rbac/) provides granular filtering of user actions. With Kubernetes RBAC, you assign users or groups permission to create and modify resources or view logs from running application workloads. You can create roles to define permissions, and then assign those roles to users with role bindings. Permissions may be scoped to a single namespace or across the entire cluster.

If you're using cluster connect with the [service account token authentication option](cluster-connect.md#service-account-token-authentication-option), you must use Kubernetes RBAC to provide connectivity to the `apiserver` of the cluster. This connectivity doesn't require any inbound port to be enabled on the firewall. A reverse proxy agent running on the cluster can securely start a session with the Azure Arc service in an outbound manner.

If you're using [cluster connect with Microsoft Entra authentication](cluster-connect.md#azure-active-directory-authentication-option), you also have the option to use Kubernetes RBAC instead of Azure RBAC.

## Next steps

- Learn more about [Azure Microsoft Entra ID](/azure/active-directory/fundamentals/active-directory-whatis) and [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview).
- Learn about [cluster connect access to Azure Arc-enabled Kubernetes clusters](conceptual-cluster-connect.md).
- Learn about [Azure RBAC on Azure Arc-enabled Kubernetes](conceptual-azure-rbac.md)
- Learn about [access and identity options for Azure Kubernetes Service (AKS) clusters](/azure/aks/concepts-identity).
