---
title: Leaderboard Reference Architecture
description: This reference architecture describes a leaderboard use case and implementation with different alternatives, enabling you to architect your own cloud solution so you can have full control and customization to fit your game design like a glove.
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.prod: azure-gaming
---

# Leaderboard Reference Architecture

These reference architectures describe a leaderboard use case and implementation with different alternatives, enabling you to architect your own cloud solution so you can have full control and customization to fit your game design perfectly.

> [!TIP]
> If you are looking for an out-of-the-box leaderboard solution, **Azure PlayFab** is a complete back-end platform for building, launching, and growing cloud connected games that has [leaderboard support](https://docs.microsoft.com/gaming/playfab/features/social/tournaments-leaderboards/).

## Leaderboard design

There are many variables which can be taken into consideration when defining a leaderboard. Here are some examples for you to consider:

- **Geography**: Physical location where the player is located. Be as granular as you want, from countries/regions to continents and beyond. Examples: France, China, USA, North America, Asia, EMEA.
- **Platform**: Device or service the game is played on. You could also consider enabling leaderboards that combine more than one platform. Examples: Xbox, PlayStation, PC, iOS, Android, Mobile, Consoles.
- **Mode**: Distinct configuration that changes gameplay and affects how other game mechanics behave. Examples: solo, duo, squad, PvE, PvP, campaign.
- **Difficulty**: User-selected challenge. Examples: easy, medium, hard.
- **Option**: Individual options the player has chosen. Examples: character (human, vehicle), item (weapon, tool), setting (manual or automatic shifting).
- **Level**: In-game level or stage. Examples: Level 2-1, Track 3.
- **Statistic**: Individual value that the player generates based on usage or actions. Examples: score, wins, losses, coins collected.

### Player authentication and identity

This reference architecture doesn't cover player authentication nor player identity.  For more information on authentication, check out Azure PlayFab, which offers many [forms of authentication](https://docs.microsoft.com/rest/api/playfab/client/authentication) so players can be tracked across multiple devices.

### Size and complexity

Leaderboards can vary from a very simple system of ranking players based on a score, or it can grow in size and complexity by tracking a combination of multiple statistics, options, levels, platforms, and more. For example, imagine a leaderboard combining the variables **Platform**, **Geography**, **Mode**, **level** and **Statistic**.  This would allow ranking of players in a game scenario using values like "score (Stat) of players using an *Xbox One* (Platform) in *France* (Geography) for the mode *Player vs Player* (mode) on level *3* (level)". With this data being tracked, you could then remove the Geography filter and rank all players regardless of where they live.

In another example, imagine combining the variables **System**, **Level** and **Statistic**. This would allow the game to rank players in a racing game scenario like "time to complete the track *Nürburgring* (level) by players competing on *PC* (system).

Of course, specific games might have even more ways to rank players against each other. The documented sample implementations are a starting point using just a few of the variables above. In this use case, the **player's score will be ranked based on two variables: platform and level**. Players from each platform will compete for the highest score on each level. Feel free to build from here and modify to suit your game's needs.

### Data refreshing

A leaderboard with stale data is almost as bad as not having a leaderboard at all. However, for cost reasons, it may make sense to avoid refreshing data frequently. Try to space out the refreshes and recalculate the rankings in a range from every minute up to every hour. In some cases it may even make sense even to refresh once per day.  However, if you really need to have the leaderboard data refreshed in (near) real-time, this reference architecture will accommodate.

### Players around me

It is a best practice to show where the user is located on the leaderboard, so the player can see who is above and below them, and show that they can climb higher.  This can contribute to a more engaged player base by demonstrating they can easily climb higher in the rankings, while not demoralizing the player by showing them only the top-ranked players in the game.

### Anti-cheating

It's inevitable that some players will try to inject fake scores into your leaderboards, and that will have a negative impact on the community. There are a few things that you can do to try to make things difficult for cheaters:

- Encrypt the communications between the client and the cloud to prevent packet sniffing.
- Add telemetry and then use it on the server side to see if a score makes sense, ideally automatically, so cheated scores can be detected on the fly. Two straight forward checks would be:
    1. Could the player realistically finish a level in the time stated?
    2. How often is the player submitting a score?
- Do not discard scores that are incorrect. Instead, mark both the score and the player that submitted the score as suspect (carefully, as you may accidentally impact a legitimate user.)  After that, when known legitimate users are accessing the leaderboard, retrieve only the scores not marked as potentially cheating. When a cheater submits a leaderboard view request, retrieve only the incorrect scores. The idea is to avoid giving the cheater direct feedback that the bogus score injection didn't work, so they believe they succeeded.

## Reference implementation details

When it comes to choosing the database that will store leaderboard information, one of the first decisions to make is whether to use a relational or non-relational (NoSQL) data structure. There are certain differences to bear in mind:

||Relational|Non-relational|
|----------|----------|-----------|
| **Preparative** | Structure/schema of the data must be determined beforehand, changes later can be disruptive | Minimal structure required
| **Rigidness** | All data must use the same structure | Each data entry can have its own structure and new fields can be easily added later on
| **Primary storage model** | Table based | Document store, graph database, key-value store or wide column store
| **Scalability** | Vertically scalable - increase server CPU, RAM or storage | Horizontally scalable - sharding or add more servers

For all the details on the different storage models, please see the [choose the right data store](https://docs.microsoft.com/azure/architecture/guide/technology-choices/data-store-overview) documentation.

On a final note, personal expertise on either alternative is something to take into consideration. Choosing a known path will spare you from having to deal with an entirely new set of unknown problems and the time it will take to become proficient on new best practices and concepts.

Here are two different implementations of a simple leaderboard use cases to get you started:

- [Non-relational (NoSQL)](./leaderboard-non-relational.md): Using Azure Cache for Redis for small scale or Azure Cosmos DB for large scale.
- [Relational](./leaderboard-relational.md): Using Azure SQL Database or MySQL for small scale or Azure SQL Database for large scale.

## Additional resources and samples

[Azure Data Architecture Guide](https://docs.microsoft.com/azure/architecture/data-guide/)