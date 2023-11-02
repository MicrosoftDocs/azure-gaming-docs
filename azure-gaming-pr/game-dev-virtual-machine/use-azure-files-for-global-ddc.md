---
title: Use Azure Files for network derived data cache with Unreal Engine 
description: Setup and use DDC with Azure Storage File Share for the Game Development Virtual Machine.
author: meaghanlewis
ms.topic: how-to
ms.date: 03/15/2022
ms.author: mosagie
ms.prod: azure-gaming
---

# Use Azure Files for network derived data cache with Unreal Engine

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

It is essential that you understand the Azure Files performance and redundancy, and Active Directory authentication and authorization features to support the requirement of using Derived Data Cache (DDC) for the Unreal Engine.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.com/free).
- This article assumes that you know what [Azure Files](/azure/storage/files/storage-files-introduction) is and the benefits it provides.

## Plan Azure Files

The first thing to be done is to [plan for an Azure Files for deployment](/azure/storage/files/storage-files-planning). Since the Game Development Virtual Machine is running on the Windows operating system, the SMB file protocol is recommended. For more information about what to take into consideration when planning Azure Files for deployment refer to the following articles:

- [SMB file shares in Azure Files](/azure/storage/files/files-smb-protocol?tabs=azure-portal)
- [SMB Multichannel performance](/azure/storage/files/storage-files-smb-multichannel-performance)
- [Azure Files scalability and performance targets](/azure/storage/files/storage-files-scale-targets)
- [Azure Files identity-based authorization](/azure/storage/files/storage-files-active-directory-overview)

When you’re ready to deploy, you will first need to [create an Azure file share](/azure/storage/files/storage-how-to-create-file-share?tabs=azure-portal). Your Azure File share should look like the following:

:::image type="content" source="./media/use-azure-files-for-global-ddc/azure-file-share.png" alt-text="Screenshot showing an Azure file share":::

## Mount Azure Storage File Share

There are two different methods to mount the Azure Storage File Share.

### Method 1: Mount Azure File automatically during Game Development Virtual Machine creation

1. Begin walking through the steps to create a [Game Development Virtual Machine with Unreal Engine](./create-game-development-vm-for-unreal.md).
1. When you get to the **Data Storage** tab, select to **Mount an existing Azure Storage File Share**.
1. Select the **Storage Account**.
1. Select the **File Share**.
1. Finish creating the VM.

:::image type="content" source="./media/use-azure-files-for-global-ddc/mount-existing-storage-file-share.png" alt-text="Screenshot showing where to mount an existing Azure Storage File Share":::

### Method 2: Mount Azure File manually after Game Development Virtual Machine creation

1. Follow the instructions to mount the [SMB Azure File share manually](/azure/storage/files/storage-how-to-use-files-windows).
1. Verify that the mounted Azure file is accessible with permission to read/write files and folders after successful remote Game Development Virtual Machine access.

:::image type="content" source="./media/use-azure-files-for-global-ddc/verify-mounted-file-accessible.png" alt-text="Screenshot showing a mounted Azure Storage File Share":::

## Setup and use DDC

You can now follow the Epic Games documentation to [setup and use shared DDC](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DerivedDataCache/) using Azure file share.  

Below is the example of completed configuration of shared DDC, the Global Network DDC, in the Epic Unreal Engine Editor. The **S:/test** path is now connected to the Azure File share on your Game Development Virtual Machine.

:::image type="content" source="./media/use-azure-files-for-global-ddc/epic-engine-global-network-ddc-path.png" alt-text="Screenshot showing the path of the connected Azure File share":::

## Next steps

Continue using Unreal Engine with a shared DDC set up on Azure Files. This improves the team collaboration experience with faster processing time.
