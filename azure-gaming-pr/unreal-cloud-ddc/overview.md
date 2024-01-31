---
title: What is Unreal Cloud DDC on Azure? 
description: Describes Unreal Cloud DDC on Azure and how to use the Unreal Cloud DDC for game development.
author: joannaleecy
ms.topic: overview
ms.date: 10/20/2022
ms.author: joanlee
ms.service: azure-gaming
---

# What is Unreal Cloud DDC on Azure?

[!INCLUDE [retire-banner](./includes/retire-banner.md)]

Unreal Cloud Derived Data Cache (DDC) on Azure is a [managed application](/azure/azure-resource-manager/managed-applications/overview) that helps you deploy and operate Unreal Cloud DDC technology as a solution in the Azure cloud.

Typically, Unreal assets (.uassets), data stored in source, can't be used as it is within the Unreal Editor or in a final product.
It often requires additional processing to make it suitable for the application or target platform.
Textures and material assets are common examples, though there are many more.

The cost of processing these data locally can be significant and impacts developer efficiency.
In order to avoid this cost every time you load a source asset, the output of previously processed source data, or commonly known *Derived Data*, is cached and can be reused until the point the source data changes again.
The cached derived data is also known as Derived Data Cache (DDC).

Unreal Cloud DDC on Azure provides an efficient, scalable cloud solution to share the derived data cache and reduce local processing cost for all developers. This time and cost savings is most noticeable for studios with geographically distributed teams.

## User experience

When your studio uses Unreal Cloud DDC in Azure, as a user, you'll have the usual seamless experience in Visual Studio with the following upgrades.

* Less time updating your build because these frequently updated derived data resources are in the cloud
* Reduced shader compiler build times by reusing compiled shaders that were previously built
* Easily share latest build results and Unreal project data with others in the studio

Our recommended cloud setup is to use Unreal Engine with Unreal Cloud DDC technology and Visual Studio, leveraging the Perforce source control solution.
This setup is currently used by game studios like The Coalition.

## Setup overview

Your studio administrator is able to set up and operate Unreal Cloud DDC in Azure following these steps.

* Deploy: Create an Unreal Cloud DDC on Azure managed application using the Azure portal interface or an Azure Resource Manager (ARM) template. You can further customize your solution using the advanced options. This allows you to include services like [Azure Storage Account](/azure/storage/blobs/storage-blobs-introduction), [Azure Key Vault](/azure/key-vault/general/overview), [Azure Cosmos DB](/azure/cosmos-db/introduction), and [Azure Public IP addresses](/azure/virtual-network/ip-services/public-ip-addresses).
* Connect: Connect to Unreal Cloud DDC directly from Visual Studio or [Azure Game Development Virtual Machine](https://azure.microsoft.com/products/virtual-machines/game-development-virtual-machines/#overview). Simply provide the Unreal Cloud DDC URL and complete the authentication prompts to connect to your Unreal Cloud DDC deployment.
* Monitor cost and support: Resources are organized in a nested resource group which simplifies cost monitoring and access management. Troubleshooting is provided through Azure support plans.

To get started, see [Set up Unreal Cloud DDC in Azure portal](quickstart-portal.md).

## Studio benefits

Advantages of a cloud-based development environment.

* Scale as team locations and size evolves over time
* Shared storage for assets across studios and over multiple titles
* Improve overall team productivity from reduced build timings
* All teams have access to Unreal project development data and intermediate build results
* Azure role-based access control enables granular settings to protect sensitive data such as build access.
* When used together with Game Dev VM, you're able to move even more of your game development pipeline to the cloud.

By leveraging Unreal Cloud DDC technology to store pre-compute elements, you're able to more efficiently share assets between developers.

* Active replication quickly replicates data without user intervention.
* Content Addressable (CAS) identification of stored content, enables efficient storage, replication, and de-duplication.
* Smart garbage collection self-cleans unreferenced files, or files with low likelihood of being requested based on usage patterns.

## New to Azure?

For users who are new to Azure, these are some resources to help you understand how Azure works and get you set up.

* [Azure for developers overview](/azure/developer/intro/azure-developer-overview)
* [Create Azure resources](/azure/developer/intro/azure-developer-create-resources)
* Different ways you can manage Azure cloud resources:
    * [Azure Portal](https://portal.azure.com/) - A web-based interface designed for managing Azure resources.
    * [Azure PowerShell](/powershell/azure/install-az-ps)
    * [Azure CLI](/cli/azure/install-azure-cli)

## Next steps

* [Set up an instance of Unreal Cloud DDC in the Azure Portal](quickstart-portal.md)
* [Use Bicep to automate the deployment of Unreal Cloud DDC](quickstart-bicep.md)
* [Use GitHub to automate and manage the deployment of Unreal Cloud DDC](quickstart-github.md)
