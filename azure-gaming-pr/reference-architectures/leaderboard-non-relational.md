---
title: Non-relational Leaderboard
description: This is a reference architecture to enable a leaderboard in your game using a non-relational database.
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.prod: azure-gaming
---

# Non-relational Leaderboard Reference Architecture

## Simple leaderboard for small scale

[![Simple leaderboard look and feel](media/leaderboard/leaderboard-simple-redis.png)](media/leaderboard/leaderboard-simple-redis.png)

### Architecture diagram

[![Simple leaderboard for small scale using a non-relational database](media/leaderboard/leaderboard-non-relational-redis.png)](media/leaderboard/leaderboard-non-relational-redis.png)

### Implementation details

In this complete [tutorial](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-web-app-cache-aside-leaderboard) that includes source code you can learn how to implement a leaderboard that uses Azure Cache for Redis in tandem with another database to improve data throughput and reduce database load.

It leverages the [cache-aside pattern](https://docs.microsoft.com/azure/architecture/patterns/cache-aside) using Azure Cache for Redis as a caching layer for the persistent database backend. Whenever you do a write you write the data to both the persistent database and Azure Cache for Redis. Whenever you read, you read the data from the cache first and if you get a *miss* (the data is no longer in the cache, as it expires after a certain time) then you read the data from the persistent database and right after that you write it into the cache. If you get a cache *hit*, you read the data from the cache.

The sample is using **Entity Framework**, a .NET abstraction layer to model and express relationship between data objects. Using it and in a nutshell, from a programming model standpoint you are getting a bunch of objects and setting properties to those objects, then the Entity Framework will map all the operations into SQL operations.

The step by step is:

1. The device client connects to the **Azure Web App** (web service), like it's shown in the architecture diagram above, to upload a new player's record. Alternatively you could substitute the web service for example with Azure Functions, each of them doing a specific task, or a load balancer with virtual machines.
2. The **Azure Web App** receives the action to upload a new record and, following the cache-aside pattern, writes it first into the persistent database, **Azure SQL Database** in this case.
3. After that, continuing with the cache-aside pattern, the same record is saved into the **Azure Cache for Redis** database.
4. Another player wants to see a specific leaderboard, so it connects to the **Azure Web App** (web service).
5. The **Azure Web App** receives the action to retrieve the records from a specific leaderboard. It attempts first to read it from the **Azure Cache for Redis**. If it finds it there (*hit*), then it just returns the information to the device client.
6. If the attempt to read the records from Azure Cache for Redis gets a *miss* (the data is no longer in the cache, as it expires after a certain time), then the **Azure Web App** reads the records from the persistent database, **Azure SQL Database** in this case.
7. The records retrieved from the persistent database are then saved into the **Azure Cache for Redis** and finally sent to the device client.

### Alternatives

Alternatively to [Azure SQL Database](https://docs.microsoft.com/azure/sql-database/sql-database-technical-overview), you could use other database like [Azure Database for MySQL](https://docs.microsoft.com/azure/mysql/overview) for example.

## Advanced leaderboard for large scale

### Architecture diagram

![Non-relational database leaderboard use case](media/leaderboard/leaderboard-change-feed.png)

### Architecture services

- [Azure Functions](https://docs.microsoft.com/azure/azure-functions/)
- [Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/)

In this implementation, an Azure Function is used to write the data, instead of the Azure Cosmos DB SDK. If you are looking for finer control and debugging, you can leverage an app service and the [Change Feed Processor](https://docs.microsoft.com/azure/cosmos-db/change-feed-processor).

### Architecture considerations

There are a variety of design considerations and choices to make when designing a leaderboard for your game.

#### Containers, partitions and partition keys

Before we go any further, it's worth explaining some of the Azure Cosmos DB foundational aspects.

**Partitioning** Cosmos DB is a horizontally scalable database with data stored on individual servers known as [physical partitions](https://docs.microsoft.com/azure/cosmos-db/partitioning-overview#physical-partitions). As data or throughput is increased. Cosmos adds additional partitions that allows the database to "scale out", increasing storage capacity and processing power.

Azure Cosmos DB abstracts away physical partitions with what are known as **logical partitions**. When a new container is created, the user specifies a property that will be used to assign each item to a logical partition where it will be stored.

A container with a partition key of `userId` with 1000 users will have 1000 logical partitions. However logical partitions are virtual concepts. There is no limit on the number of logical partitions in a container. All data in a logical partition shares the same partition key value. So all data where `userId = abc` will all be in the same logical partition.

Data within each logical partition can be uniquely identified by the combination of it's partition key value and id value. All insert, update and delete operations are done using the partition key value and id value. Data can also be read using queries using a [special SQL syntax](https://docs.microsoft.com/azure/cosmos-db/sql-query-getting-started) unique to Azure Cosmos DB and designed to work with JSON data.

There are three key things that you need to take into consideration when defining a partition key:

1. There is a limit of 20GB per logical partition. Data should not grow beyond this amount so careful design will be needed to ensure high enough cardinality for your partitioning strategy.
2. The partition key property cannot be changed after the container has been created. If a partition key is defined as /userId, it cannot be later changed to be /gameId. However, [migrate a container](#partition-key-update) to overcome this.
3. In write-heavy scenarios, your partition key should spread your writes as evenly as possible to avoid hot spots. In read heavy workloads, queries should strive to serve data from one or as few partitions as possible and avoid high volume of cross-partition or fan-out queries. Where workloads are both read and write heavy, [Change Feed](https://docs.microsoft.com/azure/cosmos-db/change-feed-design-patterns) can be used to materialize data with a different partition key to better answer queries.

For all the details on how to choose a partition key for your leaderboard, see [choosing a partition key](https://docs.microsoft.com/azure/cosmos-db/partitioning-overview#choose-partitionkey).

The implementation of this leaderboard reference architecture will leverage a **global leaderboard general collection** - aka the game play container that stores all the entries - and a **partition key for each combination of values**.

In this use case the **player's score will be ranked based on two variables: system and level**, where the partition key used is `system_level`. Here is an example of the schema for our leader board.

```json
{
    "id": "1",
    "level_system": "Xbox_3",
    "name": "Brian",
    "score": 12345
}
```

To use a concrete example, if a game leveraging this reference architecture and implementation was launched on both Xbox, PlayStation and PC platforms and it only had two levels (1 and 2), that would end up generating the following partitions in Azure Cosmos DB:

- Partition 1 (all_level1)
- Partition 2 (all_level2)
- Partition 3 (xbox_level1)
- Partition 4 (ps4_level1)
- Partition 5 (pc_level1)
- Partition 6 (xbox_level2)
- Partition 7 (ps4_level2)
- Partition 8 (pc_level2)

The schema for the game play data sent at the completion of a level is this:

```json
{
    "id": "1",
    "system": "Xbox Live",
    "level": 2,
    "character": "Damian",
    "name": "Brian",  //platform gamertag
    "score": 12345 // stat
}
```

Merely for clarification purposes, to demonstrate what would happen in a more complex scenario with 3 variables from a racing game: **level** (Laguna Seca and Monza), **choice** (fast class and slow class) and **stat** (fastest lap and power ups picked up). The partition key structure used would be `level_choice_stat` generating the following combinations as users upload their records:

- Partition 1 (lagunaseca_fastclass_fastestlap) [000]
- Partition 2 (lagunaseca_fastclass_powerupspicked) [001]
- Partition 3 (lagunaseca_slowclass_fastestlap) [010]
- Partition 4 (lagunaseca_slowclass_powerupspicked) [011]
- Partition 5 (monza_fastclass_fastestlap) [100]
- Partition 6 (monza_fastclass_powerupspicked) [101]
- Partition 7 (monza_slowclass_fastestlap) [110]
- Partition 8 (monza_slowclass_powerupspicked) [111]

In this scenario, if the first player is driving in the track *Monza* (level) using a car belonging to the *fast class* (choice), the information stored in Azure Cosmos DB would look like this:

| Area | Data |
| - | - |
| Global leaderboard collection (leaderboards_all_data) | { userID: "XYZ", stat: "fastestlap", choice: "fastclass", level: "monza", value: "123" }  { userID: "XYZ", stat: "powerupspicked", choice: "fastclass", level: "monza", value: "1" } |
| Partition 5 (monza_fastclass_fastestlap) | { userID: "XYZ", stat: "fastestlap", choice: "fastclass", level: "monza", value: "123" } |
| Partition 6 (monza_fastclass_powerupspicked) | { userID: "XYZ", stat: "powerupspicked", choice: "fastclass", level: "monza", value: "1" } |

#### Cascade writing pattern

Game play data may typically be written into multiple containers, especially if you have related variables (for example: iOS, Android and mobile. Where mobile is a filter that shows records from both iOS and Android users) the best practice is to not do more than a single writing operation and instead let the [Azure Cosmos DB change feed](https://docs.microsoft.com/azure/cosmos-db/change-feed) do the work by trickling effect to avoid inconsistencies that could happen if only one of the writes actually completed.

#### Changing the Partition key

As previously mentioned, it's not possible to change an existing partition key. There are two strategies to deal with this. If you are still designing your data model you can design your schema to name your partition key with a generic name such as "pk". Then if you need to change what partition key value for a row, simply write a new row with the new partition key value and delete the old row.

If you have an existing container with a descriptive name you can migrate the data to a new container. Let's say that you have a partition key of `system_level` and want to add the difficulty users are playing with (easy, medium, hard). To enable this you would migrate the container from the old partition key `system_level` to `System_level_difficulty`. To do this you can use the [Live Data migration tool](https://github.com/Azure-Samples/azure-cosmosdb-live-data-migrator) available on GitHub that follows these steps:

1. Reads the whole container using the change feed.
2. Re-ingests the data using the different partition key.

#### Azure Cosmos DB request units per second

Have a look at the [general guidelines documentation](./general-guidelines.md#azure-cosmos-db) to understand how Azure Cosmos DB collections are billed, how to track the number of request units and the rule of thumb for requesting them.

### Deployment template

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes an article summarizing the naming rules and restrictions for Azure services.

### Optimization considerations

With respect to optimizing Cosmos DB cost, here are some useful resources:

- [Pricing model](https://docs.microsoft.com/azure/cosmos-db/how-pricing-works)
- [Optimizing for development and testing in Azure Cosmos DB](https://docs.microsoft.com/azure/cosmos-db/optimize-dev-test)
- [Total cost of ownership (TCO)](https://docs.microsoft.com/azure/cosmos-db/total-cost-ownership)
- [Understand your bill](https://docs.microsoft.com/azure/cosmos-db/understand-your-bill)
- [Optimize provisioned throughput cost](https://docs.microsoft.com/azure/cosmos-db/optimize-cost-throughput)
- [Optimize query cost](https://docs.microsoft.com/azure/cosmos-db/optimize-cost-queries)
- [Optimize storage cost](https://docs.microsoft.com/azure/cosmos-db/optimize-cost-storage)
- [Optimize reads and writes cost](https://docs.microsoft.com/azure/cosmos-db/optimize-cost-reads-writes)
- [Optimize multi-regions cost](https://docs.microsoft.com/azure/cosmos-db/optimize-cost-regions)
- [Optimize dev/testing workloads](https://docs.microsoft.com/azure/cosmos-db/optimize-dev-test)
- [Optimize with reserved capacity](https://docs.microsoft.com/azure/cosmos-db/cosmos-db-reserved-capacity)

## Additional potential features

### Adding support for push notifications

Depending on the platform your players are using, you may want to let them know when their score has been beaten by a friend for example, you can leverage the [Azure Cosmos DB change feed](https://docs.microsoft.com/azure/cosmos-db/change-feed) and the [Azure Notification Hubs service](https://docs.microsoft.com/azure/notification-hubs/) for enabling that.

![Push notification support architecture](media/leaderboard/leaderboard-push-notification.png)

## Additional resources and samples

### Functions and Cosmos DB via MongoDB API

Set up a [game Leaderboards API hosted on Azure Functions and backed by Azure Cosmos DB with Mongo API](https://github.com/dgkanatsios/AzureFunctionsNodeLeaderboards-Cosmos) that stores game leaderboards (scores) and exposes them via HTTP(s) methods/operations. Use this API service in your game and post new scores, get the top scores, find out the latest ones and more.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures, the total amount depends on the number of events that will run though the analytics pipeline. See the pricing webpages for each of the services that were used in the reference architectures:

- [Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Cosmos DB pricing](https://azure.microsoft.com/pricing/details/cosmos-db/)
- [Azure Virtual Machines pricing](https://azure.microsoft.com/pricing/details/virtual-machines)

You also have available the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/), to configure and estimate the costs for the Azure services that you are planning to use.
