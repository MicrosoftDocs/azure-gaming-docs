---
title: Multiplayer Game Server Hosting Using Azure Service Fabric
description: This is a reference architecture to build a scalable game server hosting on Azure Service Fabric
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Synchronous Multiplayer Using Azure Service Fabric

The game server pools are managed by Azure Service Fabric, responsible for **creating and orchestrating Azure Virtual Machine Scale Sets**. Each region will have it's own pool of game servers.

## Architecture diagram

[![Synchronous multiplayer using Azure Service Fabric](media/multiplayer/multiplayer-shortsession-sfhosting.png)](media/multiplayer/multiplayer-shortsession-sfhosting.png)

## Relevant services

- [Azure Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-overview): selected as it connects the player to the most appropiate regional zone based on latency.
- [Azure Service Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-overview): makes it easy to package, deploy, and manage scalable and reliable game servers within containers.

Leverage one resource group for the Azure Traffic Manager and one resource group for each regional Service Fabric Cluster.

## Deployment template

Refer to [this repository](https://github.com/Azure-Samples/service-fabric-cluster-templates) containing sample Service Fabric cluster templates that you can customize for use in setting up your clusters. The intent is to provide a wide variety of templates, based on the input and the types of clusters we have seen other developers create. The templates cover both Windows and Linux clusters.

The most basic template to start with is [this one](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/5-VM-Windows-1-NodeTypes-Secure-NSG). It allows to deploy a secure 5 node, Single Node Type Service Fabric Cluster running Windows Server 2016 Datacenter with containers on a Standard_D2_v2 Size Virtual Machine Scale set with Azure Diagnostics turned on and network security groups enabled.

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes an article summarizing the naming rules and restrictions for Azure services.

The game server is stored in Azure Storage or alternative Azure Container Registry. Service Fabric will fetch the game server from either of them.

## Step by step

1. The player's device client connects to the **Azure Traffic Manager** to route a request for the player to find a game server.
2. The Azure Traffic Manager connects to the regional zone with the lowest latency and points to the matchmaker to get a game server available.
3. The matchmaker has all the information required to select a game server, if more capacity is required it proactively pings the **Azure Service Fabric** service to start scaling out in a particular Service Fabric cluster.
4. The Azure Service Fabric service receives the request and begins scaling out. If automated scaling was set up, it may have proactively kicked off the process depending on the rules established.
5. The game servers regularly sends the matchmaker an status update once a game session is over and they are *available*, also their most updated IP and port.
6. Each of the player devices connect directly to the game server using the connection information provided by the matchmaker.

Optionally, after the game session is over, relevant information could be stored in an Azure Storage account.

## Scaling

There are two main approaches:

1. The matchmaker doesn't scale and Azure Service Fabric owns scaling leveraging [Azure Service Fabric auto scaling](https://docs.microsoft.com/azure/service-fabric/service-fabric-cluster-resource-manager-autoscaling), where the service dynamically scale your game servers based on the load that they are reporting, or based on their usage of resources. Auto scaling gives great elasticity and enables provisioning of additional instances or partitions of your game server on demand. The entire auto scaling process is automated and transparent, and once you set up your policies, there is no need for manual scaling operations at the game server level. Auto scaling can be turned on either at the creation time, or at any time via updating.

    A common scenario where auto-scaling is useful is when the load varies over time, like in multiplayer games.

2. Alternatively like in this example, you can task the matchmaker to proactively let Azure Service Fabric know when to scale out. The best practice is to use the Azure Service Fabric pool manager pattern.

See [Scaling in Server Fabric](https://docs.microsoft.com/azure/service-fabric/service-fabric-concepts-scalability) to read about how you can build scalable microservices games.

## Additional resources and samples

To manage Azure Service Fabric games and clusters successfully, there are operations that are highly recommend to perform, to optimize for the reliability of the production environment; please perform operations defined in [this document](https://docs.microsoft.com/azure/service-fabric/service-fabric-best-practices-overview), that cover security, networking, infrastructure as a core and monitoring amongst other things.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures, the total amount depends on the number of events that will run though the analytics pipeline. See the pricing webpages for each of the services that were used in the reference architectures:

- [Azure Service Fabric](https://azure.microsoft.com/pricing/details/service-fabric/)
- [Azure Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/)

You also have available the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/), to configure and estimate the costs for the Azure services that you are planning to use.