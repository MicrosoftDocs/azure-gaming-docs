---
title: Multiplayer Matchmaker
description: This is a reference architecture to build a multiplayer matchmaker on Azure
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.prod: azure-gaming
---

# Multiplayer Matchmaker Reference Architecture

## Design considerations

For implementing a matchmaker you will need a database storing the data from the players that are looking for a matchmaking session and a process running on a server or a set of serverless Azure Functions responsible for handling the logic:

1. **Add player to the queue** - Invoked when the player wants to start searching for a multiplayer session.
2. **Delete player from the queue** - This is invoked when the player has decided to stop searching for a multiplayer session (cancel matchmaking).
3. **Give player a match** - Invoked via polling mechanism, it returns either a game session server connection details or a timeout event.
4. **Add servers** - When a new server is created, it passes the relevant connection information (IP:Port) for being added to the database.
5. **High level manager** - Looks for game sessions that can be started and finds a server for each of them.

Optionally you can have another process or Azure Function to **request a scale out** when it detects that there are not enough servers. The alternative is to delegate this to the game hosting orchestrator should you be using one.

When you are building your matchmaking logic, there are three key variables to take into consideration. Aim to make it work for two out of three at least:

- **Best skill match** - From all the players considered for a game session, how close each of them are in terms of expertise with the game?
- **Best latency** - How close each of the players are in terms of latency?
- **Best queueing time** - How long does it take to find a game session for a player to join?

Additionally there are some related concepts to think about:

- **Allow/Deny** - Can players, or you as the game creator, block certain players or allow specific players to join a game session? 
- **Join in progress** - Are players able to join a game session after it has started?
- **Auto-cancellations** - Will you set a limit where a specific game session is auto-cancelled if it fails to start after some time?

To wrap up, a good approach to reduce random matchmaking time is to have a queue for each type that your game supports and put each player attempting to matchmake in *all* queues matching their request. Then when a queue is full, just start that specific game session and remove the involved players from all of their queues.

> [!TIP]
> If you are looking for an out-of-the-box matchmaking solution, **Azure PlayFab** is a complete back-end platform for building, launching, and growing cloud connected games that has [matchmaking support](https://docs.microsoft.com/gaming/playfab/features/multiplayer/matchmaking/).

## Reference implementation details

Here are different implementations of the same use case to get you a head start:

- [Serverless](./multiplayer-matchmaker-serverless.md) - using Azure Functions and Azure Cache for Redis.
