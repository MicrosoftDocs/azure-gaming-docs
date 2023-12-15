---
title: Entitlement Data for Activision Blizzard Games overview
description: Entitlement Data for Activision Blizzard Games overview.
author: nascrims
ms.topic: overview
ms.date: 12/14/2023
ms.author: nascrims
ms.prod: azure-gaming
---

# Entitlement Data for Eligible Games 

Please note that under the  commitments Microsoft made to the European Commission in connection with its acquisition of Activision, Microsoft committed to provide cloud gaming Streaming Services serving customers in the European Economic Area, with consumer consent, access to Entitlement Data (e.g., whether a consumer has already purchased an Eligible Game or whether a consumer is an active subscriber to a multi-game subscription service that includes an Eligible Game) through a standard interface. This obligation is limited to Streaming Services that are already licensed to provide cloud game streaming by at least one *Major Game Publisher*\* at the time the Streaming Service seeks access to a Microsoft Game Store. Should a cloud game streaming provider want to avail itself of this provision, it should fill out the Streaming Provider Application [here](https://www.xbox.com). For more information on this, please see the full text of the EC Commitments on the European Commission’s website [here](https://ec.europa.eu/competition/mergers/cases1/202330/M_10646_9311516_7443_3.pdf)). Additional Frequently Asked Questions about the EC Commitments are available [here](https://www.xbox.com/en-US/legal/activision-blizzard-cloud-game-streaming-eu/FAQ)

As Eligible Activision Blizzard games may be available on both Battle.net and/or the Microsoft Store, there are two separate Entitlement endpoints for Streaming Providers to call for comprehensive Entitlement Data. For both APIs, Streaming Providers will need to perform secure Service-to-Service (S2S) with OAuth Credentials. Providers will need to share their Azure Application App ID and Tenant ID with Microsoft and Blizzard to allow list the Provider's application as a secure caller. To learn more about this pattern, learn how to [Configure protected web API apps](https://review.learn.microsoft.com/entra/identity-platform/scenario-protected-web-api-app-configuration?branch=main&tabs=aspnetcore).

When Streaming Providers complete the Streaming Provider License, they will also attest to following certain Data Protection Agreements as they request customer Entitlement Data. This request requires user consent to share Entitlement Data from each Store with the provider, which the Provider must request as outlined in each API's documentation.

This page will document the Entitlement API for Microsoft Store Entitlement Data of Eligible Activision Blizzard games and subgames. For documentation on the Battle.net Entitlement API, please read more [here](https://develop.battle.net/documentation).

\**Major Game Publisher includes one or more of: Tencent, Valve Corporation, Nexon, NetEase, EA, SmileGate Embracer (THQ) - Perfect World, Roblox Corporation, Take Two, Epic Games, Ubisoft, Square Enix, Bandai Namco Entertainment, Sony and Nintendo.*



## Entitlement Data Application and Onboarding 

__If you are a Streaming Provider looking to obtain Entitlement Data from Microsoft and Battle.net, please follow the steps below__. If you encounter any issues, you can send an email to abkstreaming@microsoft.com for help: 

1. Apply for Entitlement Data via the Streaming Provider License form.
1. Review the API documentation on the Azure Gaming learning site (this page) and [Battle.net Developer Portal](https://develop.battle.net/documentation).
1. Create a free Azure account if you do not have one already. For step-by-step tutorial, see - [Create an Azure account](/learn/modules/create-an-azure-account/).
1. Create a free Azure Application on azure.com.
    1. __Follow the steps below in Azure App Configuration with MSA v2 before the next step!__ 
1. Send your Azure Application ID and Tenant ID to the game streaming provider email alias, abkstreaming@microsoft.com. 
1. The game streaming provider alias will confirm your application has been allow listed to call the Entitlement APIs, and credentials will be provided via email. 
1. Streaming Provider may now call Entitlement APIs from their application. 



## Azure App Configuration with MSA v2 

__If you have not previously created an Azure Application to leverage secure service-to-service calls and user authentication, follow these steps before going back to step 5 above:__ 

1. Go to your Azure portal, select App Registration. Provide a friendly name for your application. This app name will be shown users in your game streaming client during collecting user consent to share their entitlement data with you (consent dialog example shown below). 
1. For supported account, type select the multi-tenant and personal account option. 
1. Provide your redirect URI. The example below is configured to use postman for testing purposes. 

![an image showing the Register an application screen in Microsoft Azure as an example with a test name and redirect URI](media/register-azure-application.png)

After registering your Azure Application, configure your secret. For testing with postman we are using a client secret, for production it is recommended to use a certificate. 

![an image showing an Azure Application's certificates and secrets configuration page](media/azure-app-secret.png)

After creating the secret, send the Application (Client) Id and Azure Tenant ID to abkstreaming@microsoft.com for secure access to the Microsoft Store and Battle.net endpoints.  

![an image showing an Azure application's Application ID and Tenant ID in the Overview section](media/azure-app-appid.png)

Streaming Providers will also need to trigger a consent dialog for a user to consent to sharing their Entitlement Data from each Store (Microsoft Store, Battle.net) by calling the authorization endpoint with a particular Scope, such as the below example: 

![an image showing a consent dialog example for an application requesting a user's entitlement data](media/oauth-consent.png)

Information on how to call the endpoint with the correct Scopes will be shared directly with the Streaming Provider over email (step 6 above).


### See also 
* [Activision Blizzard Games Entitlement API Documentation](entitlement-api-documentation.md)
* [Activision Blizzard Cloud Game Streaming FAQ](https://www.xbox.com/legal/activision-blizzard-cloud-game-streaming-eu/FAQ)
* [Configure protected web API apps](/entra/identity-platform/scenario-protected-web-api-app-configuration?branch=main&tabs=aspnetcore).
* [Create an Azure account](/learn/modules/create-an-azure-account/)
* [Microsoft Store Service APIs](/gaming/gdk/_content/gc/commerce/service-to-service/microsoft-store-apis/xstore-v9-query-for-products)
