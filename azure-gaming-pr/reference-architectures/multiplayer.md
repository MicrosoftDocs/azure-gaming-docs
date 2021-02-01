---
title: Multiplayer Backend Reference Architectures
description: These reference architectures describe a variety of multiplayer backend use cases and implementations with different alternatives, enabling you to architect a cloud solution that works for your game.
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.prod: azure-gaming
---

# Multiplayer Backend Reference Architectures

These reference architectures describe a variety of multiplayer backend use cases and implementations with different alternatives, enabling you to architect a cloud solution that works for your game.

## Use Cases

There are many variables which can be taken into consideration when designing a multiplayer backend for your game. Here are some examples:

1. **Level of management** - From putting all the effort on yourself to letting the platform take care of everything
1. **Operating system** - Windows or Linux
1. **Where does the game session run** - Dedicated server or peer-to-peer (P2P)
1. **Hardware** - What does the game session need to be able to run
1. **Processing time** - Real-time or non-real time (NRT)
1. **Latency** - Players will be at a disadvantage if they have lag or they won't
1. **Persistence** - The game world continues to exist and develop internally even when there are no players interacting with it, or each session has it's own beginning and end
1. **Number of concurrent players** - Small, medium or large
1. **Reconnection allowed** - If a player or players get disconnected, can they go back to the game or they have to start a new game session

Here are are some multiplayer backend use cases for you to explore:

- [Realtime multiplayer](./multiplayer-synchronous.md)
- [Turn-Based multiplayer](./multiplayer-asynchronous.md)

