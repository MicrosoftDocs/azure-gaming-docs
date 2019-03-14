---
title: Custom Game Server Scaling
description: See how to encapsulate your game server with Docker and build a reliable and automated deployment process of the game server using Azure Resource Manager templates, Azure Functions and DevOps practices.
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.service: azure
---

# Custom Game Server Scaling Reference Architecture

Learn how to containerize your game server with Docker, and build a reliable, automated deployment process for servers using Azure Resource Manager templates, Azure Functions and DevOps practices.

Refer to [Orchestrating and scaling Icebird's game server with Docker and Azure](https://microsoft.github.io/techcasestudies/devops/azure%20app%20service/azure%20functions/2017/04/21/IceBird.html) to read all the details. There is both **source code** and a **deployment template** available on [GitHub](https://github.com/Annonator/FuncyAutoScale).

## Architecture diagram

[![Basic game server hosting using encapsulation](media/multiplayer/multiplayer-custom-game-server-scaling.png)](media/multiplayer/multiplayer-custom-game-server-scaling.png)

## Reference implementation details

Each virtual machine includes a Docker container that runs a game session. As soon as the virtual machine starts, it instantiates the Docker container and opens the required network port(s) through a custom script extension ([Linux](https://docs.microsoft.com/azure/virtual-machines/extensions/custom-script-linux), [Windows](https://docs.microsoft.com/azure/virtual-machines/extensions/custom-script-windows)). Every container has it's own public dedicated IP address.

Additionally, there is a **get server Azure Function**, that runs on an App Service plan, which allows for additional scaling options as described in the [App Service environment](https://docs.microsoft.com/azure/app-service/environment/intro) document. Any App Service must have an Azure Storage account and, the Azure Function service will provision it.  In this storage account, an [Azure Table Storage](https://docs.microsoft.com/azure/storage/tables/table-storage-overview) table is used to store the information about the pool of servers, including the unique identifier of the server, its IP address, port, and status. The *get server Azure Function* uses this information to return connection details to the client, as well as to mark a server as not available when it is being used.

To help scale the server pool, a timer-triggered **auto-scale Azure Function** is used. Every minute or so, it looks how many servers are available and adds additional servers if they are required. If there are too many servers in the pool that are unused, it will deprovision them. You can set up how many servers you want to have warm in the pool.

Once a game server starts, it needs to talk to a third *send details Azure Function* to announce its existence, so the pertinent connection information can be updated in Azure Table Storage.

After a game session is finished, the game server pings a fourth **game session over Azure Function** that updates the state of the Azure Table Storage for that specific server.

Ultimately the goal is to free up the virtual machines as fast as possible, so this architecture is focusing on having a single game session per virtual machine only.

Keep an eye on the [Azure limits](https://aka.ms/azurelimits) page to know how many concurrent users you would be able to run based on the Azure Storage limitations. If you need to scale up, consider replacing Azure Table storage with Azure Cosmos DB and its table API.

## Deployment template

Click the following button to deploy the project to your Azure subscription:

<a href="https://aka.ms/arm-gaming-custom-server-scaling" target="_blank"><img src="media/azure-resource-manager-deploy-button.png"/></a>

This operation will trigger a template deployment of the [template.json](https://github.com/Annonator/FuncyAutoScale/blob/master/Deployment/template.json) ARM template file to your Azure subscription, which will create the necessary Azure resources. This may induce charges in your Azure account.

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes an article summarizing the naming rules and restrictions for Azure services.

>[!NOTE]
> If you're interested in how the ARM template works, review the Azure Resource Manager template documentation from each of the different services leveraged in this reference architecture:
>
> - [Automate resource deployment for your function app in Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code)
> - [Azure Container Registry template](https://docs.microsoft.com/azure/templates/microsoft.containerregistry/registries)

>[!TIP]
> To run the Azure Functions locally, update the *local.settings.json* file with these same app settings.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures.  The total amount will vary based on usage. See the pricing webpages for each of the services that were used in the reference architecture:

- [Azure Windows Virtual Machines](https://azure.microsoft.com/pricing/details/virtual-machines/windows/)
- [Azure Linux Virtual Machines](https://azure.microsoft.com/pricing/details/virtual-machines/linux/)
- [Azure Disk Storage](https://azure.microsoft.com/pricing/details/managed-disks/)
- [Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Container Registry](https://azure.microsoft.com/pricing/details/container-registry/)
- [Azure Table Storage](https://azure.microsoft.com/pricing/details/storage/tables/)

You can also use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) to configure and estimate the costs for the Azure services that you are planning to use.