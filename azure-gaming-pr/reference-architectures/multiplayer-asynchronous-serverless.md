---
title: Serverless Asynchronous Multiplayer - Azure
description: This is a reference architecture to build a serverless asynchronous multiplayer game on Azure
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Serverless Asynchronous Multiplayer Reference Architecture

## Architecture diagram

[![Asynchronous multiplayer architecture diagram](media/multiplayer/multiplayer-async-single.png)](media/multiplayer/multiplayer-async-single.png)

## Architecture services

- [Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-overview): Chosen as they are the simplest way to run the small pieces of matchmaking logic, also you are only charged when you have users attempting to matchmake and play. Note that when using a Consumption plan, Function definitions are stored in File Storage, meaning that you will have to create an Storage account. For optimal performance you should use a Storage account in the same region as the Functions.
- [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/): Chosen as the information to store is unlikely going to need schema changes, it won't be requiring persistence. Also the service is fast, lightweight, reliable and cost effective.
- [Notification Hub](https://docs.microsoft.com/azure/mysql/): choosen as it's an easy-to-use and scaled-out push engine that allows you to send notifications to any platform (iOS, Android, Windows, Kindle, Baidu, etc.).
- [SignalR](https://azure.microsoft.com/services/signalr-service/): choosen as it simplifies the process of adding real-time web functionality to applications over HTTP, allowing to push to connected device clients.
- [Key Vault](https://docs.microsoft.com/azure/key-vault/key-vault-overview): choosen as it's the best service for secret management, including database connection strings.

## Design considerations

This specific reference architecture showcases a **serverless simple tic-tac-toe** game.

To wrap this up, in this reference architecture a helper class (Data Client) will be used to take point on connecting and interacting with the database, and the rest of the Functions will make use of it when needed. Also another class (Game Session) is used for calculating the winning logic and play the turn with the information provided by the player. There will be 3 different action events supported: *forfeit* (to give up on a game), *addPlayer* (to join a player to a game session) and *takeTurn*

## Deployment template

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes an article summarizing the naming rules and restrictions for Azure services.

>[!NOTE]
> If you're interested in how the ARM template works, review the Azure Resource Manager template documentation from each of the different services leveraged in this reference architecture:
>
> - [Create an Event Hub using Azure Resource Manager template](https://docs.microsoft.com/azure/event-hubs/event-hubs-resource-manager-namespace-event-hub)
> - [Automate resource deployment for your function app in Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code)
> - [Azure Database for MySQL template](https://docs.microsoft.com/azure/templates/microsoft.dbformysql/servers)
> - [Azure Notification Hub template](https://docs.microsoft.com/azure/templates/microsoft.notificationhubs/allversions)

>[!TIP]
> To run the Azure Functions locally, update the *local.settings.json* file with these same app settings.

## Step by step

### Create a new game session

1. The device client formats any game settings selected by the player and sends the *start game session* event to the backend, then awaits for a response.
2. The backend receives the command to start a new game session. First of all it tries to find an existing game session that matches the player settings.
3. Assuming a suitable game session is **not available** for matchmaking, a new game session object is created.
4. Also and new a durable Orchestrator Function is created.
5. The durable Orchestrator Function reads the game session object and **awaits until at least 2 players have joined the game session**.
6. Then another player with the same settings selected by the other player sends a *start game session* event to the backend.
7. The backend receives the command and tries to find an existing game, in this case it **finds the game session previously created**.
8. The durable Orchestrator Function receives the *addPlayer* event and stops waiting as the 2 players have joined the game session.
9. Then the durable Orchestrator Function officially starts the match, setting the game state to **in progress** and selecting randomly a player as the starter. In a nutshell, the Orchestrator is responsible for executing the game logic and updating the game state.
10. As a next step the durable Orchestrator Function triggers an operation to **persisting the data into the database**.
11. The writing operations into the persistent database are batched via **Azure Event Hub** to avoid exhausting the database connections.
12. The Azure Event Hub is bound to an **Azure Function** that leverages the data client helper class to connect to the database to persist the data.
13. The durable Orchestrator Function continuous then checking against the logic run by the game session instance, in this case returns that it's the turn of a certain player.
14. It proceeds then **queuing the notifications** to the player or players based on the logic conditions (it's someones turn like in this case, someone won, someone forfeited, etc).
15. The durable Orchestrator then **invokes itself** with the new game state, ready for the next event to be received.
16. The device client receives the notification, including the Orchestrator Function unique identifier managing the game session.
17. The device client sends the *load game session* event to the backend including the durable Orchestrator unique identifier received in the notification.
18. The backend receives the command to **load the game session** and returns the details of the game session for being displayed in the device client.
19. The player submits an X or a O directly to the durable Orchestrator Function and ends the turn.

### Request player games list

1. The device client submits the *get list of sessions* command to the backend.
2. The backend handles the request by querying the persistent database, then sends the device client back the response including the data. Consider limiting the number of results returned by the query and ordering them.
3. The device client receives the backend response and leverages the data it contains to populate the pertinent UI.

## Implementation sample

### Exercise for the reader

The sample provided doesn't include logic to scale the *reads* from the database, only the *writes*. Consider fronting the database with a cache or scale up the database to avoid exhausting the connections to the database.

## Security considerations

If you store the database connection string plain in the code, anyone who has access to source code can search through and get access to the database, because the information required to connect is stored within it. At the minimum it should be stored as part of the Function [application settings](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings#settings), for bonus points use Key Vault instead as it's the recommended method. There is a tutorial explaining how to [create a Key Vault](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/create-an-azure-key-vault-and-secret/), how to [use a managed service identity with a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/using-managed-service-identity-msi-with-and-azure-app-service-or-an-azure-function/) and finally how to [read the secret stored in Key Vault from a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/how-to-connect-to-a-database-from-an-azure-function-using-azure-key-vault/).

## Alternatives

In this reference architecture, Azure Database for MySQL was the database that it was opted for. However, it could be replaced for [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview) or [Azure Database for PostgreSQL](https://docs.microsoft.com/azure/postgresql/).

## Additional resources and samples

### Trivia game using durable Functions and Azure SignalR service

[![Trivia game using Azure SignalR Service and Durable Functions](media/multiplayer/multiplayer-async-trivia.png)](media/multiplayer/multiplayer-async-trivia.png)

Even though not entirely asynchronous per se as it runs on a 20 seconds timer, you can find the implementation of a **trivia game** built on durable Functions and Azure SignalR Service [on this link](https://github.com/anthonychu/serverless-trivia). Clues are retrieved from [jservice.io](https://jservice.io/). To see it running, access [this link](https://aka.ms/serverlesstriviatrivia).

The step by step is:

1. The trivia owner starts the game, invoking the **HttpStartSingle HTTP triggered Azure Function**, either upon deployment or via a web request.
2. That Azure Function kicks off the **TriviaOrchestrator Durable Azure Function**.
3. Players decide to join the trivia. The device client from each player running the web app reaches out to the backend to get the Azure SignalR Service hub details. The backend receives the request via the **SignalRInfo Azure Function**, a token is generated by the binding using the key in the connection string, then it is sent to the device client. The device client sets the two signals from the Azure SignalR Service that is going to listen to: *newClue* and *newGuess*.
4. In the backend the **TriviaOrchestrator Durable Azure Function** calls the **GetAndSendClue Durable Activity Azure Function**.
5. The **TriviaOrchestrator Durable Azure Function** sets a timer to call itself every 20 seconds, that's the time players have to respond to the trivia clue.
6. The **GetAndSendClue Durable Activity Azure Function** pulls a trivia clue from the jservice.io service.
7. Then the trivia clue is **broadcasted via Azure SignalR Service** to all connected users with the target *newClue*.
8. Players receives the trivia clue from the **Azure SignalR Service**, then they think what could it be the answer to the trivia question and submit it to the backend.
9. The backend receives the requests via the **SubmitGuess Azure Function**.
10. This Azure Function calculate if the answers submitted by the players are correct or not. Then the pertinent outcomes are **broadcasted via Azure SignalR Service** to all connected users with the target *newGuess*.
11. The device client receives the outcome from **Azure SignalR Service** and updates the number of correct or incorrect guesses.

In case you want to keep the Trivia running all the time, to restart an eternal orchestration, you can make use of a TimerTrigger Azure Function to run on startup (RunOnStartup=true) and periodically check if the eternal orchestration is running, and if not, start a new one.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures, the total amount depends on the number of events that will run though the analytics pipeline. See the pricing webpages for each of the services that were used in the reference architectures:

- [Azure Function](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Database for MySQL](https://azure.microsoft.com/pricing/details/mysql/)
- [Azure Notification Hubs](https://azure.microsoft.com/pricing/details/notification-hubs/)
- [Azure SignalR Service](https://azure.microsoft.com/pricing/details/signalr-service/)
- [Azure Key Vault](https://azure.microsoft.com/pricing/details/key-vault/)

You also have available the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/), to configure and estimate the costs for the Azure services that you are planning to use.