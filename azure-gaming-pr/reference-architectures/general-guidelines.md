---
title: General Guidelines - Azure
description: Set of best practices and rules of thumb from services leveraged in the Azure gaming reference architectures
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# General Guidelines and Best Practices

## Naming Conventions

The choice of a name for any service in Azure is important because:
- It is difficult to change a name later.
- Names must meet the requirements of their specific resource type.

Consistent naming conventions make services easier to locate. They can also indicate the role of a service in a solution.

See the [naming conventions](https://docs.microsoft.com/azure/architecture/best-practices/naming-conventions#naming-subscriptions) article summarizing the naming rules and restrictions for Azure services and a baseline set of recommendations for naming conventions.

## Resource groups

In Azure, all resources are allocated in a resource management group. Resource groups provide **logical groupings of resources** that make them easier to work with as a collection so they can be managed by lifetime, owner, or other criteria.

See the page [manage resource groups](https://docs.microsoft.com/azure/azure-resource-manager/resource-group-portal#manage-resource-groups) for all the details and information.

## Azure Storage Regions

If the input and output of different storage resources are located in different regions, **you will be billed to move data between regions**.  Please see the [bandwidth pricing details](https://azure.microsoft.com/pricing/details/bandwidth/) for more information. Pair the Azure services in the same region to avoid the outbound data transfer.

> [!TIP]
> Try to use a region that will be closest to the majority of your users.

## Security

Please see this [collection of security best practices](https://azure.microsoft.com/resources/security-best-practices-for-azure-solutions/) when you’re designing, deploying, and managing your cloud solutions with Azure. These best practices come from our experience with Azure and the experiences of developers like you.

Sections of the white paper are published as individual articles as follows:

- [Optimize identity and access management](https://docs.microsoft.com/azure/security/azure-security-identity-management-best-practices)
- [Use strong network controls](https://docs.microsoft.com/azure/security/azure-security-network-security-best-practices)
- [Lock down and secure VM and computer operating systems](https://docs.microsoft.com/azure/security/azure-security-iaas)
- [Protect data](https://docs.microsoft.com/azure/security/azure-security-data-encryption-best-practices)
- [Secure databases](https://docs.microsoft.com/azure/security/azure-database-security-best-practices)
- [Define and deploy strong operational security practices](https://docs.microsoft.com/azure/security/azure-operational-security-best-practices)
- [Design, build, and manage secure cloud applications](https://docs.microsoft.com/azure/security/security-paas-deployments)

See [Azure security best practices and patterns](https://docs.microsoft.com/azure/security/security-best-practices-and-patterns) for additional security best practices.

Also check out the [Azure DDoS Protection: Best practices and reference architectures](https://aka.ms/ddosbest) documentation.

## Privacy

To reduce the **privacy liability footprint**, leverage features offered by some of the services that store your user's data:

- You can expire old data automatically stored in Azure Cosmos DB using [Azure Cosmos DB TTL](https://docs.microsoft.com/azure/cosmos-db/time-to-live) (Time To Live), setting a time horizon where stored documents will be purged.
- You can delete blobs at the end of their lifecycle using [Azure Blob Storage lifecycle management policy](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts).

## Azure Resource Manager

Leverage Azure Resource Manager to create, update, and delete resources in your Azure subscriptions. See [Troubleshoot common Azure deployment errors with Azure Resource Manager](https://aka.ms/rps-not-found) and [Resolve errors for resource provider registration](https://docs.microsoft.com/azure/azure-resource-manager/resource-manager-register-provider-errors) if you are running into issues.

## Azure Event Hub Partitions

The **number of partitions can't be changed after creation**. As you are planning to target production and potentially receive events from a large number of users, you should perform some pre-calculations, backed by load testing and data consumption, to find out the exact number of partitions you will need.

Please see this [video](https://azure.microsoft.com/resources/videos/build-2015-designing-and-sizing-a-global-scale-telemetry-platform-on-azure-event-hubs/) for a complete video guide about designing and sizing a global scale telemetry platform on Azure Event Hubs.

1 [Throughput Unit](https://docs.microsoft.com/azure/event-hubs/event-hubs-features#throughput-units) (TU going forward) equates 1 MB/sec or 1000 messages/sec, whichever happens first.  You will be charged for TUs and to adjust your cost you can change TUs as per your load requirements.

Each individual partition is maxed out by 1 MB/sec or 1000 messages/sec ingress, whichever happens first. Consider the following principles to decide the number of partitions:

- Partitions offer high-availability.  If you are sending a message to Event Hubs and want the sends to succeed, you should create multiple partitions and send using `EventHubClient.Send`.
- The number of partitions will determine how wide the Event Hub pipe is and how fast you can receive and process the messages.  If you have 16 partitions on your Azure Event Hub, it's capacity is maxed out to 16 TUs.  The analogy is **partitions are equal to lanes on a highway**.
- TU is configured at the namespace level.  And one Event Hubs namespace can have multiple Azure Event Hubs. Each Azure Event Hub can have different number of partitions.

You **can't take advantage of more TUs than you have partitions**, if you have 3 partitions, you can't use more than 3 TUs. It's typical to have more partitions than TUs, for the following reasons:

- You can scale the number of TUs up if your traffic increases, but you can't change the number of partitions.
- You can't have more concurrent readers than you have partitions.  If you want to have 10 concurrent readers, you need 10 partitions.

> [!TIP]
> The recommended path would be that you model your partition count starting with 1 TU while you are developing your solution.  Once that's done, when you are doing load testing in your private beta, increase TUs to tune to your load.  As a reminder, you could have multiple Azure Event Hubs in the same Event Hub Namespace.  So, having 20 TUs at the Azure Event Hub Namespace level and 10 Event Hubs with 4 partitions each can deliver 20 MBPS across the entire Azure Event Hub Namespace.

## Azure Cosmos DB

Azure Cosmos DB resources are billed based on **provisioned throughput and storage**. Azure Cosmos DB throughput is expressed in terms of  [Request Units](https://docs.microsoft.com/azure/cosmos-db/request-units) per second (RU/s going forward).

To provide predictable performance, you should reserve throughput in units of 100 RU/second. You can estimate your throughput needs by using the Azure Cosmos DB [request unit calculator](https://www.documentdb.com/capacityplanner).

> [!TIP]
> As a thumb rule/best practice, the maximum RUs number should not exceed 20 times the minimum/steady-state throughput.
> - You can increase to 2x your initial provisioned throughput instantaneously. You can asynchronously increase to any throughput value up to 1M asynchronously (higher via support ticket).
> - You can decrease to 10x your initial provisioned throughput instantaneously (the total range is 20x). But it’s possible to decrease beyond this range in some cases.

For more information see [provision throughput on Azure Cosmos containers and databases](https://docs.microsoft.com/azure/cosmos-db/set-throughput).

## Azure Storage Account Limits

Azure Storage has certain limits you which are documented at the [Azure subscription service limits](https://docs.microsoft.com/azure/azure-subscription-service-limits#storage-limits) page.

If the needs of your game exceed the scalability targets of a single storage account, you can build your application to **use multiple storage accounts**. You can then partition your data objects across those storage accounts.

## Azure Storage Explorer

[Azure Storage Explorer](https://azure.microsoft.com/features/storage-explorer/) is a tool that you can use to managed your Azure Storage resources very easily. Upload, download, and manage blobs, files, queues, tables, and Azure Cosmos DB entities. Gain easy access to manage your virtual machine disks. Supports Windows, MacOS and Linux.

## Azure Stream Analytics Units

Choosing the number of required [Streaming Units](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-streaming-unit-consumption) (SUs going forward) for a particular job depends on the partition configuration for the inputs and the query that's defined within the job. It is a best practice to **allocate more SUs than needed**. The Azure Stream Analytics processing engine optimizes for latency and throughput at the cost of allocating additional memory.

In general, the best practice is to start with 6 SUs for queries that don't use **PARTITION BY**, then determine the sweet spot by using a trial and error method in which you modify the number of SUs after you pass representative amounts of data and examine the SU% Utilization metric. The maximum number of streaming units that can be used by a Stream Analytics job depends on the number of steps in the query defined for the job and the number of partitions in each step. You can learn more about the limits [here](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-parallelization#calculate-the-maximum-streaming-units-of-a-job).

For more information about choosing the right number of SUs, see this page: [Scale Azure Stream Analytics jobs to increase throughput](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-scale-jobs)

> [!NOTE]
> Choosing how many SUs are required for a particular job depends on the partition configuration for the inputs and on the query defined for the job. You can select up to your quota in SUs for a job. By default, each Azure subscription has a quota of up to 200 SUs for all the analytics jobs in a specific region. To increase SUs for your subscriptions beyond this quota, you will have to contact [Microsoft Support](https://support.microsoft.com/). Valid values for SUs per job are 1, 3, 6, and up in increments of 6.

Also, Azure Stream Analytics uses variable size batches to process events and write to outputs. Typically the Azure Stream Analytics engine does not write one message at a time, and uses batches for efficiency. When both the incoming and the outgoing events rate is high, it uses larger batches. When the egress rate is low, it uses smaller batches to keep latency low.  See the [output batch size](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-define-outputs#output-batch-size) page that explains some of the Azure Stream Analytics considerations to output batching.

## Azure Database for MySQL

Retry logic is essential when you are using Azure database for MySQL. It is important that Azure Database for MySQL database games are **built to detect and retry dropped connections and failed transactions**. When the application retries, the application's connection is transparently redirected to the newly created instance, which takes over for the failed instance. Internally in Azure, a gateway is used to redirect the connections to the new instance. Upon an interruption, the entire fail-over process typically takes tens of seconds. Since the redirect is handled internally by the gateway, the external connection string remains the same for the client applications. See [managing connections](https://docs.microsoft.com/azure/mysql/concepts-database-application-development#managing-connections) to learn how to make sensible use of connections when accessing the database.

Regarding **scaling up or down**, when an Azure Database for MySQL is scaled up or down, a new server instance with the specified size is created. The existing data storage is detached from the original instance, and attached to the new instance. During the scale operation, an interruption to the database connections occurs. The client applications are disconnected, and open uncommitted transactions are canceled. Once the client application retries the connection, or makes a new connection, the gateway directs the connection to the newly sized instance. See [scale resources](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers#scale-resources) for more details.

The following [link](https://docs.microsoft.com/azure/mysql/concepts-limits) describes capacity, storage engine support, privilege support, data manipulation statement support, and functional limits in the database service.

It’s worth knowing that the server is marked read-only when the amount of free storage reaches less than 5 GB or 5% of provisioned storage, it’s recommended that you **set up an alert to notify you when your server storage is approaching the threshold** so you can avoid getting into the read-only state. For more information, see the documentation on [how to set up an alert](https://docs.microsoft.com/azure/mysql/howto-alert-on-metric).

## Azure Container Instances

In general, it's recommended to **use Linux** over Windows with Azure Container Instances, as Windows containers are often heavier deployment images that take longer to spin up and load. Also Linux has an additional set of features with Azure Container Instances not currently supported on Windows, like virtual networks.