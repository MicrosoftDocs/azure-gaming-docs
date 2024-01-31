---
title: Entitlement Data for Activision Blizzard Games overview
description: Entitlement Data for Activision Blizzard Games overview.
author: nascrims
ms.topic: overview
ms.date: 12/14/2023
ms.author: nascrims
ms.service: azure-gaming
---

# Entitlement Data for Eligible Games 

Please note that, pursuant to the Commitments entered into by Microsoft and made legally binding by the European Commission in its decision under Article 8(2) of Regulation (EC) 139/2004 in case M. 10646 - Microsoft/Activision Blizzard ("Commitments"), Microsoft committed to provide Streaming Services serving customers in the European Economic Area, with Consumer consent, access to Entitlement Data (e.g., whether a Consumer has already purchased an Eligible Game or whether a consumer is an active subscriber to a multi-game subscription service that includes an Eligible Game) through a standard interface. This obligation is limited to Streaming Services that are already licensed to provide cloud game streaming by at least one Major Game Publisher at the time the Streaming Service seeks access to a Microsoft Game Store. Should a cloud game streaming provider want to avail itself of this provision, it should fill out the streaming provider pplication [here](https://www.xbox.com). For more information on this, please see the [full text of the EC Commitments](https://ec.europa.eu/competition/mergers/cases1/202330/M_10646_9311516_7443_3.pdf) on the European Commission’s website. For additional frequently asked questions about the EC Commitments, see [Activision Blizzard Cloud Game Streaming in the European Economic Area FAQ](https://www.xbox.com/en-US/legal/activision-blizzard-cloud-game-streaming-eu/FAQ)

As Eligible Activision Blizzard games may be available on both Battle.net and/or the Microsoft Store, there are two separate Entitlement endpoints for streaming providers to call for comprehensive Entitlement Data. For both APIs, streaming providers will need to perform secure Service-to-Service (S2S) with OAuth Credentials. Providers will need to share their Azure Application App ID and Tenant ID with Microsoft and Blizzard to allow list the Provider's application as a secure caller. To learn more about this pattern, learn how to [Configure protected web API apps](/entra/identity-platform/scenario-protected-web-api-app-configuration?branch=main&tabs=aspnetcore).

When streaming providers avail themselves of the Streaming Provider License, they will also attest to following certain Data Protection Agreements if they decide to request customer Entitlement Data. This request requires user consent to share Entitlement Data from each Store with the provider, which the provider must request as outlined in each API's documentation.

This page will document the Entitlement API for Microsoft Store Entitlement Data of Eligible Activision Blizzard games and subgames. For documentation on the Battle.net Entitlement API, please read more [here](https://develop.battle.net/documentation).


The following definitions apply, as per the Commitments:

*“Consumers” include consumers based in the EEA who are licensed to play Eligible Games for their personal use pursuant to the Consumer License.*

*“Streaming Service” includes a cloud game streaming service that allows Consumers to play, from the service provider’s cloud-based servers, PC or console games for which the Consumers have already obtained a license for the game (including through either buy-to-play, free-to-play, or subscription).*

*“Major Game Publisher” refers to Tencent, Valve Corporation, Nexon, NetEase, EA, SmileGate Embracer (THQ) - Perfect World, Roblox Corporation, Take Two, Epic Games, Ubisoft, Square Enix, Bandai Namco Entertainment, Sony and Nintendo.*


## Entitlement Data Application and Onboarding 

__If you are a Streaming Provider looking to obtain Entitlement Data from Microsoft and Battle.net, please follow the steps below__. If you encounter any issues, you can send an email to abkstreaming@microsoft.com for help: 

1. Apply for Entitlement Data via the Streaming Provider License form, or email the game streaming provider email alias, abkstreaming@microsoft.com.
1. Review the API documentation on the Azure Gaming learning site (this page) and [Battle.net Developer Portal](https://develop.battle.net/documentation).
1. Create a free Azure account if you do not have one already. For step-by-step tutorial, see - [Create an Azure account](/learn/modules/create-an-azure-account/).
1. Create a free Azure Application on Azure.com.
    1. __Follow the steps below in Azure App Configuration with MSA v2 before the next step!__ 
1. Send your Azure Application ID and Tenant ID to the game streaming provider email alias, abkstreaming@microsoft.com. 
1. We'll confirm that your application has been added to the allow list. This allows your application to call the Entitlement APIs. Credentials will be provided via email. 
1. Streaming Provider may now call Entitlement APIs from their application. 



## Azure App Configuration with MSA v2 

__If you have not previously created an Azure Application to leverage secure service-to-service calls and user authentication, follow these steps before going back to step 5 above:__ 

1. Go to your Azure portal, select App Registration. Provide a friendly name for your application. This app name will be shown users in your game streaming client during collecting user consent to share their entitlement data with you (consent dialog example shown below). 
1. For __Supported Account Type__, select __Accounts in any organizational directory (Any Azure AD directory - Multitenant) and personal Microsoft accounts (e.g. Skype, Xbox)__
1. Under __Redirect URI__, provide a suitable redirect URI. For testing purposes, select __Web__ and enter https://oauth.pstmn.io/v1/callback to use Postman.

![an image showing the Register an application screen in Microsoft Azure as an example with a test name and redirect URI](media/register-azure-application.png)

After registering your Azure Application, configure your secret. For testing using Postman, you can use a client secret. For production, we recommended that you use a certificate. 

![an image showing an Azure Application's certificates and secrets configuration page](media/azure-app-secret.png)

After creating the secret, send the Application (Client) ID and Azure Tenant ID to abkstreaming@microsoft.com for secure access to the Microsoft Store and Battle.net endpoints.  

![an image showing an Azure application's Application ID and Tenant ID in the Overview section](media/azure-app-appid.png)

Streaming Providers will also need to trigger a consent dialog for a user to consent to sharing their Entitlement Data from each Store (Microsoft Store, Battle.net) by calling the authorization endpoint with a particular Scope. The screenshot below shows an example dialog. 

![an image showing a consent dialog example for an application requesting a user's entitlement data](media/oauth-consent.png)

Information on how to call the endpoint with the correct Scopes will be shared directly with the Streaming Provider over email (step 6 above).


### See also 
* [Activision Blizzard Games Entitlement API Documentation](entitlement-api-documentation.md)
* [Activision Blizzard Cloud Game Streaming FAQ](https://www.xbox.com/legal/activision-blizzard-cloud-game-streaming-eu/FAQ)
* [Configure protected web API apps](/entra/identity-platform/scenario-protected-web-api-app-configuration?branch=main&tabs=aspnetcore).
* [Create an Azure account](/learn/modules/create-an-azure-account/)
* [Microsoft Store Service APIs](/gaming/gdk/_content/gc/commerce/service-to-service/microsoft-store-apis/xstore-v9-query-for-products)
