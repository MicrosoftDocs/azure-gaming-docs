---
title: What's new on the Game Development Virtual Machine 
description: Release notes for the Azure Game Development Virtual Machine
author: meaghanlewis
ms.topic: reference
ms.date: 08/09/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Azure Game Development Virtual Machine release notes

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

In this article, learn about updates for Azure Game Development Virtual Machine. For a full list of tools included, check out the page: [Tools included with the Azure Game Development Virtual Machine](./tools-included-azure-game-dev-kit.md). And see the list of [known issues](./known-issues.md) to learn about known bugs and workarounds.

Due to the rapidly evolving needs and packages updates, we target to release new Azure Game Development Virtual Machine images every month. Azure portal users will always find the latest image available for provisioning the Game Development Virtual Machine. For CLI or ARM users, we keep images of individual versions available for twelve months. After that period, a particular version of image is no longer available for provisioning.

## February 2023
- Offered Windows 11 and Server 2022 images with Visual Studio 2022 and Unreal Engine 5.1 built-in.
- Supported generation 2 VMs on Windows 11 based SKUs .
- Supported Azure Spot Virtual Machines.
- Mulitple performance improvements and bug fixes.   

## September 2022

- Integrated [PIX](https://devblogs.microsoft.com/pix/introduction/).
- Integrated [Simplygon](https://www.simplygon.com/).
- Published Game Dev VM to Azure Government Marketplace.  

## August 2022

- Announcing General Availability of the [Azure Game Dev VM](https://azure.microsoft.com/products/virtual-machines/game-development-virtual-machines/), with [Azure Game Production Pipeline: Create Games on the Cloud](https://developer.microsoft.com/games/blog/azure-game-production-pipeline-create-games-on-the-cloud/)
- Game Dev VM is now supported under publisher: **microsoft-azure-gaming**, offer ID: **game-dev-vm**.
- Added option for not pre-installing Unreal Engine.
- Dozens of minor bug fixes and feature improvements.

## July 2022

- Added support for new Azure VM size: [NVadsA10 v5-series](/azure/virtual-machines/nva10v5-series).
- Support ARM deployment.
- Support integration with Azure Virtual Desktop (AVD).
- Game Engine is now saved on a dedicated Azure data disk.

## May 2022

- Announcing the future supportability of [Microsoft Dev Box](https://techcommunity.microsoft.com/t5/azure-developer-community-blog/introducing-microsoft-dev-box/ba-p/3412063) at [Microsoft Build 2022](https://www.youtube.com/watch?v=GonrcDal3QU&t=275s).
- Added support for custom image creation using sysprep.
- Added option to use Game Dev VM as a build server which enabled non-GPU VM sizes.

## April 2022

- Integrated the official release of Unreal Engine 5.
- Published more customer facing docs: [Game Development Virtual Machine documentation](/gaming/azure/game-dev-virtual-machine/known-issues).
- Improvement of stability and minor bug fixes.

## March 2022

Announcing the public preview of the Azure Game Dev VM at the Game Developers Conference (GDC).

See the blog post and video on [Game Creation Cloud Adoption and The Azure Game Dev VM](https://developer.microsoft.com/games/blog/game-creation-cloud-adoption-and-the-azure-game-dev-vm/) for more details.
