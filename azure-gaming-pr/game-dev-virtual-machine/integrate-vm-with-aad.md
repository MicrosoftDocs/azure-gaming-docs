---
title: Integrate with Azure Active Directory
description: Integrate Azure Active Directory with your Game Development Virtual Machine during resource creation.
author: meaghanlewis
ms.topic: how-to
ms.date: 03/15/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Integrate with Azure Active Directory

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

You may integrate Azure Active Directory authentication with this Game Development Virtual Machine during resource creation.

## Enabling Azure AD

When you are creating an Azure Game Development Virtual Machine, under the **Management** tab, select the **Identity** and **Azure AD** options.

:::image type="content" source="./media/integrate-gm-with-add/enable-azure-ad.png" alt-text="Screenshot showing how to enable Azure AD when creating a new Game Development VM":::

The Identity option is a system assigned managed identity and is automatically enabled when you enable Azure AD. However, the system assigned identity may also be enabled independently without including Azure AD. It is a similar experience to what you do for a regular Azure VM on Azure portal after VM is created: [Using Azure portal create VM experience to enable Azure AD login](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#using-azure-portal-create-vm-experience-to-enable-azure-ad-login).

Two additional things need to be taken into consideration so that Azure AD is successfully enabled.

1. Although Azure AD can be enabled during the Game Development Virtual Machine creation, it is still necessary to configure Azure RBAC (role-based access control) policy to decide who can log in to the VM using Azure AD. Either Virtual Machine Administrator Login or Virtual Machine User Login role is needed on the game dev VM. Learn more about how to [configure role assignments for the VM](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#configure-role-assignments-for-the-vm).
1. Windows 10 or higher is required for you to start the remote desktop connection that are either Azure AD registered (minimum required build is 20H1), or Azure AD joined, or hybrid Azure AD joined to the same directory as the VM. Make sure to [Log in using Azure AD credentials to a Windows VM](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#log-in-using-azure-ad-credentials-to-a-windows-vm).

## Troubleshooting

If you have sign-in issues with Azure AD to the game dev VM, you can follow the [troubleshooting steps](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#troubleshoot-sign-in-issues).

## Next steps

Sign-in to this game development VM using your enterprise credentials with Azure AD and start exploring game development on Azure.

For more information about the Game Development Virtual Machine, see [What is the Azure Game Development Virtual Machine?](./overview.md).
