---
title: Azure Cloud Build Pipelines - Simple build distribution
description: This section of the guide shows how to set up a storage account for build distribution. This is part 6 of an 8 part series.
author: whenslunch
keywords: 
ms.topic: reference-architecture
ms.date: 3/18/2022
ms.author: tzong
ms.service: azure-gaming
---

# Section 5: Simple build distribution

[![Azure Cloud Build Section 5 Overview](media/cloud-build-pipeline/acb5-dist/acb5-roadmap.png)](media/cloud-build-pipeline/acb5-dist/acb5-roadmap.png)

We will use an Azure Blob Storage container to hold all our builds. Let’s set that up now.

1.In your Azure subscription, create a Storage account

[![Storage Account](media/cloud-build-pipeline/acb5-dist/storageaccount.png)](media/cloud-build-pipeline/acb5-dist/storageaccount.png)

2. Enter your **Subscription**, **resource group**, **storage account name**, and **region**.

[![Create Storage Account 1](media/cloud-build-pipeline/acb5-dist/createstorageaccount1.png)](media/cloud-build-pipeline/acb5-dist/createstorageaccount1.png)

3. For **Performance and Redundancy**, you can leave the default choices.

4. At this point, you can click on **Review + create**, then **Create**, for the purposes of this demo. Feel free, however, to customize networking, security and other settings.

5. Once the storage account is provisioned, go to the resource. 

[![Create Storage Account 2](media/cloud-build-pipeline/acb5-dist/createstorageaccount2.png)](media/cloud-build-pipeline/acb5-dist/createstorageaccount2.png)

6. Now, create a container within the new storage account. In the left settings menu, click on **Containers**, then **+Container**.

7. In the New container blade:
    - Name: Enter the name of your new container
    - Public access level: select Container (anonymous read access for containers and blobs)

> [!NOTE
>
> This access level is for demo purposes only. Please do not store sensitive information in this container, alternatively secure your setup with information from [here](/azure/architecture/framework/services/storage/storage-accounts/security).

8. Click **Create.**

:pencil: ***Save this info!*** :pencil:

-	Storage account name
-	Container name

## Next steps

Next, go to Section 6: [Azure DevOps Pipeline](./azurecloudbuilds-6-azdopipeline.md).

Or return to the [Introduction](./azurecloudbuilds-0-intro.md).

Troubleshooting page is [here](./azurecloudbuilds-9-troubleshooting.md).
