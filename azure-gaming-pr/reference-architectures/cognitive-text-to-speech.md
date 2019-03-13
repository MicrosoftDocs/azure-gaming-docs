---
title: Text to Speech
description: In the case of players with reading disabilities, being able to hear text messages that were sent out using text to speech services may help bringing them into the conversation.
author: BrianPeek
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: brpeek
ms.service: azure
---

# Text to Speech Reference Architecture

Help bring everyone into the conversation by converting text messages to audio using **text to speech**.

We're going to describe the solution listed here on [GitHub](https://github.com/Azure-Samples/gaming-cognitive-services-text-to-speech). Keep in mind that the code from this reference architecture is only an example for guidance and there may be a few places to optimize the code pattern before it's ready for production.

## Architecture Diagram

[![Text to speech reference architecture](media/cognitive/cognitive-text-to-speech.png)](media/cognitive/cognitive-text-to-speech.png)

## Architecture Services

- [Azure Event Hub](https://docs.microsoft.com/azure/event-hubs/event-hubs-about) - Chosen as it keeps the order of the messages received.
- [Azure Function](https://docs.microsoft.com/azure/azure-functions/functions-overview) - Simplest way to run code on-demand in the cloud.
- [Azure Content Moderator](https://docs.microsoft.com/azure/cognitive-services/content-moderator/overview) - Included to avoid detect profanity or other undesirable language.
- [Azure Text Analytics](https://docs.microsoft.com/azure/cognitive-services/text-analytics/overview) - This service detects the language used by the player, which is required for the Azure Speech service. Alternatively, [Azure Content Moderator](https://docs.microsoft.com/azure/cognitive-services/content-moderator/overview) can also detect the language.
- [Azure Speech](https://docs.microsoft.com/azure/cognitive-services/speech-service/text-to-speech) - The service that provides the text to speech functionality.
- [Azure Premium Blob Storage](https://docs.microsoft.com/azure/storage/blobs/storage-blob-storage-tiers) - Selected due to latency requirements, as the standard Azure Blob Storage may have limitations if the voice-file read is on demand during game play.

## Architecture Considerations

You only need to create a single Azure Event Hub namespace that contains the 2 Azure Event Hubs used for sending and receiving the messages respectively.

There is a message limit of 245,760 bytes for each event sent.  When the result of the text to speech service is serialized, it will be too large for this buffer.  Instead, saving the result in a persistent storage and passing a pointer to the stored item as part of the return message is a feasible approach.

When enabling this functionality in your game, keep in mind the following variables:

- **Voices and languages supported** - For a complete list of voices, see the [language support](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech) topic.
- **Regions supported** - For information about regional availability, see the [regions](https://docs.microsoft.com/azure/cognitive-services/speech-service/regions#text-to-speech) topic.
- **Audio outputs** - There is a list of supported audio formats. Each incorporates a bitrate and encoding type. The Speech Service supports 24-KHz and 16-KHz audio outputs. For all the details see the [audio outputs](https://docs.microsoft.com/azure/cognitive-services/speech-service/rest-apis#audio-outputs) topic.

## Deployment Template

Click the following button to deploy the project to your Azure subscription:

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure-Samples%2Fgaming-cognitive-services-text-to-speech%2Fmaster%2Fazuredeploy.json" target="_blank"><img src="media/azure-resource-manager-deploy-button.png"/></a>

This operation will trigger a template deployment of the [azuredeploy.json](https://github.com/Azure-Samples/gaming-cognitive-services-text-to-speech/blob/master/azuredeploy.json) ARM template file to your Azure subscription, which will create the necessary Azure resources.

Have a look at the [general guidelines documentation](./general-guidelines.md#naming-conventions) that 
includes a section summarizing the naming rules and restrictions for Azure services.

>[!NOTE]
> If you're interested in how the ARM template works, review the Azure Resource Manager template documentation from each of the different services leveraged in this reference architecture:
>
> - [Create an Event Hub using Azure Resource Manager template](https://docs.microsoft.com/azure/event-hubs/event-hubs-resource-manager-namespace-event-hub)
> - [Automate resource deployment for your function app in Azure Functions](https://docs.microsoft.com/azure/azure-functions/functions-infrastructure-as-code)

There are two types of Azure Cognitive Services subscriptions. The first is a subscription to a single service, such as Computer Vision or the Speech Services. A single-service subscription is restricted to just that service. The second type is a multi-service subscription. This allows you to use a single subscription for multiple Azure Cognitive Services. This option also consolidates billing. To make this reference architecture as modular as possible, the cognitive services are each setup as a single service.

Finally, add these Function [application settings](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings) so the sample project can connect to the Azure services:

- EVENTHUB_CONNECTION_STRING - The [connection string](https://docs.microsoft.com/azure/event-hubs/event-hubs-get-connection-string) to the Azure Event Hub namespace that was created
- TEXTANALYTICS_KEY - The [access key](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#access-your-resource) for the Azure Text Analytics cognitive service that was created
- SPEECH_KEY - The [access key](https://docs.microsoft.com/azure/cognitive-services/cognitive-services-apis-create-account#access-your-resource) for the Azure Speech Cognitive Service that was created.

>[!TIP]
> To run the Azure Functions locally, update the *local.settings.json* file with these same app settings.

## Step by Step

1. The player's device opens a persistent and encrypted connection to the chat server in a specific region determined by **Azure Traffic Manager**. The chat server creates a process responsible for maintaining connectivity with the player along with some basic metadata.
1. The player's client sends a chat message to the chat server over the secure connection previously created. The player's process in the chat server receives the message, decrypts it, and parses it.
1. Standard validations are run, and the chat server calls the **Azure Event Hub** service, as it keeps the message order.
1. The **Azure Event Hub** serves as an input trigger for an **Azure Function**.
1. Optionally, yet a best practice, the **Azure Function** invokes the **Azure Content Moderator** Cognitive Service to review the content.
1. The **Azure Function** then invokes the **Azure Text Analytics** service to detect the language used by the player.
1. The Azure Function then uses that information to submit a request to start the conversion to audio. The **Azure Speech** service's response body is an audio file.
1. The Azure Function saves the audio file into persistent storage (**Azure Premium Blob Storage**) and gets the stored location.
1. Another **Azure Event Hub** is set as the output for the **Azure Function** and receives the pointer to the persistent storage containing the audio file.
1. The chat server receives the result from the **Azure Event Hub**.
1. The chat server reads the voice file from persistent storage using the pointer received.
1. The chat message and the audio file is sent to the recipient players' processes in the chat server. The processes run further validation on the text message, serialize both text and audio, and send them to the recipient players' devices over their secure connections. Finally the text is displayed in the chat screen and the audio file is played.

### Blob Storage Clean Up

Be diligent about cleaning up the audio files saved in persistent storage.  See the [managing Blob storage lifecycle](https://docs.microsoft.com/azure/storage/blobs/storage-lifecycle-management-concepts) documentation for more information.

### Azure Text to Speech Service

If you are looking for **samples** of the Text to Speech Cognitive Service, see [Microsoft Speech Service API: Text-to-Speech Samples](https://github.com/Azure-Samples/Cognitive-Speech-TTS).

### Azure Text Analytics

This service is required to detect the language of the chat string submitted by the player. At the moment, the service is only able to return the ISO 639-1 name ("en", "fr", etc) meaning a conversion table is going to be required as the Text to Speech language codes are a more granular, supporting specific language and dialect. For the full list see [language and region support for Speech Service API](https://docs.microsoft.com/azure/cognitive-services/speech-service/language-support#text-to-speech).

Alternatively, instead of the conversion table, you could let your players choose their preferred local language (i.e: Mexican Spanish instead of Argentinian Spanish) and the voice as part of the game settings.

### Alternatives

Azure Content Moderator can also detect the language of a string sent for moderation, meaning you could leverage it instead of Azure Text Analytics for that purpose, with the added benefit of having the string moderated.

## Security Considerations

Do not hard-code any Event Hub or Cognitive Services connection strings into the source of the Function.  Instead, at a minimum, leverage the [Function App Settings](https://docs.microsoft.com/azure/azure-functions/functions-how-to-use-azure-function-app-settings#manage-app-service-settings) or, for even stronger security, use [Key Vault](https://docs.microsoft.com/azure/key-vault/) instead. There is a tutorial explaining how to [create a Key Vault](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/create-an-azure-key-vault-and-secret/), how to [use a managed service identity with a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/using-managed-service-identity-msi-with-and-azure-app-service-or-an-azure-function/) and finally how to [read the secret stored in Key Vault from a Function](https://blogs.msdn.microsoft.com/benjaminperkins/2018/06/13/how-to-connect-to-a-database-from-an-azure-function-using-azure-key-vault/).

Review the [Event Hub authentication and security model overview](https://docs.microsoft.com/azure/event-hubs/event-hubs-authentication-and-security-model-overview) and put it into practice to ensure only your chat server can talk to the Event Hub.

## Additional Resources and Samples

[Azure Event Hubs SDK for Unity](https://docs.microsoft.com/sandbox/gamedev/unity/azure-event-hubs-unity): This is a sandbox project. The content in this article is unsupported, and therefore may be out of date or not in a working state.

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures.  The total amount will vary based on usage. See the pricing webpages for each of the services that were used in the reference architecture:

- [Azure Traffic Manager](https://azure.microsoft.com/pricing/details/traffic-manager/)
- [Azure Event Hub](https://azure.microsoft.com/pricing/details/event-hubs/)
- [Azure Function](https://azure.microsoft.com/pricing/details/functions/)
- [Azure Content Moderator](https://azure.microsoft.com/pricing/details/cognitive-services/content-moderator/)
- [Azure Text Analytics](https://azure.microsoft.com/pricing/details/cognitive-services/text-analytics/)
- [Azure Premium Blob Storage](https://azure.microsoft.com/pricing/details/storage/blobs/)

You can also use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) to configure and estimate the costs for the Azure services that you are planning to use.