> [!TIP]
> If you are looking for an out-of-the-box scaling multiplayer server solution, **Azure PlayFab** is a complete back-end platform for building, launching, and growing cloud connected games that has [multiplayer servers support](https://docs.microsoft.com/gaming/playfab/features/multiplayer/servers/).

## Multiplayer Design

### Level of management

Compute services vary based on the level of management they offer, from those managed entirely by you, to those managed entirely by the platform:

- **Raw Virtual Machines** - Everything is managed by you, it needs a custom scaling solution
- **Azure Container Instances (ACI)** - Everything is managed by you but in a container, it needs a custom scaling solution
- **Virtual Machine Scale Sets** / **Batch** - Manages the scaling of Virtual Machines on your behalf based on rules you define
- **Service Fabric** / **Azure Kubernetes Service (AKS)** - Manages the orchestration of containers on your behalf
- **Azure PlayFab Multiplayer Servers** - Higher level orchestration of game servers on your behalf, running on top of Azure. For more information see [Azure PlayFab Multiplayer Servers](https://docs.microsoft.com/gaming/playfab/features/multiplayer/servers/).

### Operating System

The table below provides an overview of what operating systems are supported by the different Azure compute services.

|Service|Only Windows|Only Linux|
|----------|-----------|------------|
|Raw Virtual Machines|[Yes](https://docs.microsoft.com/azure/virtual-machines/windows/overview)|[Yes](https://docs.microsoft.com/azure/virtual-machines/linux/overview)|
|Azure Container Instance (ACI)|Yes* (Windows 10 1607 only)|Yes|
|Virtual Machine Scale Sets|[Yes](https://docs.microsoft.com/azure/virtual-machine-scale-sets/quick-create-template-windows)|[Yes](https://docs.microsoft.com/azure/virtual-machine-scale-sets/quick-create-template-linux)|
|Batch|Yes|[Yes](https://docs.microsoft.com/azure/batch/batch-linux-nodes)|
|Service Fabric|[Yes](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started)|[Yes](https://docs.microsoft.com/azure/service-fabric/service-fabric-get-started-linux)|
|Azure Kubernetes Service (AKS)|[Only via AKS-Engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/windows.md)|[Yes](https://docs.microsoft.com/azure/aks/intro-kubernetes)|
|Functions|Yes|Yes (dedicated mode)|

\* Note: Multi-container groups are currently restricted to Linux containers.

### Where Does the Game Session Run?

#### Dedicated Server

The players connect their client devices to the server in order to play the game.

Some things to keep in mind when you are considering using dedicated servers:

- They provide connectivity and reliability benefits.
- They are fairly straightforward to implement and scale well.
- It's more difficult for a player to be able to cheat.
- Other player's internet could affect if the game is frame synchronized.
- Depending on the type of game, players who live very close to the dedicate servers may have a measurable advantage over those who live further away.

#### Peer to Peer (P2P)

Some things to keep in mind when you are considering a P2P model:

- Operating cost is cheaper than leveraging dedicated servers.
- It is more difficult to pull off a truly successful implementation in contrast to a dedicated server solution.
- In many places, home internet service does not have high enough upload speeds to cope with more than a small number of players.
- A player with a bad connection can influence the game for the other players.
- Clients sometimes cannot connect due to NAT punch-through issues. Port forwarding may be required.
- Avoid handing each player's IP address directly to every other player, otherwise players are open to DDoS attacks from other players in the game.
- Without a central authority in form of a neutral server, there is no easy way to prevent cheating.

### Lobby

A **lobby system** is fairly common for getting players together before actually starting the game session, as players need to start from the same initial state. Enabling late join support is technically possible but it's uncommon due to the considerable complexity it adds. A lobby system requires a server and typically works as follows:

1. Player starts a lobby and optionally sets game parameters.
1. Other players can view/search for the lobby via any querying mechanism the game offers.
1. One of those players joins the lobby.
1. When there are enough players (based on the rules established by the game and, if offered, the lobby creator) the lobby securely distributes the details of each player waiting in the lobby, and its work is done.
1. Players then start sending packets in a decentralized way.

### Monitoring and Alerts

In addition to your preferred telemetry solution, you could leverage [Azure Monitor](https://docs.microsoft.com/azure/azure-monitor/overview) to expose metrics from the Azure services used in your backend solution, along with diagnostic and activity logs. Azure Monitor can also help you identify issues affecting your game. Consider **enabling alerts** for:

- CPU usage - You want to be notified if your hosts' CPUs are nearing saturation.
- Disk I/O - Set up an alert if your game is either reading from disk or writing to disk more often than expected.
- Memory utilization - When memory paging is excessive you should receive an alert.
- Network traffic - You should consider generating an alert when your network traffic is close to saturation or suddenly plummets.

You can automate the configuration of Azure Monitor using [Azure Resource Manager](https://docs.microsoft.com/azure/azure-monitor/platform/template-workspace-configuration). For more information about Azure Monitor see [Full stack end to end monitoring with Azure Monitor](https://channel9.msdn.com/Shows/Azure-Friday/Full-stack-end-to-end-monitoring-with-Azure-Monitor).

### Hardware

Get a good understanding of the processing power, storage, memory, and network traffic that you are going to need to run a game session, as it's directly related to what virtual machine type you are going to require for your multiplayer backend.

There are some data points that are relevant in the decision making process and go beyond how much CPU is required to run your game session:

- How many threads does the game require?
- How much RAM does one game session require?
- How much bandwidth does a user playing the game require?
- How many users will be part of one game session?
- Does the game need hyper-threading? If your game server relies on hyper-threading, then you need to use hyper-threaded cores.
- What is the memory to core ratio?

By collecting this information, you can then figure out:

- How many cores will you need for one session?

  If game server runs in one thread, one core is enough. If the game server runs in 1.5 threads, you need two cores.

  If you want to host 5 game sessions in a server, and each game session requires a thread, then you need at least 5 cores. If you want to host 5 game sessions in a server, and each game session required 1.5 threads, then you need at least 8 cores.

  Take the **operating system consumption** into consideration. The rule of thumb is to over-provision the virtual machines by 1 or 2 cores.

- How much egress will be produced?

  This is a relevant number as it has economical impact. Also, if you have too many players in one virtual machine, you may face **networking restrictions**, especially when you are attempting multiple thousands of users in one NIC, and bottlenecks may surface.

Putting all of this together, you can then determine what virtual machines you are going to want and how much you will need to pay for them. Different virtual machine types have different bandwidth throughput. You will need to ensure the network interface is capable of handling the demand. Calculate the *number of players* per virtual machine multiplied by *the bandwidth consumed per player*. That amount can't be larger than the *max NICs / expected network bandwidth (Mbps)* documented for each of the VM sizes:

- [Sizes for Linux virtual machines in Azure](https://docs.microsoft.com/azure/virtual-machines/linux/sizes)
- [Sizes for Windows virtual machines in Azure](https://docs.microsoft.com/azure/virtual-machines/windows/sizes)

It is worth bringing up that you may want to make use of [premium storage](https://docs.microsoft.com/azure/virtual-machines/windows/premium-storage) to increase the availability of a single instance virtual machine. With premium storage a virtual machine has an SLA of 99.9%.

### Latency Impact

In certain games, split second reflexes are required, and when you sum the player's reaction time plus latency, the game experience could end up being negatively affected.

To reduce or mitigate latency, there are several things to consider. From the implementation stand-point, it is fairly standard practice to enable **prediction on the client side**, to "get in front" of what the player will do at the expense of creating infrequent disagreements between what the player sees on their screen and what actually happens in game. To mitigate such disagreements then some compensation for the lag could be applied, using mechanisms like extrapolation or interpolation for determining where to display the game objects.

From an infrastructure stand-point, the longer the distance from the player to the game server, the greater the latency eventually will be. **Connecting the players to the game servers that are closest to their vicinity** will be have an impact. Azure has more [global regions](https://azure.microsoft.com/global-infrastructure/regions/) than any other cloud provider, offering the scale to bring you closer to your players around the world. [Accelerated networking](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli) is a possibility for reducing latency on the server side, but keep in mind it's only enabled in virtual machines with at least 4 vCPUs. Also if you are going to be using Linux virtual machines, you could consider [DPDK](https://docs.microsoft.com/azure/virtual-network/setup-dpdk) to optimize latency and throughput when you have a large number of players in the same virtual machine.

For games that support groups, there is the need to tackle the case where different members of the group are far from each other. In your game, add the ability of manually choosing the region to connect to, or add a lowest latency common denominator algorithm.

In scenarios where mitigation efforts are unsuccessful, the network latency can reach unmanageable levels. When that happens, more drastic measures need to be applied, such as disconnecting the player suffering the largest latency.

>[!TIP]
> As a best practice, enable telemetry for measuring player latency and combine it together with a feedback mechanism for your QA team or beta testers to let you know when they are experiencing lag (i.e: a key combination, gamepad button combination or icon on screen). With both it will be easier to find out what was leading to the player's perception of lag.

### Protocol

For games that require **real-time** communication, the best practice is to transmit via the **user datagram protocol (UDP)**.  This is often more complex to implement than transmission control protocol (TCP), but has far better performance considerations.

For **asynchronous** or turn-based communications, leveraging JSON **over HTTPS** will suffice.

### Number of Concurrent Players Per Game Session

From the sportsmanship of tic-tac-toe with just 2 players, to the last man standing carnages of battle royale games with 100 players, to the thousands of players of some persistent world games, the number of concurrent players in the same game session impacts the architecture and services to leverage. The three ranges that were considered in these use cases are:

- **Small** - 10 or less concurrent players.
- **Medium** - Between 11 and 50 concurrent players.
- **Large** - More than 50 concurrent players.

## Best Practices

### Capacity Planning is Critical

As with any scaling service, spending time working on planning the proper number of instances required is a critical step. If too few instances are used, then the performance will be degraded. If too many are used, then you will incur unnecessary costs.

Test early and give yourself time. Run a **proof of concept** early on and **one or several betas** before releasing your game to get an idea of what times of the day your players are going to be using it most, and how much capacity you are going to support it.

Make sure your software is scalable and is prepared for your players to enjoy the game, from the login and matchmaking to the game server itself. Test at different levels of player concurrency (100, 1000, 10000, 100000, etc) as every time you add a zero to the number of concurrent players, you are likely going to identify new bottlenecks and you are going to have to make tweaks.

### Deployment Methodology

If you are leveraging Virtual Machine Scale Sets, Service Fabric or Batch, make use of a **single cluster**, **multiple Virtual Machine Scale sets** and **scale in small increments between 1-4 nodes at a time**. Bear in mind that leveraging multiple Virtual Machine Scale Sets will speed the scaling up at the expense of increasing the cost.

Once you have captured daily workloads in your early tests, use that information to plan your scale up operations proactively **an hour before the capacity is going to be needed**.

### Avoid Changing Infrastructure

Do your best to use [Azure Logic Apps](https://docs.microsoft.com/azure/scheduler/migrate-from-scheduler-to-logic-apps) and try to avoid changing infrastructure as much as you can. Keeping a warm pool of running instances will allow you to get players into a game instantly, but will also increase your cost.

### Fail-safe

If a virtual machine fails, what happens to your game session? Depending on the type of game and how much complexity you want to add to the implementation, there are several alternatives.

For quick in-and-out games where the session last a few minutes, you could consider doing nothing and return players to the lobby to start again, as the impact on the players may not be too bad. If you are working on a persistent world game, a virtual machine fail could be a more serious thing, and you may want to do something like move a the player to a new server automatically while retaining their state.

Use a monitor service to figure out if a node has failed and enable alerts. You could also consider keeping the virtual machine that failed for forensics on why the environment failed, or simply delete it.

## Additional resources and samples

- [Ethr is a Network Performance Measurement Tool for TCP, UDP & HTTP](https://github.com/Microsoft/Ethr)
- [Azure Data Architecture Guide](https://docs.microsoft.com/azure/architecture/data-guide/)
