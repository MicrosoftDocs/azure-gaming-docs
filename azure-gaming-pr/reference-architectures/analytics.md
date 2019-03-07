---
title: Analytics Reference Architectures - Azure
description: These reference architectures describes a variety of analytics use cases and implementations with different alternatives, enabling you to architect your own cloud solution so you can have full control and customization to fit your game design like a glove.
manager: 
keywords: analytics 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Analytics Reference Architectures

These reference architectures describes a variety of analytics use cases and implementations with different alternatives, enabling you to architect your own cloud solution so you can have **full control and customization to fit your game design** like a glove.

> [!TIP]
> If you are looking for an **out-of-the-box solution**, [PlayFab](https://playfab.com/features/#analytics) is a complete back-end platform for building, launching, and growing cloud connected games that has [analytics support](https://api.playfab.com/docs/tutorials/landing-players/).

## Use Cases

There are multiple variables which can be taken into consideration when designing an analytics system. Here are some examples:

1. **Processing time**: real-time, bulk/batch or mixed.
2. **Stage**: development only, production only or both.
3. **Presentation**: dashboard (slice and dice), in-game (directly into the game engine) or both.
4. **Message source**: direct connection between the device and the cloud infrastructure or using a proxy.

What doesn't change much is the construction steps that are part of an standard analytics pipeline:

1. **Event production**: this is for example the application, device or gateway that is generating the events that you are interested on capturing.
2. **Event queuing and ingestion**: this is the service that receives the events and dispatches them.
3. **Storage**: this is where the information is saved in a format and place that can be later used for.
4. **Action**: from displaying the data in dashboards to automation to kick-off workflows or machine learning, whatever method suits your analysis and review process.

Following are some analytics use cases for you to explore:

- [Non-real time analytics dashboard](./analytics-nonrealtime.md)
- [In-editor debugging telemetry](./analytics-devtuning.md)

## Additional potential features

Once you have your analytics pipeline established, you can boost it up leveraging **Azure Machine Learning** building for example a [customer churn analysis project](https://docs.microsoft.com/azure/machine-learning/studio/azure-ml-customer-churn-scenario).

## Additional resources and samples

[Azure Data Architecture Guide](https://docs.microsoft.com/azure/architecture/data-guide/)