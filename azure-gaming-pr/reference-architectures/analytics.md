---
title: Analytics Reference Architectures
description: These reference architectures describe a variety of analytics use cases and possible implementations to help you architect your own cloud solution customized for your game's needs.
author: BrianPeek
keywords: analytics 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.prod: azure-gaming
---

# Azure Game Analytics

These reference architectures describe a variety of analytics use cases and implementations with different alternatives, enabling you to architect your own cloud solution so you can have **full control and customization to fit your game design**.

> [!TIP]
> Azure PlayFab is a complete back-end platform for building, launching, and growing games. Learn more about Azure PlayFab's **out-of-the-box** [analytics solutions](/gaming/playfab/?branch=master#pivot=documentation&panel=analytics).

## Use Cases

Analytics is a broad area with many use cases. First, consider which types of analytics are needed by your game. Here are some examples:

1. **Real Time Analytics** - Commonly used for development, operational health, customer support, and launch-window monitoring.
2. **Direct Data Exploration** - Commonly used to answer questions about user behavior below the surface level.
3. **Performance Metrics** - Commonly used to assess the health of the business against established targets.
4. **Custom Reporting** - Commonly used to build graphs and dashboards from custom events in your game. 

Common components of a typical analytics pipeline include:

1. **Events** - The moments captured by your game or services for later analysis. 
2. **Event queues and ingestion** - The service that receives the events and dispatches them.
3. **Storage** - Where events are saved for use in reporting or exploration.
4. **Enrichment** - Jobs that transform event data and generate metrics on a scheduled cadence.
5. **Dashboards** - UI that surfaces event level and metrics data in a visual chart or graph
6. **Query Engine** - Compute resources and query language used to explore the data. 
7. **Import/Export** - Services that move external data sets between storage location. 
8. **Machine Learning** - Enrchiment activities used to create predictive metrics from patterns in your data.

Here are several analytics use cases for you to explore:

- [Non-real time analytics dashboard](./analytics-non-real-time-dashboard.md)
- [In-editor debugging telemetry](./analytics-in-editor-debugging.md)

## Additional potential features

Once you have your analytics pipeline established, you can add things like **Azure Machine Learning** to supplement this data and learn more.  Here is an example using a [customer churn analysis project](/azure/machine-learning/studio/azure-ml-customer-churn-scenario).

## Additional resources and samples

[Azure Data Architecture Guide](/azure/architecture/data-guide/)