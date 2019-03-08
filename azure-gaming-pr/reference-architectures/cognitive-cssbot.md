---
title: Customer Service Bot
description: Provide your players a conversational assistant tailored to your game that understands natural language. This assistant can answer questions defined in one or more knowledge sets containing information from marketing materials to troubleshooting guides and everything in between.
author: BrianPeek
manager: timheuer
keywords: 
ms.topic: reference-architecture
ms.date: 10/29/2018
ms.author: brpeek
ms.service: azure
---

# Customer Service Bot Reference Architecture

Provide your players a conversational assistant tailored to your game that understands natural language.  This assistant can answer questions defined in one or more knowledge sets containing information from marketing materials to troubleshooting guides and everything in between.

[![Customer service bot look and feel](media/cognitive/cognitive-customerservicebot-qna.png)](media/cognitive/cognitive-customerservicebot-qna.png)

## Architecture Diagram

[![Customer service bot reference architecture](media/cognitive/cognitive-customerservicebot.png)](media/cognitive/cognitive-customerservicebot.png)

## Architecture Services

- [Azure Bot Service](https://docs.microsoft.com/azure/bot-service/): Azure out-of-the-box solution for building serverless and scalable bots.
- [Azure Language Understanding (LUIS)](https://docs.microsoft.com/azure/cognitive-services/luis/what-is-luis): Applies custom machine-learning intelligence to a user's conversational, natural language text to predict overall meaning, and pull out relevant, detailed information.
- [Azure QnA Maker](https://docs.microsoft.com/azure/cognitive-services/QnAMaker/overview/overview): Creates a question and answer repository of data based on information you provide.
- [Azure Application Insights](https://docs.microsoft.com/azure/azure-monitor/app/app-insights-overview): Optionally used to monitor the customer service bot. It will automatically detect performance anomalies and help understand what users actually do with the bot.

## Architecture considerations

This reference architecture will provide support for **two different knowledge databases**: one for game information and the other for technical support information.

You can, of course, extend and support more knowledge databases of your own.  Please see the [Azure Search pricing page](https://azure.microsoft.com/pricing/details/search/) to find out what pricing tier is more adequate for you, as each knowledge database requires the use of one search index.

## Deployment

At the present time, it's not possible to automate the deployment of this reference architecture, as some of the services are managed through separate portals. Follow these steps to setup the backend:

### Azure Language Understanding Service (LUIS)

1. Create a new application at the [LUIS portal](https://luis.ai/).  Note that this app will not not show up in the Azure portal, it lives on the LUIS portal only.
1. Add intents ("Game" and "Support", for example, or whatever names make sense for your scenario).  When these intents are recognized, you can determine what responses to provide to the user.
1. For each intent, add some questions that a user might enter.  For example, in the "Game" intent, you might have a question like, "Where can I find the sword?" or "How do I beat the second boss?"
1. After you have created your intents and questions, you will need to train the service.  Click the Train button at the top of the page.
1. After training press the Publish button to make the service live.
1. Under the Manage tab, you will find several bits of information.  Make a note of the following:
   - Application Information tab
     - Application ID
   - Keys and Endpoints tab
      - Key 1
      - Endpoint

### QnA Maker Service

There are two tiers for QnA Maker.  Pease read through the pricing information for the service located at the bottom of this page. Be aware that you will have to pay for Azure App Service, Azure Search, and optionally Azure Insights as part of the QnA Maker service.

1. Navigate to the [QnA Maker portal](https://qnamaker.ai/).
1. Click **Create a knoweldge base** at the top of the page.
1. If this is your first knowledge base, create a QnA Maker service by clicking the button provided.  This will redirect you to the Azure portal.
    - Name it *azgaming-refarch-bot-qna* (for example)
    - Choose the pricing tiers that make sense for your usage and select appropriate locations for the resources.  Note that the Search pricing tier will determine how many different knowledge bases you can create, and also note that the service itself will use a single index, so you are left with N-1 indexes/knowledge bases.
1. Once complete, back at the QnA Maker portal, click the Refresh button and select the service you just created.
1. Name the knowledge base accordingly (similar to the "Game" and "Support" names we used earlier).
1. If you have an existing FAQ or other information, you can enter it here.  Otherwise, click the Create your KB button.
1. Once created, select the **My knowledge bases** tab, and click your knowledge base.  Here, you can add key/value pairs for questions and answers.  For example, above we created a question of "Where can I find the sword?" in LUIS.  Here, you would create a key/value pair where the key is "sword" and the value would be the answer of "You will find the sword in the woods."  Of course, these are free-form entries, use what makes sense for your scenario.
1. Repeat these steps for each knowledge base you wish to respond to.

### Azure Bot Service

1. In the Azure Portal, create a **Web App Bot**.
   - The free tier is just fine, but note [other things](https://azure.microsoft.com/pricing/details/bot-service/) you could end up paying for
   - Note: you can use the same app service as the one you created above
1. For the **Bot template** selection, select the **SDK v3** option and choose the **Language understanding** option.
1. After the bot is created, go to the **Application settings** item in the Web App Bot you just created.
1. Find the **LuisAppId** entry and set its value to the Application ID you received above in the LUIS steps.
1. You can modify the code as shown in [this article](https://docs.microsoft.com/azure/cognitive-services/QnAMaker/tutorials/integrate-qnamaker-luis#change-code-in-basicluisdialogcs), or see the code in the [GitHub repo](https://github.com/TODO) for this project.
1. With the code in place and built, you can test the bot using the **Test in Web Chat** option in the Azure portal.

### Channels

In order for the public to use this bot, you will need to set up which "channels" it runs on.  Please see [this article](https://docs.microsoft.com/azure/bot-service/bot-service-manage-channels) for information on how to configure each channel.

## Additional Resources and Samples

[Use bot with QnA Maker and LUIS to distribute your knowledge base](https://docs.microsoft.com/azure/cognitive-services/QnAMaker/tutorials/integrate-qnamaker-luis)

## Pricing

If you don't have an Azure subscription, create a [free account](https://aka.ms/azfreegamedev) to get started with 12 months of free services. You're not charged for services included for free with Azure free account, unless you exceed the limits of these services. Learn how to check usage through the [Azure Portal](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-on-the-azure-portal) or through the [usage file](https://docs.microsoft.com/azure/billing/billing-check-free-service-usage#check-usage-through-the-usage-file).

You are responsible for the cost of the Azure services used while running these reference architectures. See the pricing webpages for each of the services that were used in the reference architecture:

- [Azure Bot Service](https://azure.microsoft.com/pricing/details/bot-service/)
- [Azure Language Understanding (LUIS)](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/)
- [Azure QnA Maker](https://azure.microsoft.com/pricing/details/cognitive-services/qna-maker/)
- [Azure Search pricing](https://azure.microsoft.com/pricing/details/search/)
- [Azure Application Insights](https://azure.microsoft.com/pricing/details/monitor/)

You can also use the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) to configure and estimate the costs for the Azure services that you are planning to use.