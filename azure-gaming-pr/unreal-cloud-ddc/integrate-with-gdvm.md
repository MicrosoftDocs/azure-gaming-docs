---
title: Connect from Game Development VM
description: Connect to Unreal Cloud DDC from a Game Development VM
author: dciborow
ms.topic: how-to
ms.date: 10/02/2022
ms.author: dciborow
ms.service: azure-gaming
---
# Connect to Unreal Cloud DDC from a Game Development VM

[!INCLUDE [preview](./includes/preview.md)]

## Prerequisites

* If you don't have an Azure subscription, create a [free account](https://azure.microsoft.com/free/) before you begin.
* When you're deploying the Unreal DDC managed application, you need to have **Contributor** and **RBACRoleAssignment**, or **Owner** access to the Azure subscription. This requires specific permission within the Azure Active Directory tenant. For more information, see [Manage access](../intro-to-azure/concept-manage-access.md) and [Azure built-in roles](/azure/role-based-access-control/built-in-roles).

## Create a new Game Development Virtual Machine

Start by creating a new Game Development Virtual Machine using one of the following guides.

* [Azure Portal](/gaming/azure/game-dev-virtual-machine/create-game-development-vm-for-unreal)
* [Azure ARM Template](/gaming/azure/game-dev-virtual-machine/create-game-development-vm-arm-template)
* [Azure Bicep](/gaming/azure/game-dev-virtual-machine/create-game-development-vm-bicep-template)

Then, connect to the Game Development Virtual Machine using RDP.

## Next steps

In this guide you have created a Game Development Virtual Machine.
Now, learn how to start using Unreal Cloud DDC with Visual Studio from this machine. All of the prerequisites have been setup on the VM.

* [Use with Visual Studio](integrate-with-vs.md)