---
title: "Azure RBAC on Azure Arc-enabled Kubernetes"
ms.date: 04/22/2025
ms.topic: concept-article
description: "This article provides a conceptual overview of the Azure RBAC capability on Azure Arc-enabled Kubernetes."
# Customer intent: As a Kubernetes administrator, I want to implement Azure RBAC on my Azure Arc-enabled clusters so that I can manage user access and authorization efficiently, ensuring secure operations and compliance across my resources.
---

# Azure RBAC on Azure Arc-enabled Kubernetes clusters

[Microsoft Entra ID ](/entra/fundamentals/whatis)and [Azure role-based access control (Azure RBAC)](/azure/role-based-access-control/overview) let you control authorization checks on your Azure Arc-enabled Kubernetes cluster. Using Azure RBAC with your cluster gives you the benefits of Azure role assignments, such as activity logs showing changes made by users to your Azure resource. 

## Architecture

:::image type="content" source="media/conceptual-azure-rbac.png" alt-text="Diagram showing Azure RBAC architecture.":::

[!INCLUDE [arc-jumpstart-diagram](~/reusable-content/ce-skilling/azure/includes/arc-jumpstart-diagram.md)]

In order to route all authorization access checks to the authorization service in Azure, a webhook server ([guard](https://github.com/appscode/guard)) is deployed on the cluster.

The `apiserver` of the cluster is configured to use [webhook token authentication](https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication) and [webhook authorization](https://kubernetes.io/docs/reference/access-authn-authz/webhook/) so that `TokenAccessReview` and `SubjectAccessReview` requests are routed to the guard webhook server. The `TokenAccessReview` and `SubjectAccessReview` requests are triggered by requests for Kubernetes resources sent to the `apiserver`.

Guard then makes a `checkAccess` call on the authorization service in Azure to see if the requesting Microsoft Entra entity has access to the resource of concern.

If that entity has a role that permits this access, an `allowed` response is sent from the authorization service to guard. Guard, in turn, sends an `allowed` response to the `apiserver`, enabling the calling entity to access the requested Kubernetes resource.

If the entity doesn't have a role that permits this access, a `denied` response is sent from the authorization service to guard. Guard sends a `denied` response to the `apiserver`, giving the calling entity a 403 forbidden error on the requested resource.

## Enable Azure RBAC on your Arc-enabled Kubernetes clusters

For detailed information about how to set up Azure RBAC and create role assignments for your clusters, see [Use Azure RBAC on Azure Arc-enabled Kubernetes clusters](azure-rbac.md).

## Next steps

* Use our quickstart to [connect a Kubernetes cluster to Azure Arc](./quickstart-connect-cluster.md).
* [Set up Azure RBAC](./azure-rbac.md) on your Azure Arc-enabled Kubernetes cluster.
* Help to protect your cluster in other ways by following the guidance in the [security book for Azure Arc-enabled Kubernetes](conceptual-security-book.md).
