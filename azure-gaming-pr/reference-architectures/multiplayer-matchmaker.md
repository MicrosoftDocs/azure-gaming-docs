---
title: Multiplayer Matchmaker - Azure
description: This is a reference architecture to build a multiplayer matchmaker on Azure
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Multiplayer Matchmaker Reference Architecture

## Design considerations

For implementing a matchmaker it will be needed a database storing the data from the players that are looking for a matchmaking session and a process running on a server or a set of serverless Azure Functions responsible for handling the logic:

1. **Add player to the queue**: invoked when the player wants to start searching for a multiplayer session.
2. **Delete player from the queue**: this is invoked when the player has decided to stop searching for a multiplayer session (cancel matchmaking).
3. **Give player a match**: invoked via polling mechanism, it returns either a game session server connection details or a timeout event.
4. **Add servers**: when a new server is created, it passes the relevant connection information (IP:Port) for being added to the database.
5. **High level manager**: looks for game sessions that can be started and finds a server for each of them.

Optionally you can have another process or Azure Function to **request a scale out** when it detects that there are not enough servers. The alternative is to delegate this to the game hosting orchestrator should you are using one.

When you are building your matchmaking logic, there are three key variables to take into consideration. Aim to make it work for two out of three at least:

- **Best skill match**: from all the players considered for a game session, how close each of them are in terms of expertise with the game?
- **Best latency**: how close each of the players are in terms of latency?
- **Best queueing time**: how long does it take to find a game session for a player to join?

Additionally there are some related concepts to think about:

- **Whitelisting/blacklisting**: can players, or you as the game creator, block certain players or allow specific players to join a game session? 
- **Join in progress**: are players able to join a game session after it has started?
- **Auto-cancellations**: will you set a limit where a specific game session is auto-cancelled if it fails to start after some time?

To wrap up, a good approach to reduce random matchmaking time is to have a queue for each type that your game supports and put each player attempting to matchmake in *all* queues matching their request. Then when a queue is full, just start that specific game session and remove the involved players from all of their queues.

> [!TIP]
> If you are looking for an out-of-the-box matchmaking solution, **PlayFab** is a complete back-end platform for building, launching, and growing cloud connected games that has [matchmaking support](https://docs.microsoft.com/gaming/playfab/features/multiplayer/matchmaking/).

## Reference implementation details

Here are different implementations of the same use case to get you a head start:

- [Serverless](./multiplayer-matchmaker-serverless.md): using Azure Functions and Azure Cache for Redis.