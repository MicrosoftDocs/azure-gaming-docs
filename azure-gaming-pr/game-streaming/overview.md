---
title: Entitlement Data for Activision Blizzard Games overview
description: Entitlement Data for Activision Blizzard Games overview.
author: nascrims
ms.topic: overview
ms.date: 12/4/2023
ms.author: nascrims
ms.prod: azure-gaming
---

# Entitlement Data for Eligible Games 

Please note that under the EC Commitments, Microsoft committed to provide, with consumer consent, access to Entitlement Data (e.g., whether a consumer has already purchased an Eligible Game or whether a consumer is an active subscriber to a multi-game subscription service that includes an Eligible Game) through a standard interface. This obligation is limited to Streaming Services that are already licensed to provide cloud game streaming by at least one Major Game Publisher at the time the Streaming Service seeks access to a Microsoft Game Store. Should a cloud game streaming provider want avail itself of this provision, it should fill out the Streaming Provider Application [here](https://www.xbox.com). For more information on this, please see the full text of the EC Commitments on the European Commission’s website [here](https://ec.europa.eu/competition/mergers/cases1/202330/M_10646_9311516_7443_3.pdf)). Additional Frequently Asked Questions about the EC Commitments are available [here](https://www.xbox.com/en-US/legal/activision-blizzard-cloud-game-streaming-eu/FAQ)

As Eligible Activision Blizzard games are available on both Battle.net and the Microsoft Store, there are two separate Entitlement endpoints for Streaming Providers to call for comprehensive Entitlement Data. For both APIs, Streaming Providers will need to perform secure Service-to-Service (S2S) with OAuth Credentials. Providers will need to share their Azure Application App ID and Tenant ID with Microsoft and Blizzard to allow list the Provider's application as a secure caller. Providers will also need to trigger a consent dialog for a user to consent to sharing their Entitlement Data from each Store.

This page will document the Entitlement API for Microsoft Store Entitlement Data of Eligible Activision Blizzard games and subgames. For documentation on the Battle.net Entitlement API, please read more [here](https://www.xbox.com).

# Entitlement Data Application and Onboarding 

To obtain Entitlement Data from Microsoft and Battle.net, here is a high level overview of the process for a Streaming Provider: 
1. Apply for Entitlement Data via Streaming Provider License form on [https://www.xbox.com]
2. Review API documentation on Microsoft Game Dev and Battle.net Developer Portal. 
3. Create a free Azure account if you do not have one already. 
4. Create a free Azure Application. 
5. Send Azure Application ID and Tenant ID to the game streaming provider email alias, abkstreaming@microsoft.com. 
6. Game streaming provider alias will confirm your application has been allow listed to call the Entitlement APIs, and credentials will be provided via email. 
7. Streaming Provider may now call Entitlement APIs from their application. 
