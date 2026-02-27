---
title: Set Up Conditional Access for Azure Container Registry
description: Learn how to configure conditional access to your Azure Container Registry by using Azure CLI and Azure portal to enhance security.
ms.author: rayoflores
ms.service: azure-container-registry
ms.custom: devx-track-azurecli
ms.topic: how-to
ms.date: 02/27/2026
# Customer intent: As a DevOps engineer, I want to configure a Conditional Access policy for Azure Container Registry so that I can enhance security and enforce strong user authentication and access controls for my Azure services.
---

# Conditional Access policy for Azure Container Registry

Azure Container Registry (ACR) gives you the option to create and configure [Microsoft Entra Conditional Access policies](/entra/identity/conditional-access/overview) to enforce strong authentication and access controls.

The Conditional Access policy is designed to enforce strong authentication. The policy helps your organization meet compliance requirements and keeps data and user accounts safe.

Conditional access policies apply after a user authenticates to the Azure Container Registry. You can configure options to block or grant access based on your [policy decisions](/azure/active-directory/conditional-access/overview#common-decisions).

Azure Container Registry supports Conditional Access policy for Microsoft Entra user accounts only. Conditional Access policy for service principal accounts isn't currently supported.

>[!IMPORTANT]
> To configure Conditional Access policy for the registry, first configure all registries in your tenant to [accept only ACR-scoped Microsoft Entra authentication](container-registry-disable-authentication-as-arm.md) for all the registries within the desired tenant.

This article shows you how to create and configure a Conditional Access policy for Azure Container Registry, and how to troubleshoot common issues.

## Prerequisites

* An Azure account with the [Conditional Access Administrator](/entra/identity/role-based-access-control/permissions-reference#conditional-access-administrator) role.
* An Azure Container Registry. If you don't have a registry, create one by following the [Quickstart: Create a container registry using the Azure portal](/azure/container-registry/container-registry-get-started-portal) tutorial.

## Create and configure a Conditional Access policy

Create a Conditional Access policy and assign your test group of users as follows:

   1. Sign in to the [Azure portal](https://portal.azure.com) with an account that has the [Conditional Access Administrator](/entra/identity/role-based-access-control/permissions-reference#conditional-access-administrator) role.

   1. Search for and select **Microsoft Entra ID**.

   1. In the service menu, under **Manage**, select **Security**.

   1. Select **Conditional Access**, select **+ New policy**, and then select **Create new policy**.

   1. Enter a name for the policy, such as *demo*.

   1. Under **What does this policy apply to?**, select **Users and groups**.

   1. Under **Include**, choose **Select users and groups**, and then select **All users**.

      :::image type="content" alt-text="A screenshot of the page for creating a new policy, where you select options to specify users." source="media/container-registry-enable-conditional-policy/03-conditional-access-users-groups-select-users.png":::

   1. Under **Exclude**, choose **Select users and groups**, then make any selections for users or groups to exclude from the policy.

   1. Under **Cloud apps or actions**, choose **Cloud apps**.

   1. Under **Include**, choose **Select apps**.

      :::image type="content" alt-text="A screenshot of the page for creating a new policy, where you select options to specify cloud apps." source="media/container-registry-enable-conditional-policy/04-select-cloud-apps-select-apps.png":::

   1. Select *Azure Container Registry*, then choose **Select**.

   1. Under **Conditions**, set the [conditions](/entra/identity/conditional-access/concept-conditional-access-conditions) you want to apply, such as *User risk level*, *Sign-in risk level*, *Sign-in risk detections (Preview)*, *Device platforms*, *Locations*, *Client apps*, *Time (Preview)*, *Filter for devices*.

   1. Under **Access controls**, select **Grant**. Select **Grant access** with **Require multifactor authentication**, then choose **Select**.

      >[!TIP]
      > For details about how to configure and grant multifactor authentication, see [Configure the conditions for multifactor authentication.](/entra/identity/authentication/tutorial-enable-azure-mfa#configure-the-conditions-for-multi-factor-authentication)

   1. Under **Session**, optionally choose options to enable any control on session level experience of the cloud apps.

   1. Set **Enable policy** to**On**, then select **Create**

   You have now created a Conditional Access policy for your Azure Container Registry.

## Troubleshoot Conditional Access policy problems

For problems with Conditional Access sign-in, see [Troubleshoot Conditional Access sign-in](/entra/identity/conditional-access/troubleshoot-conditional-access).

For problems with Conditional Access policy, see [Troubleshoot Conditional Access policy](/entra/identity/conditional-access/troubleshoot-conditional-access-what-if).

## Next steps

* Learn more about [Azure Policy definitions](/azure/governance/policy/concepts/definition-structure) and [effects](/azure/governance/policy/concepts/effects).
* Review the > [Conditional Access policy components](/azure/active-directory/conditional-access/concept-conditional-access-policies).
* Learn about [common access concerns that Conditional Access policies can help with](/azure/active-directory/conditional-access/concept-conditional-access-policy-common).
