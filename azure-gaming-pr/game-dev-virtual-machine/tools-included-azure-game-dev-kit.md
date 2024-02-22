---
title: Tools included with the Azure Game Development Virtual Machine
description: The Azure Game Development Virtual Machines are pre-configured with the complete operating system, security patches, drivers, and core tools and frameworks used in the life cycle of game development.
author: meaghanlewis
ms.topic: overview
ms.date: 03/16/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Tools included with the Azure Game Development Virtual Machine

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

Microsoft has partnered with many of the top game development tooling companies to combine the common tools needed to build a game development workstation in the cloud, which we call the Azure Game Development Virtual Machine. With this game development VM, developers can quickly spin up a GPU Workstation or HPC build server in Azure to start developing and building the next game title in the cloud. Built on top of Visual Studio Community Edition 2019 Windows image in Azure, the Game Development Virtual Machines are pre-configured with the complete operating system, security patches, drivers, and core tools and frameworks that are being used in the life cycle of game development. You can also choose the virtual desktop remoting technology such as Parsec, Teradici or RDP, which are all configured and [optimized](/azure/virtual-desktop/configure-vm-gpu) for GPU-intensive workloads; and configure striped disk volume(s) for faster IOPS during the creation of this VM.

The following list of software is installed on the Game Development Virtual Machine.

> [!NOTE]
> This is a growing list. As the Virtual Machine evolves, so will the software installed. To recommend common software that should be included, please provide feedback using the mechanism on this page.

## Operating systems

* Windows Server 2019
* Windows 10

## Remote access technology

* [Parsec Agent](https://parsec.app/)
* [Teradici PCoIP Agent](https://www.teradici.com/)  
* RDP [GPU accelerated encoding](/azure/virtual-desktop/configure-vm-gpu#configure-gpu-accelerated-frame-encoding)

## Game engines

* [Unreal Engine 4.27](https://www.unrealengine.com/)
* [Unreal Engine 5](https://www.unrealengine.com/unreal-engine-5)

## IDEs

* Visual Studio 2019 Community Edition
* [Visual Studio Code](https://code.visualstudio.com/)

## Version control tools

* [Perforce's Helix Visual Client (P4V)](https://www.perforce.com/downloads/helix-visual-client-p4v)
* [Git / Git LFS](https://git-scm.com/downloads)

## Game SDKs

* [Microsoft Game Development Kit (GDK)](https://github.com/microsoft/GDK)
* [PlayFab SDK](/gaming/playfab/sdks/sdk-overview) and [UE Marketplace Plugin](https://www.unrealengine.com/marketplace/en-US/product/playfab-sdk)
* [Windows 10 SDK (Includes DirectX)](https://developer.microsoft.com/windows/downloads/windows-10-sdk/)

## Other tools installed

* Azure CLI
* [Blender](https://www.blender.org/)
* Chocolatey
* DirectX Runtime
* [Epic Games Launcher](https://www.epicgames.com/store/download)
* [Incredibuild Visual Studio Plugin](https://marketplace.visualstudio.com/items?itemName=vs-publisher-1193210.IncrediBuild), Agent and Coordinator (Bring Your Own License)
* NodeJS
* [NVIDIA GPU GRID drivers](/azure/virtual-machines/windows/n-series-driver-setup)
* [PIX](https://devblogs.microsoft.com/pix/introduction/)
* Python
* [Quixel Bridge](https://quixel.com/bridge)
* [Simplygon 10](https://www.simplygon.com/)
* [UnrealVS Plugin](https://docs.unrealengine.com/4.27/en-US/ProductionPipelines/DevelopmentSetup/VisualStudioSetup/UnrealVS/)
