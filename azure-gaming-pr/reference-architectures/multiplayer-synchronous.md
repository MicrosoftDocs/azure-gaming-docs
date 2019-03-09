---
title: Synchronous Multiplayer
description: This reference architecture describes a non-persistent real-time multiplayer scenario where latency matters leveraging dedicated servers, and how to get it implemented.
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Synchronous Multiplayer Reference Architecture

This reference architecture describes a **non-persistent real-time** multiplayer scenario where **latency matters** leveraging **dedicated servers**, and how to get it implemented. It also provides different alternatives. It is assumed that the game does not support player groups and that the game is responsible for finding the most suitable game server to connect to.

## Design considerations

Multiplayer servers for real-time games need to be **stateful** (memory), **can't be explicitly shut down** since players might be still enjoying their game and, as a rule of thumb, their connection with the players must be of **minimal latency**. See these [recommendations](./multiplayer.md#latency-impact) to achieve minimal latency and maximum competition fairness.

There are two main differentiated parts in a synchronous multiplayer backend: **matchmaker** and the **game servers hosting**.

## Reference implementation details

Refer to the [matchmaker](./multiplayer-matchmaker.md) reference architecture page to see all the details about how to implement one.

Here are different implementations of the same game server hosting use case to get you a head start:

- [Azure Batch](./multiplayer-synchronous-batch.md)
- [Azure Kubernetes Service (AKS)](./multiplayer-synchronous-aks.md)
- [Azure Service Fabric](./multiplayer-synchronous-sf.md)
- [Azure Container Instances (ACI)](./multiplayer-synchronous-aci.md)
- [Custom game server scaling](./multiplayer-custom-server-scaling.md)