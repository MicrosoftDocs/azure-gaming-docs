---
title: What is the Azure Game Development Virtual Machine? 
description: Describes the Azure Game Development Virtual Machine, sample use cases, and how to use the Azure Game Development Virtual Machine as a game build server.
author: meaghanlewis
ms.topic: overview
ms.date: 2/1/2022
ms.author: mosagie
ms.prod: azure-gaming
---

# What is the Azure Game Development Virtual Machine?

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

The Game Development Virtual Machine is a customized Azure Virtual Machine built for game developers to save them time in having to spend hours to install and setup common tools for game remote development in Azure. Microsoft has assembled the top game development tooling in one single VM, allowing you to focus less on setup and more on building great games. Supported by GPU intensive Azure VMs, these common game development tools include a game engine, frameworks, remoting protocols, drivers and SDKs, allowing you to get started developing your games in Azure within minutes. It can also be used for [scaled-out build servers](/gaming/azure/reference-architectures/incredibuild-scale-out-your-cloud-builds/) which also need similar tooling to build your game. In addition to Remote Desktop Protocol, this game dev VM includes options for Teradici and Parsec technology, which both provide a high fidelity and low latency remote access experience.

The Game Development Virtual Machine is a great choice for developers who are building interactive games that use engines like Unreal and code with Visual Studio, leveraging Perforce and Git source control solutions. No matter if you’re working for your next AAA title or an Indie game, spin up a VM to enjoy the game development journey in the cloud. All tooling is pre-installed with Bring Your Own License (BYOL) options where applicable, and you only pay for the base Azure compute costs used—no additional costs are added for leveraging this VM.

The Game Development Virtual Machine is currently supported for the following operating systems:

* Windows Server 2019
* Windows 10

## Introductory video

Learn how to make remote game development easy with the Azure Game Development Virtual Machine in this video:
> [!VIDEO https://www.youtube.com/embed/5tx99WgzBcc ]

## What's included on the Azure Game Development Virtual Machine?

See a [full list of tools](./tools-included-azure-game-dev-kit.md) on the Game Development Virtual Machine.

## Why Develop Games in the Cloud?

With many teams working remotely either due to hybrid working scenarios or being geographically distributed, there are great benefits to moving either part or your entire game development pipeline to the cloud. Some of the key benefits include the following:

* Spin up powerful compute, GPUs, SSD disks and serverless file shares in minutes, allowing for immediate onboarding of new resources without waiting for hardware game developers need
* Work remotely from anywhere and use the cloud as your desktop that has persistent storage, only paying for compute when you need it
* Leveraging the speed and global scale of the cloud to take advantage of dark fiber networking across the globe
* Burstable compute capabilities for faster builds, especially when using technologies like [Incredibuild](https://www.incredibuild.com/partners/azure) for accelerated compiling and asset cooking across hundreds of distributed cores
* Better collaboration experiences when sharing your desktop or creative work across distributed teams, especially when using Parsec’s screen sharing or even Unreal Pixel Streaming.
* Allow quick turnaround times for game testers to get compiled builds faster for testing

Read this blog post [Game Creation Cloud Adoption and The Azure Game Dev VM](https://developer.microsoft.com/games/blog/game-creation-cloud-adoption-and-the-azure-game-dev-vm/) to know more.

## Use Cases for the Azure Game Development Virtual Machine

Below we illustrate some common use cases for customers.

### Developing games in Azure

For distributed game development teams who want to take advantage of the [benefits](./overview.md#why-develop-games-in-the-cloud) of developing and building games in the cloud, the Game Development Virtual Machine can be a key linchpin of your game development pipeline in Azure. It allows for a quick spin up for virtual desktops, build servers and integrating into key external solutions such as source control (e.g., Perforce or Git), distributed builds solutions like Incredibuild, and tighter DevOps orchestration with Azure DevOps, Jenkins, TeamCity and more. As a game engine and other core tools are pre-installed, this allows developers to skip the hours in setup/install and start building immediately, or even using the pre-configured VM as a base image to add your studio’s custom tools on top to your own final image.

Game engines are a ubiquitous technology driving efficiency in the games industry, enabling developers to create games quicker. Currently Unreal Engine comes pre-installed with multiple recent versions of Unreal Engine 4 and Unreal Engine 5 (preview), with more options to grow as they are released. The Game Development Virtual Machine also includes Visual Studio Community Edition 2019, along with various utilities including Microsoft’s Game Development Kit, PlayFab SDK, DirectX and more to start building your game directly in Azure. If you’re in an AAA game studio working with multiple developers, the pre-configured Perforce or Git client in this game dev VM can help connect each developer to the code repository for version control integration.

:::image type="content" source="./media/overview/game-dev-vm-diagram.png" alt-text="Screenshot of diagram showing how to improve sharing and collaboration experience for worldwide remote developers":::

### Using as build servers

Since build servers require many of the same tooling used to develop the game such as a compiler and your game engine, the Game Development Virtual Machine is a great accelerator to deploy as a build server as well. This can be setup as an individual build machine, using virtual machine scale sets, or even containerizing the image and integrating with Azure Kubernetes Service. This VM has Incredibuild tooling pre-installed (requires you to bring your license), which allows it to be a Custom Build Agent for spinning up multiple build servers to dramatically reduce the build time by using parallel computing. The Game Development Virtual Machine can be automated to pull down the latest source code and then orchestrate accelerated builds across hundreds or more of distributed cores.

:::image type="content" source="./media/overview/use-game-dev-vm-as-build-server.png" alt-text="Screenshot of diagram showing how a game dev VM can be used as a build server":::

For the step-by-step instructions, please refer to this article: [Incredibuild - Scale out your cloud builds](/gaming/azure/reference-architectures/incredibuild-scale-out-your-cloud-builds/)

### Using as test servers

Similar as using this Game Development Virtual Machine for build servers, you will find it helpful in your game testing by integrating with Azure DevTest Labs.

:::image type="content" source="./media/overview/use-game-dev-vm-as-test-server.png" alt-text="Screenshot of diagram showing how a game dev VM can be used in DevTest Labs":::

### Experimentation and evaluation with game development in Azure

As game development requires high-end computer hardware and a combination of different tools which can take hours to install and setup, using the Game Development Virtual Machine allows you to spin up Azure GPU optimized virtual machines with the common game development tools you need to get started in minutes. This could be for evaluation of how game development tooling performs in Azure, or use the deployed VM as a base image to build your own development image from. Be sure to check out our published How-to guides to learn how to get started with using and integrating with the Game Development Virtual Machine.

## Next steps

* [Create a Game Developer Virtual Machine with Unreal Engine](./create-game-development-vm-for-unreal.md)
* [Using GitHub with Git Tools on the Azure Game Development Virtual Machine](./github-git-tools-on-vm.md)
