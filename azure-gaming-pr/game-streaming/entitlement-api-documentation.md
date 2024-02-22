---
title: Activision Blizzard Games Entitlement API Documentation 
description: Technical documetnation of Entitlement API. 
author: nascrims
ms.topic: overview
ms.date: 12/14/2023
ms.author: nascrims
ms.service: azure-gaming
---


# Entitlement Endpoint Technical Documentation 



> [!NOTE]
> Exact endpoint credentials will be shared once a Streaming Provider has shared their App ID and Tenant ID with the abkstreaming@microsoft.com email alias. 



## Summary  

This endpoint expects the caller to provide an MSA V2 token and a Market and returns the list of products the user has access to for the given market. Markets are two letter country codes (ex. 'us','fr','mx'), and the list of entitlements returned may vary based on the market. The product information returned is a Microsoft Store product ID, which can be "hydrated" with a call to our [Collections](/gaming/gdk/_content/gc/commerce/service-to-service/microsoft-store-apis/xstore-v9-query-for-products) endpoint to extract product information such as product name, product description, etc.

### Request example 
```
    GET  /entitlements?market=neutral 
```

Passing in 'neutral' will return entitlements for all markets for a given user.

### Headers 
```
         Authorization: {MSA v2 Token with Library.Read scope} 

         MS-CV: {A Correlation Vector to trace individual requests} 
```
See [GitHub - microsoft/CorrelationVector](https://github.com/microsoft/CorrelationVector) for more details and implementation examples. 


### Response example 
```JSON 
{ 
    "entitlements": [ 

        { 

            "productId": "9NBLGGH52PH9", 

            "skuId": "0010" 

        }, 

    ] 

} 
```

### Other responses 
```
    204 – request is valid, but user has no entitlements for the given market. 

    400 – market is missing  

    401 – Auth is missing or invalid. 
```

### See also 
* [Entitlement Data for Activision Blizzard Games overview](overview.md)
* [Microsoft Store Service APIs](/gaming/gdk/_content/gc/commerce/service-to-service/microsoft-store-apis/xstore-v9-query-for-products)