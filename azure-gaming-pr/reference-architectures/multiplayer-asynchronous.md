---
title: Asynchronous Multiplayer - Azure
description: This is a reference architecture to build an asynchronous multiplayer game on Azure
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Asynchronous Multiplayer Reference Architecture

The backbone for building an asynchronous multiplayer is saving the game state to a **persistent database** so that it can be retrieved by all players in the game session when it’s time to take the next turn, and a **push notification** mechanism.

There are several variables which can be taken into consideration when defining an asynchronous multiplayer:

- **Support multiple games in parallel**: yes, no.
- **Deadline**: the player has a specific amount of time (6 hours, 12 hours, 24 hours, etc) to complete a turn or the game is forfeited.
- **Game custom settings**: consider allowing players that create new game sessions with specific settings from your game.

## Reference implementation details

This specific reference architecture showcases a **serverless simple tic-tac-toe** game. Here are different implementations of the same use case to get you a head start:

- [Serverless](./multiplayer-asynchronous-serverless.md): using Azure Functions.

The following sections are covering general design considerations common to the different implementations documented.

## Player authentication and identity

This reference architecture doesn't cover neither player authentication nor an in-depth player identity management, both are left as an exercise for the reader.

PlayFab offers multiple [forms of authentication](https://api.playfab.com/docs/tutorials#landing-players) so players can be tracked across multiple devices:

- Device ID for guest login
- Username/password
- Google account
- GameCenter account
- Facebook account
- Steam account
- Kongregate account
- Twitch account
- Other oAuth providers
- Android device ID
- Custom player ID

## Expansion to multiple data centers

This reference architecture only contemplates a single region for the backend. Depending on the game being built, leveraging a traffic manager and deploying to multiple data centers may not be needed. Since Traffic Manager is used for HTTP calls, it might actually get cheaper if all the resources are in one data center and not spread throughout the world. Moreover, having multiple data centers also implies that a database synchronization mechanism should be used to synchronize the database data between different data centers. Having said that, if you want to have multiple data centers, then you should use [Traffic Manager](https://azure.microsoft.com/services/traffic-manager/).

## Step by step

The general flow of an asynchronous multiplayer title is as follows:

1. The player logins to the backend.
2. Existing player's games list is retrieved from the backend.
3. The player creates a new open game with a set of settings, or invites a specific opponent.
    - If a player creates a new open game, it gets added to the game session list and waits for an opponent to join. When a player creates a new open game, they’re actually sending a request to connect to an opponent. If there are other suitable players waiting that match the settings, the game will matchmake both players directly.
    - If a player invites another specific player, it's like creating a new open game but with a specific opponent. In this reference architecture, this feature is not contemplated.
4. The player chooses an action supported in the game and it's submitted to complete the turn.
5. Once the turn is submitted, the persistent database record for this game session is updated with the updated game state.
6. A notification is sent to the opponent player, so they can begin their turn.

## Database schema

The Azure Database for MySQL persistent database that will be used to store the information required has three tables: **player** table that will contain one entry per player and should be expanded to a robust player identity system, **gamesession** that will contain one entry per game session, and the **gamesession_player** table that will contain one entry per user per game.

### Player table

|Field|Type|Notes|
|----------|----------|-----------|
|**ID**|SERIAL PRIMARY KEY|It's the primary key of this table|
|**CreatedTime**|TIMESTAMP NOT NULL|Stores when the player was added|
|**UpdatedTime**|TIMESTAMP NOT NULL|Stores when the player was updated|
|**Name**|VARCHAR(100)|Stores the name of the player|

### Game session table

|Field|Type|Notes|
|----------|----------|-----------|
|**ID**|SERIAL PRIMARY KEY|Primary key of this table|
|**GUID**|VARCHAR(36)|Unique identifier of the game session|
|**CreatedTime**|TIMESTAMP|Stores when the game session was created|
|**UpdatedTime**|TIMESTAMP|Stores when the game session was updated|
|**CreatedPlayer_ID**|BIGINT UNSIGNED|Which player created the game session|
|**GameStatus**|TINYINT NOT NULL|Matchmaking (0), in progress (1) or completed (2)|
|**BoardState**|VARCHAR(200)|Serialized board information (where the X and O are placed in the board)|
|**MovesLeft**|TINYINT|To determine when the game is meant to be over|
|**CurrentTurnPlayer_ID**|BIGINT UNSIGNED|Which player has to make a move|
|**WinningPlayer_ID**|BIGINT UNSIGNED|The player that won|

### Game session player table

|Field|Type|Notes|
|----------|----------|-----------|
|**ID**|SERIAL PRIMARY KEY|It's the primary key of this table|
|**CreatedTime**|TIMESTAMP NOT NULL|Stores when the player was added|
|**UpdatedTime**|TIMESTAMP NOT NULL|Stores when the player was updated|
|**GameSession_ID**|BIGINT UNSIGNED NOT NULL|Foreign key. This is the unique game identifier from the game session table|
|**Player_ID**|BIGINT UNSIGNED NOT NULL|Foreign key. This is the unique player identifier from the player table|

## Create database tables

See this [tutorial](
https://docs.microsoft.com/azure/mysql/quickstart-create-mysql-server-database-using-azure-portal#get-the-connection-information) to read about how to get the database connection information, configure the firewall and how to establish a connection to the database, either:
- using the [mysql command-line tool](https://dev.mysql.com/doc/refman/5.7/en/mysql.html) from the Azure portal Cloud Shell, note that it requires a cloud drive account.
- using the [MySQL Workbench GUI tool](https://dev.mysql.com/downloads/workbench/).

After that, run this command to create a database:

```sql
CREATE DATABASE asyncmpdb;
```

And this one afterwards to start using it:

```sql
USE asyncmpdb;
```

Then this commands to create the 3 tables:

```sql
CREATE TABLE player (
    ID SERIAL PRIMARY KEY,
    CreatedTime TIMESTAMP NOT NULL,
    UpdatedTime TIMESTAMP NOT NULL,
    Name VARCHAR(100) NOT NULL
);

CREATE TABLE gamesession (
    ID SERIAL PRIMARY KEY,
    GUID VARCHAR(36) NOT NULL,
    CreatedTime TIMESTAMP NOT NULL,
    UpdatedTime TIMESTAMP NOT NULL,
    CreatedPlayer_ID BIGINT UNSIGNED NOT NULL,
    GameStatus TINYINT NOT NULL,
    BoardState VARCHAR(200) NOT NULL,
    MovesLeft TINYINT NOT NULL,
    CurrentTurnPlayer_ID BIGINT UNSIGNED,
    WinningPlayer_ID BIGINT UNSIGNED,
    FOREIGN KEY(CreatedPlayer_ID) REFERENCES player(ID),
    FOREIGN KEY(CurrentTurnPlayer_ID) REFERENCES player(ID),
    FOREIGN KEY(WinningPlayer_ID) REFERENCES player(ID)
);

CREATE TABLE gamesession_player (
    ID SERIAL PRIMARY KEY,
    CreatedTime TIMESTAMP NOT NULL,
    UpdatedTime TIMESTAMP NOT NULL,    
    GameSession_ID BIGINT UNSIGNED NOT NULL,
    Player_ID BIGINT UNSIGNED NOT NULL,
    FOREIGN KEY(GameSession_ID) REFERENCES gamesession(ID),
    FOREIGN KEY(Player_ID) REFERENCES player(ID)
);
```

Check that the 3 tables are created using this command:

```sql
show tables;
```

Once the tables are created, let's create a bunch of stored procedures. The first one is for matchmaking, returning a suitable game session (if available), after a new game session is attempted to be created:

```sql
DELIMITER //
CREATE PROCEDURE gamesession_SELECT
(IN param_playerid BIGINT, IN param_limit INTEGER)
BEGIN
    SELECT GUID FROM gamesession
    WHERE GameStatus = 0
    AND CreatedPlayer_ID <> param_playerid
    ORDER BY CreatedTime
    LIMIT param_limit;
END //
DELIMITER ;
```

This one is for creating a new game session, it is executed when the matchmaking attempt hasn't returned any suitable game session:

```sql
DELIMITER //
CREATE PROCEDURE `gamesession_INSERT`
(IN param_guid VARCHAR(36), IN param_createdplayer_id BIGINT UNSIGNED, IN param_status TINYINT, IN param_boardstate VARCHAR(200), IN param_movesleft TINYINT, IN param_currentturnplayer_id BIGINT UNSIGNED, IN param_winningplayer_id BIGINT UNSIGNED)
BEGIN
    INSERT INTO gamesession (GUID, CreatedTime, UpdatedTime, CreatedPlayer_ID, GameStatus, BoardState, MovesLeft)
    VALUES (param_guid, NOW(), NOW(), param_createdplayer_id, param_status, param_boardstate, param_movesleft);
  	INSERT INTO gamesession_player (CreatedTime, UpdatedTime, GameSession_ID, Player_ID)
   	VALUES (NOW(), NOW(), LAST_INSERT_ID(), param_createdplayer_id);
END //
```

## Notification services

There are two main services available to submit notifications: Notification Hubs and SignalR. The table below showcases the main differences between those 2 services. The implementation would follow a cascade pattern, where first a SignalR notification is attempted, and if after a small period of time a *received* notification is not received, then the SignalR delivery is cancelled and Azure Notification Hub is used as a fall back.

||**SignalR**|**Notification Hubs**|
|----------|----------|-----------|
|**Message source**|Can be from either game server (broadcast or server push only), or another client (chat like bi-directional), depends on the scenario|Existing infrastructure of the platform providers (Microsoft, Google, Apple, etc)|
|**Dedicated server required**|Depends on the scenario. the service supports REST API, so message source/app can publish message to client via SignalR Service REST API directly without a server needed. Alternatively it officially supports Azure Functions bindings with SignalR Service, so it accommodates serverless scenario as well|No|
|**Code agnostic**|Yes. SignalR now has client SDKs in C#, JS, also covers Xamarin, Unity and Java SDK. Upcoming releases for C++ and Object-C/Swift. The REST API supports any REST capable language. The Azure Functions bindings supports any language Azure Function supports|Yes, you can use a [REST API and templates](https://docs.microsoft.com/azure/notification-hubs/notification-hubs-aspnet-cross-platform-notification)|
|**Message delivery**|Immediate (via WebSocket)|Immediacy not guaranteed|