---
title: High Availability & Disaster Recovery
description: Learn how to plan for disaster recovery and maintain business continuity for Unreal Cloud DDC.
author: dciborow
ms.topic: conceptual
ms.date: 10/13/2022
ms.author: dciborow
ms.service: azure-gaming
---

# Failover for high availability and disaster recovery

[!INCLUDE [retire-banner](./includes/retire-banner.md)]

To maximize your uptime, plan ahead to maintain business continuity and prepare for disaster recovery with Unreal Cloud DDC.

Microsoft strives to ensure that Azure services are always available. However, unplanned service outages may occur. We recommend having a disaster recovery plan in place for handling regional service outages. In this article, you'll learn how to:

* Plan for a multi-regional deployment of Unreal Cloud DDC and associated resources.
* Maximize chances to recover logs, notebooks, docker images, and other metadata.
* Design for high availability of your solution.
* Initiate a failover to another region.


The following table shows the Azure services are managed by Microsoft and which are managed by you. It also indicates the services that are highly available by default.

| Service | Managed by | High availability by default |
| ----- | ----- | ----- |
| **Unreal Cloud DDC infrastructure** | Microsoft | |
| **Associated resources** |
| Azure Storage | You | |
| Key Vault | You | ✓ |
| **Compute resources** |
| AKS | You |  |
| **Other data stores** such as Azure Storage, SQL Database,<br> Azure Database for PostgreSQL, Azure Database for MySQL, <br>Azure Databricks File System | You | |

The rest of this article describes the actions you need to take to make each of these services highly available.

## Plan for multi-regional deployment

A multi-regional deployment relies on creation of Unreal Cloud DDC and other resources (infrastructure) in two Azure regions. If a regional outage occurs, you can switch to the other region. When planning on where to deploy your resources, consider:

* __Regional availability__: Use regions that are close to your users. To check regional availability for Unreal Cloud DDC, see [Azure products by region](https://azure.microsoft.com/global-infrastructure/services/).
* __Azure paired regions__: Paired regions coordinate platform updates and prioritize recovery efforts where needed. For more information, see [Azure paired regions](/azure/availability-zones/cross-region-replication-azure).
* __Service availability__: Decide whether the resources used by your solution should be hot/hot, hot/warm, or hot/cold.
    
    * __Hot/hot__: Both regions are active at the same time, with one region ready to begin use immediately.
    * __Hot/warm__: Primary region active, secondary region has critical resources (for example, deployed models) ready to start. Non-critical resources would need to be manually deployed in the secondary region.
    * __Hot/cold__: Primary region active, secondary region has Unreal Cloud DDC and other resources deployed, along with needed data. Resources such as models, model deployments, or pipelines would need to be manually deployed.

> [!TIP]
> Depending on your business requirements, you may decide to treat different Unreal Cloud DDC resources differently. For example, you may want to use hot/hot for deployed models (inference), and hot/cold for experiments (training).

Unreal Cloud DDC builds on top of other services. Some services can be configured to replicate to other regions. Others you must manually create in multiple regions. The following table provides a list of services, who is responsible for replication, and an overview of the configuration:

| Azure service | Geo-replicated by | Configuration |
| ----- | ----- | ----- |
| Key Vault | Microsoft | Use the same Key Vault instance with the Unreal Cloud DDC workspace and resources in both regions. Key Vault automatically fails over to a secondary region. For more information, see [Azure Key Vault availability and redundancy](/azure/key-vault/general/disaster-recovery-guidance).|
| Storage Account | You | Unreal Cloud DDC does not support __default storage-account__ failover using geo-redundant storage (GRS), geo-zone-redundant storage (GZRS), read-access geo-redundant storage (RA-GRS), or read-access geo-zone-redundant storage (RA-GZRS). Create a separate storage account for the default storage of each workspace. </br>Create separate storage accounts or services for other data storage. For more information, see [Azure Storage redundancy](/azure/storage/common/storage-redundancy). |
| Application Insights | You | Create Application Insights for the workspace in both regions. To adjust the data-retention period and details, see [Data collection, retention, and storage in Application Insights](/azure/azure-monitor/app/data-retention-privacy#how-long-is-the-data-kept). |

To enable fast recovery and restart in the secondary region, we recommend the following development practices:

* Use Azure Resource Manager templates. Templates are 'infrastructure-as-code', and allow you to quickly deploy services in both regions.
* To avoid drift between the two regions, update your continuous integration and deployment pipelines to deploy to both regions.
* When automating deployments, include the configuration of workspace attached compute resources such as Azure Kubernetes Service.
* Create role assignments for users in both regions.
* Create network resources such as Azure Virtual Networks and private endpoints for both regions. Make sure that users have access to both network environments. For example, VPN and DNS configurations for both virtual networks.


### Compute and data services

Depending on your needs, you may have more compute or data services that are used by Unreal Cloud DDC. For example, you may use Azure Kubernetes Services or Azure SQL Database. Use the following information to learn how to configure these services for high availability.

__Compute resources__

* **Azure Kubernetes Service**: See [Best practices for business continuity and disaster recovery in Azure Kubernetes Service (AKS)](/azure/aks/operator-best-practices-multi-region) and [Create an Azure Kubernetes Service (AKS) cluster that uses availability zones](/azure/aks/availability-zones). If the AKS cluster was created by using the Unreal Cloud DDC Studio, SDK, or CLI, cross-region high availability is not supported.

__Data services__

* **Azure Blob container / Azure Files / Data Lake Storage Gen2**: See [Azure Storage redundancy](/azure/storage/common/storage-redundancy).

> [!TIP]
> If you provide your own customer-managed key to deploy an Unreal Cloud DDC workspace, Azure Cosmos DB is also provisioned within your subscription. In that case, you're responsible for configuring its high-availability settings. See [High availability with Azure Cosmos DB](/azure/cosmos-db/high-availability).

## Design for high availability

### Deploy critical components to multiple regions

Unreal Cloud DDC lets you select multiple regions during your deployment. You can also optionally enable zone redundancy to further reduce the likelihood of an outage.

## Recovery options

### Deployment deletion

If you accidentally deleted your deployment it is currently not possible to recover it. However you are able to retrieve your existing data from the corresponding storage if you follow these steps:
* In the [Azure portal](https://portal.azure.com) navigate to the storage account that was linked to the deleted Unreal Cloud DDC workspace.
* In the Data storage section on the left, select **Storage Accounts**.
* Your data is located on the storage account with the name that contains your deployment.

## Next steps

To learn about repeatable infrastructure deployments with Unreal Cloud DDC, use [Use Bicep to automate the deployment of Unreal Cloud DDC](quickstart-bicep.md).
Or, learn how to use Infrastructure as Code practices with [GitHub to automate and manage the deployment of Unreal Cloud DDC](quickstart-github.md).
