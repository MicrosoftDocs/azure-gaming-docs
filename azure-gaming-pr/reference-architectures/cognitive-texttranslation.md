---
title: Text Translation
description: It's not unusual that players in the same game session natively speak different languages and may appreciate receiving both the original message and its translation.
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Text Translation Reference Architecture

It's not unusual that players in the same game session natively speak different languages and may appreciate receiving both the original message and its **translation**.

## Architecture Diagram

[![Text translation reference architecture](media/cognitive/cognitive-texttranslation.png)](media/cognitive/cognitive-texttranslation.png)

## Architecture Services

- [Translator Text](https://docs.microsoft.com/azure/cognitive-services/translator/translator-info-overview): translates text to multiple languages on the fly.
- [Azure Traffic Manager](https://docs.microsoft.com/azure/traffic-manager/traffic-manager-overview) (optional): connects the player to the most appropiate regional zone based on latency where the chat servers would be.
- [Event Hub](https://azure.microsoft.com/services/event-hubs/) - A service tailored for real-time receiving and processing of a large number of events (chat strings in this case), in order, with little configuration or management overhead.
- [Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-overview) -  Serverless compute service to run code on-demand -- in this case, code that invokes the different cognitive services.

## Architecture Considerations

You only need to create a single Event Hub namespace that will contain the 2 Event Hubs used for sending and receiving messages, respectively.

Not every language is supported by the service today.  Please see the [language and region support for the Translator Text API](https://docs.microsoft.com/azure/cognitive-services/translator/language-support) documentation for what languages and regions are supported.

## Deployment Template

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that includes a section summarizing the naming rules and restrictions for Azure services.

>[!NOTE]
> If you're interested in how the ARM template works, review the Azure Resource Manager template documentation from each of the different services leveraged in this reference architecture:
>
> - [Create an Event Hub using Azure Resource Manager template](https://docs.microsoft.com/azure/event-hubs/event-hubs-resource-manager-namespace-event-hub)
> - [Automate resource deployment for your function app in Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code)

There are two types of Azure Cognitive Services subscriptions. The first is a subscription to a single service, such as Computer Vision or the Speech Services. A single-service subscription is restricted to just that service. The second type is a multi-service subscription. This allows you to use a single subscription for multiple Azure Cognitive Services. This option also consolidates billing. To make this reference architecture as modular as possible, the cognitive services are each setup as a single service.

Finally, add these Function [application settings](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings) so the sample project can connect to the Azure services:

- EVENTHUB_CONNECTION_STRING: the [connection string](https://docs.microsoft.com/azure/event-hubs/event-hubs-get-connection-string) to the Azure Event Hub namespace that was created
- TRANSLATORTEXT_KEY: the [access key](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings) used to access the Azure Translator Text Cognitive Service that was created

>[!TIP]
> To run the Azure Functions locally, update the *local.settings.json* file with these same app settings.

## Step by Step

1. The player's device opens a persistent and encrypted connection to the chat server in a specific region determined by **Azure Traffic Manager**. The chat server creates a process responsible for maintaining connectivity with the player along with some basic metadata.
1. The player's client sends a chat message to the chat server over the secure connection previously created. The player's process in the chat server receives the message, decrypts it, and parses it.
1. Standard validations are run, and the chat server calls the **Azure Event Hub** service, as it keeps the message order.
1. The **Azure Event Hub** serves as an input trigger for an **Azure Function**.
1. The Azure Function invokes the **Azure Text Translator** service that reviews the message sent by the player and returns an result.
1. Another **Azure Event Hub** is set as the output for the **Azure Function**.
1. The chat server receives the outcome from the **Azure Event Hub**.
1. Optionally the chat server saves both the original message and the translation from the **Azure Text Translator service** in persistent storage like **Azure Data Lake Storage**.
1. If the result was stored, optionally save into **Azure Data Lake Analytics** for analysis purposes.
1. The result from the **Azure Text Translator** service is then sent to the recipient players' pertinent processes in the chat server. The processes run further validations on the message, serialize it and send it to the recipient players' devices over their secure connections. Finally it's displayed in the chat screen.

### Azure Event Hubs

See this [quickstart](https://docs.microsoft.com/azure/event-hubs/event-hubs-dotnet-framework-getstarted-send) for a simple implementation for sending an event to an Event Hub in multiple programming languages.  Some additional samples are available [here](https://github.com/Azure/azure-event-hubs/tree/master/samples).

### Function triggered by an Azure Event Hub outputting to another Azure Event Hub

When an [Event Hubs trigger Function](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs#trigger) is triggered, the message that triggers it is passed into the Function as a string. Use the [Event Hubs output binding](https://docs.microsoft.com/azure/azure-functions/functions-bindings-event-hubs#output) to write events to an event stream.

```csharp
[return: EventHub("ehigce-output", Connection = "EventHubConnectionAppSetting")]
    public static string Run([EventHubTrigger("ehigce-input", Connection = "EventHubConnectionAppSetting")] string chatString, ILogger log)
```

Check out the [common causes and solutions for 401 Access Denied errors when calling Cognitive Services](https://blogs.msdn.microsoft.com/kwill/2017/05/17/http-401-access-denied-when-calling-azure-cognitive-services-apis/).

## Security considerations

Do not hard-code any Event Hub or Cognitive Services connection strings into the source of the Function.  Instead, at a minimum, leverage the [Function App Settings](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings#manage-app-service-settings) or, for even stronger security, use [Key Vault](https://docs.microsoft.com/azure/key-vault/) instead. There is a tutorial explaining how to [create a Key Vault](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/create-an-azure-key-vault-and-secret/), how to [use a managed service identity with a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/using-managed-service-identity-msi-with-and-azure-app-service-or-an-azure-function/) and finally how to [read the secret stored in Key Vault from a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/how-to-connect-to-a-database-from-an-azure-function-using-azure-key-vault/).

Review the [Event Hub authentication and security model overview](https://docs.microsoft.com/azure/event-hubs/event-hubs-authentication-and-security-model-overview) and put it into practice to ensure only your chat server can talk to the Event Hub.

The Translator Text API can translate behind firewalls using either domain-name or IP filtering. Domain-name filtering is the preferred method. We do not recommend running Microsoft Translator from behind an IP filtered firewall as the setup is likely to break in the future without notice. See [how to translate behind IP firewalls with the Translator Text API](https://docs.microsoft.com/azure/cognitive-services/translator/firewalls) for all the details.

## Scaling

There are two points that could become a bottleneck in this architecture that you should plan for:

1. Cognitive Services scale well but they are throttled by default. Reach out to Azure support if you are planning to make use of them in large scale to increase capacity.
1. The chat server receiving the Azure Event Hub responses will need to scale. Spin up enough virtual machines to address the demand.

## Alternatives

You could consider leveraging [Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-overview) and [Azure Databricks](https://docs.microsoft.com/azure/azure-databricks/what-is-azure-databricks) if you are interested in gathering analytics using an Apache Spark-based platform.

## Additional resources and samples

[Azure Event Hubs SDK for Unity](https://docs.microsoft.com/sandbox/gamedev/unity/azure-event-hubs-unity): This is a sandbox project. The content in this article is unsupported, and therefore may be out of date or not in a working state.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures.  The total amount will vary based on usage. See the pricing webpages for each of the services that were used in the reference architecture:

- [Azure Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/)
- [Azure Event Hub](https://azure.microsoft.com/pricing/details/event-hubs/)
- [Azure Function](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Translator Text](https://azure.microsoft.com/pricing/details/cognitive-services/translator-text-api/)

You can also use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) to configure and estimate the costs for the Azure services that you are planning to use.