---
title: In-editor Debugging Telemetry
description: This reference architecture focuses on the development phase and a small number of users, gathering data from gameplay sessions and displaying it directly within the game engine.
keywords: analytics 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.service: azure
---

# In-editor Debugging Telemetry Reference Architecture

This reference architecture **focuses on the development phase and a small number of users**, gathering data from gameplay sessions and displaying it directly within the game engine - Unreal Engine in this case. It provides the fastest response time so your development and QA teams don't have to wait to get results from testing sessions.

[![In-editor debugging telemetry look and feel](media/analytics/analytics-in-editor-telemetry.png)](media/analytics/analytics-in-editor-telemetry.png)

## Architecture diagram

[![In-editor debugging telemetry reference architecture](media/analytics/analytics-in-editor-debugging-telemetry.png)](media/analytics/analytics-in-editor-debugging-telemetry.png)

## Relevant services

- [Azure Event Hub](https://azure.microsoft.com/services/event-hubs/) - Selected as it's a service tailored for analytics pipelines and is simple to use with little configuration or management overhead. It is capable of receiving and processing events in real-time.
- [Azure Functions](https://azure.microsoft.com/services/functions/) - Selected because we would like a simple authentication mechanism, as well as to customize how data processed and queried.  
- [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db/) - Selected for being able to scale and store data with a higher rate of ingest.

## Step by step

### Ingest data

1. Invoke the **Azure Function** from the device client, sending the telemetry data.
2. Validate and forward that data to the **Azure Event Hub**.
3. The **Azure Event Hub** triggers a second **Azure Function** that transforms the data into individual Azure Cosmos DB documents.
4. From the Azure Function target, add a new document to the **Azure Cosmos DB** database with the telemetry data.

### View data

1. Within the game engine, a query is generated sent to an **Azure Function** that converts it into an **Azure Cosmos DB** query.
1. The data is the pulled from **Azure Cosmos DB** and returned to the game engine for visualization.

> [!TIP]
> If you are looking to visualize data in a dashboard, hook up the Azure Cosmos DB database to [Power BI](https://docs.microsoft.com/azure/cosmos-db/powerbi-visualize).

## Deployment template

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes a section summarizing the naming rules and restrictions for Azure services.

>[!NOTE]
> If you're interested in how the ARM template works, review the Azure Resource Manager template documentation from each of the different services leveraged in this reference architecture:
>
> - [Create an Event Hub using Azure Resource Manager template](https://docs.microsoft.com/azure/event-hubs/event-hubs-resource-manager-namespace-event-hub)
> - [Automate resource deployment for your function app in Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code)
> - [Azure Cosmos DB template](https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts)

>[!TIP]
> To run the Azure Functions locally, update the *local.settings.json* file with these same app settings.

## Implementation details

A single [Azure Function App](https://docs.microsoft.com/azure/azure-functions/functions-create-first-azure-function) should contain all the functions shown above and can share the same [Hosting Plan](https://docs.microsoft.com/azure/azure-functions/functions-scale).  

### Ingestion Function

1. Validates the incoming telemetry payload
2. Transforms the data into the expected format for the next stage of the data pipeline
3. Sends the data on to the **Azure Event Hub**
4. Returns 202 if the data was accepted by the **Azure Event Hub** 

### Event Hub Trigger Function

1. Reads the event data payload
2. Creates individual Azure Cosmos DB documents for each event 
3. Uploads the documents to Azure Cosmos DB

### Query Function

1. Parses the client generated query
2. Generates a Azure Cosmos DB SQL formatted query
3. Wraps the results in a JSON object and returns them to the client

Choosing the right pricing plan for your needs will depend on much the telemetry service is used, and what else is running in the same **Azure Function App**.

## Optimization considerations

You can **expire old data automatically stored in Azure Cosmos DB** using [Azure Cosmos DB TTL](https://docs.microsoft.com/azure/cosmos-db/time-to-live) (Time To Live), setting a time horizon where stored documents will be purged.

Events sent to the Ingestion **Azure Function** should be batched on the client to reduce HTTP overhead.  Consider using client-side compression if the batches are large.
>[!TIP]
> Compressing the batches server-side prior to transmission to **Event Hub** can help reduce costs of [Throughput Units](https://docs.microsoft.com/azure/event-hubs/event-hubs-faq#throughput-units).  This is especially helpful if multiple services are consuming the events, or there are other Event Hubs in the same Namespace.

## Additional resources and samples

- [Big data reference architecture and implementation for an online multiplayer game](https://github.com/dgkanatsios/GameAnalyticsEventHubFunctionsCosmosDatalake)
- [Processing 100,000 Events Per Second on Azure Functions](https://blogs.msdn.microsoft.com/appserviceteam/2017/09/19/processing-100000-events-per-second-on-azure-functions/)
- [Reliable Event Processing in Azure Functions (how to avoid losing a message)](https://hackernoon.com/reliable-event-processing-in-azure-functions-37054dc2d0fc)
- [In-order event processing with Azure Functions](https://medium.com/@jeffhollan/in-order-event-processing-with-azure-functions-bb661eb55428)

### Advanced streaming aggregation support

If you are looking for windowing support out-of-the-box, meaning you want to perform set-based computation (aggregation) or other operations over subsets of events that fall within some period of time, then you should consider replacing the Azure Function that connects the Azure Event Hub to Azure Cosmos DB with [Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/windowing-azure-stream-analytics).

[Azure Stream Analytics](https://docs.microsoft.com/stream-analytics-query/windowing-azure-stream-analytics) can be used for adding advanced aggregation scenarios on the event stream.  For example, **Azure Functions**, being stateless, don't natively support [windowing](https://docs.microsoft.com/azure/stream-analytics/stream-analytics-window-functions) on event streams.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures.  The total amount will vary based on usage. See the pricing webpages for each of the services that were used in the reference architecture:

- [Event Hubs pricing](https://azure.microsoft.com/pricing/details/event-hubs/)
- [Azure Functions](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Cosmos DB pricing](https://azure.microsoft.com/pricing/details/cosmos-db/)
- [Azure Stream Analytics pricing](https://azure.microsoft.com/pricing/details/stream-analytics/)
- [Azure Virtual Machines pricing](https://azure.microsoft.com/pricing/details/virtual-machines)

You can also use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) to configure and estimate the costs for the Azure services that you are planning to use. Prices are estimates and are not intended as actual price quotes. Actual prices may vary depending upon the date of purchase, currency of payment, and type of agreement you enter with Microsoft. Contact a Microsoft sales representative for additional information on pricing.