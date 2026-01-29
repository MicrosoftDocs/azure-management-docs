---
title: Use Microsoft Entra ID with the Azure mobile app
description: Use the Azure mobile app to manage users and groups with Microsoft Entra ID.
ms.date: 05/15/2025
ms.topic: how-to
ms.custom:
  - build-2025
# Customer intent: "As an IT administrator managing Azure users remotely, I want to use the mobile app to manage user accounts and groups in Microsoft Entra ID, so that I can maintain user access and security efficiently from my mobile device."
---

# Use Microsoft Entra ID with the Azure mobile app

The Azure mobile app provides access to Microsoft Entra ID. You can perform tasks such as managing users and updating group memberships from within the app.

To access Microsoft Entra ID, open the Azure mobile app and sign in with your Azure account. From **Home**, scroll down to select the **Microsoft Entra ID** card.

> [!NOTE]
> Your account must have the appropriate permissions in order to perform these tasks. For example, to invite a user to your tenant, you must have a role that includes this permission, such as [Guest Inviter or User Administrator](/entra/identity/role-based-access-control/permissions-reference).

## Invite a user to the tenant

To invite a [guest user](/entra/external-id/what-is-b2b) to your tenant from the Azure mobile app:

1. In **Microsoft Entra ID**, select **Users**, then select the **+** icon in the top right corner.
1. Select **Invite a user**, then enter the user's name and email address. You can optionally add a message for the user.
1. Select **Invite** in the top right corner, then select **Save** to confirm your changes.

## Add users to a group

To add one or more users to a group from the Azure mobile app:

1. In **Microsoft Entra ID**, select **Groups**.
1. Search or scroll to find the desired group, then tap to select it.
1. On the **Members** card, select **See All**. The current list of members is displayed.
1. Select the **+** icon in the top right corner.
1. Search or scroll to find users you want to add to the group, then select one or more users by tapping the circle next to their name.
1. Select **Add** in the top right corner to add the selected users to the group.

## Edit profile details for a user

To edit a user’s profile details in the Azure mobile app:

1. In **Microsoft Entra ID**, select **Users**.
1. Search or scroll to find the desired user, then tap to select their profile.
1. On the **Profile** card, select **Details**. The selected profile's details are displayed.
1. Select the edit (pencil) icon in the top right corner. If you don't have permission to edit a profile, you'll see a message letting you know that you can't edit the profile. Otherwise, you can proceed to make changes.
1. To update an editable item, tap the item and enter your changes.
1. Once you finish making your updates, tap **Save**.

## Add group memberships for a specified user

You can also add a single user to one or more groups in the **Users** section of **Microsoft Entra ID** in the Azure mobile app. To do so:

1. In **Microsoft Entra ID**, select **Users**, then search or scroll to find and select the desired user.
1. On the **Groups** card, select **See All** to display all current group memberships for that user.
1. Select the **+** icon in the top right corner.
1. Search or scroll to find groups to which this user should be added, then select one or more groups by tapping the circle next to the group name.
1. Select **Add** in the top right corner to add the user to the selected groups.

## Manage authentication methods or reset password for a user

To [manage authentication methods](/entra/identity/authentication/concept-authentication-methods-manage) or [reset a user's password](/entra/fundamentals/users-reset-password-azure-portal):

1. In **Microsoft Entra ID**, select **Users**, then search or scroll to find and select the desired user.
1. On the **Authentication methods** card, select **Manage**.
1. Select **Reset password** to assign a temporary password to the user, or **Authentication methods** to manage authentication methods for self-service password reset.

> [!NOTE]
> You won't see the **Authentication methods** card if you don't have the appropriate permissions to manage authentication methods and/or password changes for a user.

## Investigate risky users and sign-ins

[Microsoft Entra ID Protection](/entra/id-protection/overview-identity-protection) provides organizations with reporting they can use to [investigate identity risks in their environment](/entra/id-protection/howto-identity-protection-investigate-risk).

If you have the [necessary permissions and license](/entra/id-protection/overview-identity-protection#required-roles), you see details in the **Risky users** and **Risky sign-ins** sections within **Microsoft Entra ID**. You can open these sections to view more information and perform some management tasks.

### Manage risky users

1. In **Microsoft Entra ID**, scroll down to the **Security** card and then select **Risky users**.
1. Search or scroll to find and select a specific risky user.
1. Review basic information for this user, a list of their risky sign-ins, and their risk history.
1. To [take action on the user](/entra/id-protection/howto-identity-protection-investigate-risk), select the three dots near the top of the screen. You can:

   - Reset the user's password
   - Confirm user compromise
   - Dismiss user risk
   - Block the user from signing in (or unblock, if previously blocked)

### Monitor risky sign-ins

1. In **Microsoft Entra ID**, scroll down to the **Security** card and then select **Risky sign-ins**. It may take a minute or two for the list of all risky sign-ins to load.

1. Search or scroll to find and select a specific risky sign-in.

1. Review details about the risky sign-in.

## Activate Privileged Identity Management (PIM) roles

If you are eligible for an administrative role through Microsoft Entra Privileged Identity Management (PIM), you must activate the role assignment when you need to perform privileged actions. This activation can be done from within the Azure mobile app by selecting the **Privileged Identity Management** card on the **Home** page.

For more information, see [Activate PIM roles using the Azure mobile app](/entra/id-governance/privileged-identity-management/pim-how-to-activate-role#activate-pim-roles-using-the-azure-mobile-app).

## Next steps

- Learn more about the [Azure mobile app](overview.md).
- Download the Azure mobile app for free from the [Apple App Store](https://aka.ms/azureapp/ios/doc) or [Google Play](https://aka.ms/azureapp/android/doc)..
