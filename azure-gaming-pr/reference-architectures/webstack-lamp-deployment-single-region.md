---
title: Deploy a single region LAMP architecture
description: Detailed step by step to deploy this architecture using different methods.
author: DavidJimenez
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2019
ms.author: dajimene
ms.service: azure
---

# Deploy a single region LAMP architecture

This document covers different methods to deploy a **single region LAMP architecture**, either using **command line tools** on either Linux bash or Windows batch for a more hands on programmatic setup, or an Azure Resource Manager template for a **one-click deployment**. And in most cases there will be pointers to how to setup a certain portion of the architecture using the **Azure Portal**. Alternatively you can use third-party solutions like [Hashicorp Terraform](https://docs.microsoft.com/azure/terraform/terraform-overview).

In general, when deploying a single region LAMP architecture there are certain steps that are mostly one-offs while others need to be executed in more regular basis as your backend gets updated to match your game requirements. Here below is the full step list:

**Mostly one-offs operations**

1. Deploy a Virtual Machine on a Managed Disk with your preferred Linux OS distribution.
2. Install Apache, your preferred PHP version and other stuff you consider.
3. Deallocate and generalize the Virtual Machine.
4. Capture the Virtual Machine Disk Image to generate the custom golden image.
5. Deploy the networking resources (load balancer, etc).
6. Deploy the Azure Cache for Redis.
7. Deploy the Azure Database for MySQL.
8. Create the Azure Storage account and container.
9. Create your Virtual Machine Scale Set ensuring it references the captured Disk Image as the OS Disk.
1. Setup the autoscale settings.

> [!NOTE]
> You may want in the future to change to another Linux OS version or PHP version, so that would require to recreate the custom golden image (steps 1-4). Or you may want to make changes into the autoscaler (step 10).

**Regular basis operations**

1. Update the Virtual Machine instances from the Virtual Machine Scale Set with any PHP file modifications.

## Preparative

You'll need to have at least an Azure subscription. [Learn more](./webstack-lamp.md#pricing).

Review the [naming conventions](./general-guidelines.md#naming-conventions) for any of the names that you are choosing for the Azure resources that you are going to be creating.

### Command line general configuration variables and tools

Regardless of what step you are working on, it's best practice to keep a set of general variables handy as they are foundational:

- **YOURSUBSCRIPTIONID**: Your Azure subscription ID (format: XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX).
- **REGIONNAME**: The Azure region where the architecture will be deployed.
- **RESOURCEGROUPNAME**: The name of the resource group that will contain all the different Azure services from the architecture. Consider appending the region name as a suffix.
- **PREFIX**: The string that will precede all the Azure services for future identification purposes. i.e: the codename of your game.

#### Get the list of Azure region names

```batch
CALL az account list-locations
```

Here below are some examples of the region names currently available:

| Region | Name |
|--------|------|
| **Japan West** | japanwest |
| **Japan East** | japaneast |
| **Korea Central** | koreacentral |
| **Korea South** | koreasouth |
| **East Asia** | eastasia |
| **Southeast Asia** | southeastasia |
| **Southeast Asia** | southeastasia |
| **Australia Central** | australiacentral |
| **Australia Central 2** | australiacentral2 |

#### Initialize the variables

```bash
SET YOURSUBSCRIPTIONID=XXXXXXXX-XXXX-XXXX-XXXX-XXXXXXXXXXXX
SET RESOURCEGROUPNAME=myResourceGroup
SET REGIONNAME=japaneast
SET LOGINUSERNAME=azureuser
```

#### Tools

Should you choose to setup the architecture programmatically using a command line interface and the samples from this document, you are going to need to install [Azure CLI](https://docs.microsoft.com/cli/azure/install-az-cli2), a cross-platform command-line tool providing a great experience for managing Azure resources. The CLI is designed to make scripting easy, query data, support long-running operations, and more.

## Deploy a Virtual Machine on a Managed Disk

This Virtual Machine only has one specific use: serve as a foundation for the custom golden image. In most cases, it gets deleted afterwards.

> [!TIP]
> When deploying a Virtual Machine, it's recommended to use SSH keys rather than password to protect the access. [Learn more](https://docs.microsoft.com/azure/virtual-machines/linux/create-ssh-keys-detailed).

Regardless of what method it's used to deploy the Virtual Machine on a Managed Disk, if you look within the resource group in the Azure Portal it should look like it's shown below. The exposed resources created would be the Virtual Machine itself, a couple of disks (the one from the Operating System and another for the data), a public IP address, a Network Security Group, a Network Interface Controller (NIC) and finally a Virtual Network.

[![Outcome of creating a virtual machine](media/webstack/webstack-create-vm.png)](media/webstack/webstack-create-vm.png)

### Command line approach using Azure CLI

On top of the general configuration variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **LOGINUSERNAME** | azureuser | | | | The admin username to connect to the Virtual Machine after being deployed.
| **VMNAME** | myVirtualMachine | | | | The name of the Virtual Machine.
| **VMNAME** | Canonical:UbuntuServer:16.04-LTS:latest | | | | The Linux OS that will be installed in the Virtual Machine.
| **VMSIZE** | Standard_B1s | Standard_B1s | Standard_F4s_v2 | Standard_F32s_v2 | Virtual Machine option. Be aware that Premium SSD is not supported in every Virtual Machine option. [Learn more](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#Linux).
| **VMDATADISKSIZEINGB** | 5 | 5 | 10 | 30 | How much persistent disk storage you are going to allocate per Virtual Machine. [Benefits of using managed disks](https://docs.microsoft.com/azure/virtual-machines/windows/managed-disks-overview#benefits-of-managed-disks).

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of a Virtual Machine deployment, you can download the full Bash [1-create-vm.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/1-create-vm.sh) or Windows Batch [1-create-vm.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/1-create-vm.bat) scripts to save you time.

For more details about the process of deploying a Virtual Machine on a Managed Disk, refer to the [Create and Manage Linux VMs with the Azure CLI](https://docs.microsoft.com/azure/virtual-machines/linux/tutorial-manage-vm) tutorial that covers basic Azure virtual machine deployment items such as selecting a VM size, selecting a VM image, and deploying a VM.

#### Initialize the variables

```bat
SET VMNAME=myVirtualMachine
SET IMAGE=Canonical:UbuntuServer:16.04-LTS:latest
SET VMSIZE=Standard_B1s
SET VMDATADISKSIZEINGB=5
```

#### Login

Running this command will open a browser for you to log in with your Azure credentials. [Learn more about this command](https://docs.microsoft.com/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

```bat
CALL az login
```

#### Set the Azure subscription

If you only have subscription, this step is optional. [Learn more about this command](https://docs.microsoft.com/cli/azure/account?view=azure-cli-latest#az-account-set).

# [Bash](#tab/bash)

```bash
az account set \
 --subscription $YOURSUBSCRIPTIONID
```

# [Windows Batch](#tab/bat)

```bat
CALL az account set ^
 --subscription %YOURSUBSCRIPTIONID%
```

---

#### Create a resource group

All resources created in Azure need to be part of a resource group. [Learn more about this command](https://docs.microsoft.com/cli/azure/group?view=azure-cli-latest#az-group-create).

```bat
CALL az group create ^
 --name %RESOURCEGROUPNAME% ^
 --location %REGIONNAME%
```

Have a look at the [general guidelines documentation](./general-guidelines.md#resource-groups) to learn more about resource groups.

#### Create a Virtual Machine

This operation will take several minutes. [Learn more about this command](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-create).

```bat
CALL az vm create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMNAME% ^
 --image %IMAGE% ^
 --size %VMSIZE% ^
 --admin-username %LOGINUSERNAME% ^
 --data-disk-sizes-gb %VMDATADISKSIZEINGB% ^
 --generate-ssh-keys
```

#### Open the ports 80 and 443

[Learn more about this command](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-open-port).

```bat
CALL az vm open-port ^
 --port 80 ^
 --priority 900 ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMNAME%

CALL az vm open-port ^
 --port 443 ^
 --priority 901 ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMNAME%
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Create a Linux virtual machine in the Azure portal](https://docs.microsoft.com/azure/virtual-machines/linux/quick-create-portal) and [Attach a managed data disk to a VM by using the Azure portal](https://docs.microsoft.com/azure/virtual-machines/windows/attach-managed-disk-portal#add-a-data-disk) if you prefer to create the Virtual Machine manually using the Azure Portal.

## Install Apache and PHP

### Get the public IP of the Virtual Machine that was just created

#### Command line approach using Azure CLI

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-list).

```bat
CALL az network public-ip list ^
 --resource-group %RESOURCEGROUPNAME% ^
 --query [].ipAddress
```

#### Azure Portal

Select the **Connect** button on the overview page for your Virtual Machine.

[![Connect to a Virtual Machine via Azure Portal](media/webstack/webstack-connect-to-vm-portal.png)](media/webstack/webstack-connect-to-vm-portal.png)

On the right side of the screen a new blade will be open, in **Login using VM local account** a connection command is shown.

### Connect to the Virtual Machine

Using your preferred local bash shell, paste the SSH connection command into the shell to create an SSH session. The following example shows what the SSH connection command looks like:

```bash
ssh azureuser@[PUBLICIP]
```

### Run the installation commands one by one

The following commands will install Apache and the PHP 7.3 version. You can change to any other PHP version, like 5.6, replacing all the 7.3 references below.

```bash
sudo add-apt-repository -y ppa:ondrej/php

sudo apt-get -y update

export DEBIAN_FRONTEND=noninteractive

sudo apt-get -y install apache2 php7.3 libapache2-mod-php7.3 php7.3-mysql

sudo apt-get -y install php7.3-cli php7.3-fpm php7.3-json php7.3-pdo php7.3-zip php7.3-gd php7.3-mbstring php7.3-curl php7.3-xml php7.3-bcmath php7.3-json

sudo service apache2 start

echo "<?php phpinfo(); ?>" | sudo tee /var/www/html/phpinfo.php > /dev/null

exit
```

> [!TIP]
> You can create a shell script and execute all the commands in a more automated fashion, check [install-apache-and-php.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/scripts/install-apache-and-php.sh) out as an example.

### Validate that the web server and PHP are running properly

Replace the PUBLICIP below with the real IP address of your Virtual Machine. Then open your preferred web browser and try to access the default page going to `[http://[PUBLICIP]` or any of your specific PHPs for example, and check that everything is working as intended.

## Deallocate and generalize the Virtual Machine

### Clean up the virtual machine

To create an image for deployment, you'll need to clean the system and make it ready for re-provisioning.

Use your preferred local bash shell, the account name, and the public IP address of the Virtual Machine to connect to it remotely and deprovision it.

```bash	
ssh azureuser@[PUBLICIP]
sudo waagent -deprovision+user -force
exit 
```

The command above performs the following tasks:

- Removes SSH host keys
- Clears nameserver configuration in /etc/resolvconf
- Removes the root user's password from /etc/shadow
- Removes cached DHCP client leases
- Resets host name to localhost.localdomain
- Deletes the last provisioned user account and it’s data

### Stopping, deallocating and generalizing the virtual machine

Before creating an image it's needed to stop and prepare the Linux guest OS on the Virtual Machine. If you create an image from a Virtual Machine that **hasn't been generalized**, any Virtual Machines created from that image **won't start**. These operations should be really quick to complete.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Virtual Machine preparative, you can download the full Bash [2-prepare-vm.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/2-prepare-vm.sh) or Windows Batch [2-prepare-vm.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/2-prepare-vm.bat) scripts to save you time.

#### Command line approach using Azure CLI

Learn more about the [deallocate](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-deallocate) and [generalize](https://docs.microsoft.com/cli/azure/vm?view=azure-cli-latest#az-vm-generalize) commands.

```bat
CALL az vm deallocate ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMNAME%

CALL az vm generalize ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMNAME%
```

## Capture the Virtual Machine Disk Image to generate the custom golden image

Regardless of what method it's used to create the golden image, if you look within the resource group in the Azure Portal it should look like it's shown below. The only additional resource created would be the Image itself.

[![Outcome of creating a golden image](media/webstack/webstack-create-golden-image.png)](media/webstack/webstack-create-golden-image.png)

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Description|
|----------|----------|-----------|
| **GOLDENIMAGENAME** | myGoldenImage | The name of the custom golden image.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the creation of the custom golden image, you can download the full Bash [3-create-golden-image.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/3-create-golden-image.sh) or Windows Batch [3-create-golden-image.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/3-create-golden-image.bat) scripts to save you time.

#### Initialize the variables

```bat
SET GOLDENIMAGENAME=myGoldenImage
```

#### Create the custom golden image

[Learn more about this command](https://docs.microsoft.com/cli/azure/image?view=azure-cli-latest#az-image-create).

```bat
CALL az image create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --source %VMNAME% ^
 --name %GOLDENIMAGENAME% ^
 --os-type Linux
```

### Azure Resource Manager template

TBD

### Azure Portal

TBD

## Deploy the networking resources

> [!CAUTION]
> This is the portion of the configuration that requires a more careful look as there are multiple networking elements involved, some interconnected.

Here below is an example of how the topology should look like if you have deployed all the networking resources, including the HTTPs traffic health probe that it's only supported in the Standard Load Balancer SKU, and a Virtual Machine Scale Set with a couple of instances.

[![Connect to a Virtual Machine via Azure Portal](media/webstack/webstack-lamp-single-region-networking-topology.png)](media/webstack/webstack-lamp-single-region-networking-topology.png)

Specifically to the networking resources, if you look within the resource group in the Azure Portal it should look like it's shown below. The exposed resources created would be a load balancer, a public IP for the load balancer and a Virtual Network.

[![Outcome of creating a virtual machine](media/webstack/webstack-create-networking.png)](media/webstack/webstack-create-networking.png)

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **LBSKU** | Basic | Basic | Standard | Standard | The Azure Load Balancer SKU. [Learn more](https://azure.microsoft.com/pricing/details/virtual-machines/linux/#Linux).
| **PUBLICIPNAME** | PREFIX + PublicIP | | | | The name to identify the public IP address of the Azure Load Balancer.
| **PUBLICIPALLOCATION** | Static | | | | Dynamic or Static.
| **PUBLICIPVERSION** | Ipv4 | | | | Ipv4 or Ipv6.
| **LBNAME** | PREFIX + LB | | | | The name to identify the Azure Load Balancer.
| **VNETNAME** | PREFIX + VNET | | | | The name to identify the Azure Virtual Network.
| **VNETADDRESSPREFIX** | 10.0.0.0/16 | | | | The Azure Virtual Network address space.
| **SUBNETNAME** | PREFIX + Subnet | | | | The name to identify the subnet.
| **SUBNETADDRESSPREFIX** | 10.0.0.0/24 | | | | The subnet's address range in CIDR notation (e.g. 192.168.1.0/24). It must be contained by the address space of the Azure Virtual Network.
| **LBBEPOOLNAME** | LBNAME + BEPool | | | | The name of the Azure Load Balancer backend pool.
| **LBFENAME** | LBNAME + FE | | | | The name of the Azure Load Balancer frontend IP configuration.
| **LBFEPORTRANGESTART** | 50000 | | | | Frontend IP configuration range start port.
| **LBFEPORTRANGEEND** | 50119 | | | | Frontend IP configuration range end port.
| **LBNATPOOLNAME** | LBNAME + NATPool | | | | The Azure Load Balancer NAT pool name.
| **LBRULEHTTPNAME** | LBNAME + HTTPRule | | | | The Azure Load Balancer inbound NAT rule name for the HTTP connections.
| **LBRULEHTTPSNAME** | | Note: Only Standard SKU | LBNAME + HTTPSRule | LBNAME + HTTPSRule | The Azure Load Balancer inbound NAT rule name for the HTTPs connections.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the deployment of the networking resources, you can download the full Bash [4-create-networking.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/4-create-networking.sh) or Windows Batch [4-create-networking.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/4-create-networking.bat) scripts to save you time.

#### Initialize the variables

```bat
SET LBSKU=Basic
SET PUBLICIPNAME=%PREFIX%PublicIP
SET PUBLICIPALLOCATION=Static
SET PUBLICIPVERSION=IPv4
SET LBNAME=%PREFIX%LB
SET VNETNAME=%PREFIX%VNET
SET VNETADDRESSPREFIX=10.0.0.0/16
SET SUBNETNAME=%PREFIX%Subnet
SET SUBNETADDRESSPREFIX=10.0.0.0/24
SET LBBEPOOLNAME=%LBNAME%BEPool
SET LBFENAME=%LBNAME%FE
SET LBFEPORTRANGESTART=50000
SET LBFEPORTRANGEEND=50119
SET LBNATPOOLNAME=%LBNAME%NATPool
SET LBRULEHTTPNAME=%LBNAME%HTTPRule
SET LBRULEHTTPSNAME=%LBNAME%HTTPSRule
```

#### Create the Azure Virtual Network

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/vnet?view=azure-cli-latest#az-network-vnet-create).

```bat
CALL az network vnet create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VNETNAME% ^
 --address-prefix %VNETADDRESSPREFIX% ^
 --subnet-name %SUBNETNAME% ^
 --subnet-prefix %SUBNETADDRESSPREFIX%
```

#### Create an inbound public IP address for the load balancer

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/public-ip?view=azure-cli-latest#az-network-public-ip-create).

```bat
CALL az network public-ip create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %PUBLICIPNAME% ^
 --allocation-method %PUBLICIPALLOCATION% ^
 --sku %LBSKU% ^
 --version %PUBLICIPVERSION%
```

#### Create an Azure Load Balancer

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/lb?view=azure-cli-latest#az-network-lb-create).

```bat
CALL az network lb create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %LBNAME% ^
 --sku %LBSKU% ^
 --backend-pool-name %LBBEPOOLNAME% ^
 --frontend-ip-name %LBFENAME% ^
 --public-ip-address %PUBLICIPNAME%
```

#### Create an Azure Load Balancer health probe for HTTP

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/lb/probe?view=azure-cli-latest#az-network-lb-probe-create).

```bat
CALL az network lb probe create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --lb-name %LBNAME% ^
 --name http ^
 --protocol http ^
 --port 80 ^
 --path /
```

#### Create an Azure Load Balancer health probe for HTTPs

> [!NOTE]
> This is only supported in the Standard Load Balancer SKU.

```bat
if %LBSKU%==Standard CALL az network lb probe create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --lb-name %LBNAME% ^
 --name https ^
 --protocol https ^
 --port 443 ^
 --path /
```

#### Create an inbound NAT pool with backend port 22

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/lb/inbound-nat-pool?view=azure-cli-latest#az-network-lb-inbound-nat-pool-create).

```bat
CALL az network lb inbound-nat-pool create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %LBNATPOOLNAME% ^
 --backend-port 22 ^
 --frontend-port-range-start %LBFEPORTRANGESTART% ^
 --frontend-port-range-end %LBFEPORTRANGEEND% ^
 --lb-name %LBNAME% ^
 --frontend-ip-name %LBFENAME% ^
 --protocol Tcp
```

#### Create a load balancing inbound rule for the port 80

[Learn more about this command](https://docs.microsoft.com/cli/azure/network/lb/rule?view=azure-cli-latest#az-network-lb-rule-create).

```bat
CALL az network lb rule create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %LBRULEHTTPNAME% ^
 --lb-name %LBNAME% ^
 --protocol tcp ^
 --frontend-port 80 ^
 --backend-port 80 ^
 --probe http ^
 --frontend-ip-name %LBFENAME% ^
 --backend-pool-name %LBBEPOOLNAME%
```

#### Create a load balancing inbound rule for the port 443

> [!NOTE]
> This is only supported in the Standard Load Balancer SKU.

```bat
if %LBSKU%==Standard CALL az network lb rule create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %LBRULEHTTPSNAME% ^
 --lb-name %LBNAME% ^
 --protocol tcp ^
 --frontend-port 443 ^
 --backend-port 443 ^
 --probe https ^
 --frontend-ip-name %LBFENAME% ^
 --backend-pool-name %LBBEPOOLNAME%
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Create a Basic Load Balancer by using the Azure portal](https://docs.microsoft.com/azure/load-balancer/quickstart-create-basic-load-balancer-portal) and [Create a Standard Load Balancer to load balance VMs using the Azure portal](https://docs.microsoft.com/azure/load-balancer/quickstart-load-balancer-standard-public-portal) to learn how to create either Azure Load Balancer SKU using the Azure Portal.

## Deploy the Azure Cache for Redis

Regardless of what method it's used to create the Azure Cache for Redis, if you look within the resource group in the Azure Portal it should look like it's shown below. The only additional resource created would be the cache itself.

[![Outcome of creating a golden image](media/webstack/webstack-create-redis.png)](media/webstack/webstack-create-redis.png)

Deploying the Azure Cache for Redis should take less than 30 minutes in total.

### Command line approach using Azure CLI 

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **REDISNAME** | PREFIX + Redis | |  | | The Azure Cache for Redis name.
| **REDISNAMEUNIQUE** | REDISNAME + [Random number] | | | | **Important**: The name of the Azure Cache for Redis has to be entirely unique across all Azure customers. Hence the scripts use a random generator.
| **REDISVMSIZE** | C1 | C3 | C4 | P4 | Basic/Standard(C0, C1, C2, C3, C4, C5, C6), Premium (P1, P2, P3, P4, P5)
| **REDISSKU** | Standard | Basic | Standard | Premium | Basic – Single node, multiple sizes, ideal for development/test and non-critical workloads. The basic tier has no SLA.<br>Standard – A replicated cache in a two node Primary/Secondary configuration managed by Microsoft, with a high availability SLA.<br>Premium – The new Premium tier includes all the Standard-tier features and more, such as better performance compared to Basic or Standard-tier caches, bigger workloads, data persistence, and enhanced network security.
| **REDISSHARDSTOCREATE** | | Note: Only Premium SKU | Note: Only Premium SKU | 10 | Number of shards per cluster.
| **REDISSUBNETNAME** | | Note: Only Premium SKU | Note: Only Premium SKU | REDISNAME + Subnet | When an Azure Cache for Redis instance is configured with an Azure Virtual Network, it is not publicly addressable and can only be accessed from virtual machines and applications within the Azure Virtual Network. [Learn More](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet).
| **SUBNETADDRESSPREFIX** | | Note: Only Premium SKU | Note: Only Premium SKU | 10.0.1.0/24 | **Important**: When deploying an Azure Cache for Redis to an Azure Virtual Network, the cache must be in a dedicated subnet that contains no other resources except for Azure Cache for Redis instances.
| **SUBNETID** | | Note: Only Premium SKU | Note: Only Premium SKU | SUBNETID | Note: The full string is required.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Cache for Redis deployment, you can download the full Bash [5-create-redis.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/5-create-redis.sh) or Windows Batch [5-create-redis.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/5-create-redis.bat) scripts to save you time.

#### Initialize the variables

```bat
SET REDISNAME=%PREFIX%Redis
SET REDISNAMEUNIQUE=%REDISNAME%%RANDOM%
SET REDISVMSIZE=P1
SET REDISSKU=Premium
SET REDISSHARDSTOCREATE=2
SET VNETNAME=%PREFIX%VNET
SET REDISSUBNETNAME=%REDISNAME%Subnet
SET SUBNETADDRESSPREFIX=10.0.1.0/24
SET SUBNETID=/subscriptions/%YOURSUBSCRIPTIONID%/resourceGroups/%RESOURCEGROUPNAME%/providers/Microsoft.Network/virtualNetworks/%VNETNAME%/subnets/%REDISSUBNETNAME%
```

#### Create a specific subnet named cache

> [!NOTE]
> This is only supported in the Premium Azure Cache for Redis SKU.

```bat
CALL az network vnet subnet create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --vnet-name %VNETNAME% ^
 --name %REDISSUBNETNAME% ^
 --address-prefixes %SUBNETADDRESSPREFIX%
```

#### Create an Azure Cache for Redis

[Learn more about this command](https://docs.microsoft.com/cli/azure/redis?view=azure-cli-latest#az-redis-create).

```bat
CALL az redis create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %REDISNAMEUNIQUE% ^
 --location %REGIONNAME% ^
 --sku %REDISSKU% ^
 --vm-size %REDISVMSIZE% ^
 --shard-count %REDISSHARDSTOCREATE% ^
 --subnet-id %SUBNETID%
```

> [!NOTE]
> Just remove the `--shard-count` and `--subnet-id` if you are using a non-premium SKU or you don't want to setup a cluster and secure the cache behind an Azure Virtual Network.

#### Get details of the cache (hostName, enableNonSslPort, port, sslPort, primaryKey and secondaryKey)

```bat
CALL az redis show ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %REDISNAMEUNIQUE% ^
 --query [hostName,enableNonSslPort,port,sslPort] ^
 --output tsv

CALL az redis list-keys ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %REDISNAMEUNIQUE% ^
 --query [primaryKey,secondaryKey] ^
 --output tsv
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Create a cache](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-dotnet-how-to-use-azure-redis-cache#create-a-cache) to create an Azure Cache for Redis using the Azure Portal. Then refer to [How to configure Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-configure) that describes the configurations available for the Azure Cache for Redis instances.

Refer to [How to configure Redis clustering for a Premium Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-clustering) that describes how to configure clustering in a premium Azure Cache for Redis instance using the Azure Portal.

Refer to [How to configure Virtual Network Support for a Premium Azure Cache for Redis](https://docs.microsoft.com/azure/azure-cache-for-redis/cache-how-to-premium-vnet) that describes how to configure virtual network support for a premium Azure Cache for Redis instance using the Azure Portal.

## Deploy the Azure Database for MySQL

Regardless of what method it's used to create the Azure Cache for Redis, if you look within the resource group in the Azure Portal it should look like it's shown below. The only additional exposed resources created would be the master database itself and, if you created them, the replica or replicas.

[![Outcome of creating a golden image](media/webstack/webstack-create-mysql.png)](media/webstack/webstack-create-mysql.png)

Deploying the Azure Database for MySQL master and replicas should take less than 30 minutes in total.

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **MYSQLNAME** | PREFIX + MySQL | |  | | The name of the MySQL server.
| **MYSQLUSERNAME** | azuremysqluser | | | | The admin username to connect to the MySQL server.
| **MYSQLPASSWORD** | CHang3thisP4Ssw0rD | | | | The admin password to connect to the MySQL server. Change it for whichever you consider, as robust as possible.
| **MYSQLDBNAME** | gamedb | | | | The name of the game database.
| **MYSQLBACKUPRETAINEDDAYS** | 7 | 7 | 15 | 30 | The backup retention period. [Learn more](https://docs.microsoft.com/azure/mysql/concepts-backup).
| **MYSQLGEOREDUNDANTBACKUP** | Disabled | Disabled | Disabled | Enabled | [Learn more](https://docs.microsoft.com/azure/mysql/concepts-backup#backup-redundancy-options). Important: Configuring locally redundant or geo-redundant storage for backup is only allowed during server create. Once the server is provisioned, you cannot change the backup storage redundancy option.
| **MYSQLSKU** | GP_Gen5_2 | GP_Gen5_2 | GP_Gen5_8 | MO_Gen5_16 | **Important**: There is a connection limit depending on the SKU type and number of cores. [Learn more](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers#storage).
| **MYSQLSTORAGEMBSIZE** | 51200 | 51200 | 256000 | 1024000 | Space and IOPS vary depending on the SKU and allocated storage size. [Learn more](https://docs.microsoft.com/azure/mysql/concepts-pricing-tiers#storage).
| **MYSQLVERSION** | 5.7 | 5.7 | 5.7 | 5.7 | MySQL version.
| **MYSQLREADREPLICANAME** | | | MYSQLNAME + Replica | MYSQLNAME + Replica1 ... | Read replica MySQL name.
| **MYSQLREADREPLICAREGION** | | | REGIONNAME | REGIONNAME | Azure region where the read replica will be deployed.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Database for MySQL deployment, you can download the full Bash [6-create-mysql.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/6-create-mysql.sh) or Windows Batch [6-create-mysql.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/6-create-mysql.bat) scripts to save you time.

#### Initialize the variables

```bat
SET MYSQLNAME=%PREFIX%MySQL
SET MYSQLUSERNAME=azuremysqluser
SET MYSQLPASSWORD=CHang3thisP4Ssw0rD
SET MYSQLDBNAME=gamedb
SET MYSQLBACKUPRETAINEDDAYS=7
SET MYSQLGEOREDUNDANTBACKUP=Disabled
SET MYSQLSKU=GP_Gen5_2
SET MYSQLSTORAGEMBSIZE=51200
SET MYSQLVERSION=5.7
SET MYSQLREADREPLICANAME=%MYSQLNAME%Replica
SET MYSQLREADREPLICAREGION=westus
```

#### Enable Azure CLI db-up extension (in preview)

```bat
CALL az extension add --name db-up
```

#### Create the server, database and other routinely tasks

> [!NOTE]
> In addition to creating the server, the az mysql up command creates a sample database, a root user in the database, opens the firewall for Azure services, and creates default firewall rules for the client computer

[Learn more about this command](https://docs.microsoft.com/cli/azure/ext/db-up/mysql?view=azure-cli-latest#ext-db-up-az-mysql-up).

```bat
CALL az mysql up ^
 --resource-group %RESOURCEGROUPNAME% ^
 --server-name %MYSQLNAME% ^
 --admin-user %MYSQLUSERNAME% ^
 --admin-password %MYSQLPASSWORD% ^
 --backup-retention %MYSQLBACKUPRETAINEDDAYS% ^
 --database-name %MYSQLDBNAME% ^
 --geo-redundant-backup %MYSQLGEOREDUNDANTBACKUP% ^
 --location %REGIONNAME% ^
 --sku-name %MYSQLSKU% ^
 --storage-size %MYSQLSTORAGEMBSIZE% ^
 --version=%MYSQLVERSION%
```

#### Create a read replica using the MySQL server as a source (master)

[Learn more about this command](https://docs.microsoft.com/cli/azure/mysql/server/replica?view=azure-cli-latest#az-mysql-server-replica-create).

```bat
CALL az mysql server replica create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %MYSQLREADREPLICANAME% ^
 --source-server %MYSQLNAME% ^
 --location %MYSQLREADREPLICAREGION%
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Design an Azure Database for MySQL database using the Azure portal](https://docs.microsoft.com/azure/mysql/tutorial-design-database-using-portal), to learn how to create and manage your server, configure the firewall and setup the database.

Refer to [How to create and manage read replicas in Azure Database for MySQL using the Azure portal](https://docs.microsoft.com/azure/mysql/howto-read-replicas-portal), to learn how to create and manage read replicas in the Azure Database for MySQL service using the Azure Portal.

## Create the Azure Storage account and container

Regardless of what method it's used to create the Azure Storage account and container, if you look within the resource group in the Azure Portal it should look like it's shown below. The only additional exposed resource created would be the Azure Storage itself.

[![Outcome of creating a golden image](media/webstack/webstack-create-storage.png)](media/webstack/webstack-create-storage.png)

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **STORAGENAME** | mygamebackendstrg%RANDOM% | | | | The name of the storage account. **Important**: The name of the Azure Storage has to be entirely unique across all Azure customers. Hence the scripts use a random generator. And it has to be all lowercase.
| **STORAGESKU** | Standard_LRS | Standard_LRS | Premium_LRS | Premium_LRS | The storage SKU to setup, either standard, premium or ultra.
| **STORAGECONTAINERNAME** | %STORAGENAME%cntnr | | | | The blobs need to be stored within a container.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Storage and container deployment, you can download the full Bash [7-create-storage.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/7-create-storage.sh) or Windows Batch [7-create-storage.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/7-create-storage.bat) scripts to save you time.

#### Initialize variables

```bat
SET STORAGENAME=mygamebackendstrg%RANDOM%
SET STORAGESKU=Standard_LRS
SET STORAGECONTAINERNAME=%STORAGENAME%cntnr
```

#### Create a storage account

[Learn more about this command](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create).

```bat
CALL az storage account create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %STORAGENAME% ^
 --sku %STORAGESKU% ^
 --location %REGIONNAME%
```

#### Get the connection string from the storage account

```bat
CALL az storage account show-connection-string -n %STORAGENAME% -g %RESOURCEGROUPNAME% --query connectionString -o tsv > connectionstring.tmp
SET /p STORAGECONNECTIONSTRING=<connectionstring.tmp
CALL DEL connectionstring.tmp
```

#### Create a storage container into the storage account

[Learn more about this command](https://docs.microsoft.com/cli/azure/storage/container?view=azure-cli-latest#az-storage-container-create).

```bat
CALL az storage container create ^
 --name %STORAGECONTAINERNAME% ^
 --connection-string %STORAGECONNECTIONSTRING%
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Create a storage account](https://docs.microsoft.com/azure/storage/common/storage-quickstart-create-account?tabs=azure-portal#create-a-storage-account-1), showing you how to create an Azure Storage account using the Azure Portal.

Refer to [Create a container](https://docs.microsoft.com/azure/storage/blobs/storage-quickstart-blobs-portal#create-a-container), showing you how to create an storage containe in the Azure portal.

## Create a Virtual Machine Scale Set

A virtual machine scale set allows you to deploy and manage a set of identical, auto-scaling virtual machines.

Scale sets have an "upgrade policy" that determine how VMs are brought up-to-date with the latest scale set model. The three modes for the upgrade policy are:

- **Automatic** - In this mode, the scale set makes no guarantees about the order of VMs being brought down. The scale set may take down all VMs at the same time, which may cause down time.
- **Manual** - In this mode, when you update the scale set model, nothing happens to existing VMs. It isn't the most suitable to use when the number of instances is high and you don't have any automation to handle the updates.
- **Rolling** - In this mode, the scale set rolls out the update in batches with an optional pause time between batches. Rolling upgrade updates only a portion of the instances from the scale set at a time, meaning your game should be prepared to handle that, at the same time, a subset of the backend servers may be running the older version while the rest is up to date; eventually all the servers will be up to date. Rolling upgrade requires that a health probe is associated to the Virtual Machine Scale Set and also all the Virtual Machine instances.

Regardless of what method it's used to create the Azure Virtual Machine Scale Set, if you look within the resource group in the Azure Portal it should look like it's shown below. The only additional exposed resource created would be the scale set itself.

[![Outcome of creating a golden image](media/webstack/webstack-create-vmss.png)](media/webstack/webstack-create-vmss.png)

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **VMSSNAME** | PREFIX + VMSS | | | | The name of the scale set.
| **VMSSSKUSIZE** | Standard_B1s | Standard_B1s | Standard_F4s_v2 | Standard_F32s_v2 | The SKU to setup, either standard, premium or ultra.
| **VMSSVMTOCREATE** | 2 | 2 | 10 | 20 | The number of Virtual Machine instances that will be deployed upon creation of the scale set.
| **VMSSSTORAGETYPE** | Standard_LRS | Standard_LRS | Premium_LRS | Premium_LRS | The storage SKU to setup, either standard, premium or ultra.
| **VMSSACELERATEDNETWORKING** | false | false | true | true | [Learn more](https://docs.microsoft.com/azure/virtual-network/create-vm-accelerated-networking-cli#benefits) about the benefits of accelerated networking.
| **VMSSUPGRADEPOLICY** | Manual | Manual | Rolling | Rolling | Manual, Automatic or Rolling. Explained above.
| **HEALTHPROBEID** |  |  | Use the health probe ID | Use the health probe ID | Required if Rolling upgrade mode.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Storage and container deployment, you can download the full Bash [8-create-vmss.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/8-create-vmss.sh) or Windows Batch [8-create-vmss.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/8-create-vmss.bat) scripts to save you time.

#### Initialize variables

```bat
SET VMSSNAME=%PREFIX%VMSS
SET GOLDENIMAGENAME=myGoldenImage
SET VMSSSKUSIZE=Standard_B1s
SET VMSSVMTOCREATE=2
SET VMSSSTORAGETYPE=Premium_LRS
SET VMSSACELERATEDNETWORKING=false
SET VMSSUPGRADEPOLICY=Manual
SET HEALTHPROBEID=/subscriptions/%YOURSUBSCRIPTIONID%/resourceGroups/%RESOURCEGROUPNAME%/providers/Microsoft.Network/loadBalancers/%LBNAME%/probes/http
SET VMSSOVERPROVISIONING=--disable-overprovision
```

#### Create a scale set

[Learn more about this command](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az-vmss-create).

```bat
CALL az vmss create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMSSNAME% ^
 --image %GOLDENIMAGENAME% ^
 --upgrade-policy-mode %VMSSUPGRADEPOLICY% ^
 --load-balancer %LBNAME% ^
 --lb-sku %LBSKU% ^
 --vnet-name %VNETNAME% ^
 --subnet %SUBNETNAME% ^
 --admin-username %LOGINUSERNAME% ^
 --instance-count %VMSSVMTOCREATE% ^
 --backend-pool-name %LBBEPOOLNAME% ^
 --storage-sku %VMSSSTORAGETYPE% ^
 --vm-sku %VMSSSKUSIZE% ^
 --lb-nat-pool-name %LBNATPOOLNAME% ^
 --accelerated-networking %VMSSACELERATEDNETWORKING% ^
 --generate-ssh-keys %VMSSOVERPROVISIONING%
```

#### Confirm scale set upgrade policy

```bat
CALL az vmss show ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMSSNAME% ^
 --query upgradePolicy
```

#### Associate the load balancer health probe to the scale set

[Learn more about this command](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az-vmss-show).

```bat
CALL az vmss update ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMSSNAME% ^
 --query virtualMachineProfile.networkProfile.healthProbe ^
 --set virtualMachineProfile.networkProfile.healthProbe.id='%HEALTHPROBEID%'
```

#### Update all the instances from the scale set

[Learn more about this command](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az-vmss-update-instances).

```bat
CALL az vmss update-instances ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMSSNAME% ^
 --instance-ids *
```

#### Switch to Rolling upgrade mode

```bat
CALL az vmss update ^
 --resource-group %RESOURCEGROUPNAME% ^
 --name %VMSSNAME% ^
 --query upgradePolicy ^
 --set upgradePolicy.mode=Rolling
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Create a virtual machine scale set in the Azure portal](https://docs.microsoft.com/azure/virtual-machine-scale-sets/quick-create-portal) to learn how to deploy a VM scale set using the Azure Portal.

## Create the autoscaler

It monitors the performance of the Virtual Machine instances in your scale set. These autoscale rules increase or decrease the number of Virtual Machine instances in response to these performance metrics.

Regardless of what method it's used to create the autoscaler, it won't show up  directly exposed resource within the resource group unless you enable the **Show hidden types** checkbox. Once that's enable, the autoscaler should look like it's shown below.

[![Outcome of creating a virtual machine](media/webstack/webstack-create-autoscaler.png)](media/webstack/webstack-create-autoscaler.png)

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Small T-shirt value|Medium T-shirt value|Large T-shirt value|Description|
|----------|----------|-----------|----------|----------|-----------|
| **VMSSAUTOSCALERNAME** | PREFIX + Autoscaler | | | | The name of the autoscaler.
| **VMSSAUTOSCALERCRITERIA** | Percentage CPU | Percentage CPU | Percentage CPU | Percentage CPU | The general criteria that will be used to trigger the autoscaler. [Learn more](https://docs.microsoft.com/azure/azure-monitor/platform/metrics-supported#microsoftcomputevirtualmachinescalesets).
| **VMSSAUTOSCALERMAXCOUNT** | 10 | 10 | 15 | 40 | The maximum number of Virtual Machines that will be part of the scale set. After that number is reached, the autoscaler won't create more.
| **VMSSAUTOSCALERMINCOUNT** | 2 | 2 | 10 | 20 | The minimum number of Virtual Machines that will be part of the scale set. After that number is reached, the autoscaler won't switch off any more. Usually matches VMSSVMTOCREATE
| **VMSSAUTOSCALERUPTRIGGER** | 50 avg 5m | 50 avg 5m | 50 avg 5m | 50 avg 5m | Condition to trigger the scaling out operation. [Learn more](https://docs.microsoft.com/Azure/azure-monitor/platform/autoscale-best-practices) about best practices.
| **VMSSAUTOSCALERDOWNTRIGGER** | 30 avg 5m | 30 avg 5m | 30 avg 5m | 30 avg 5m | Condition to trigger the scaling in operation. [Learn more](https://docs.microsoft.com/Azure/azure-monitor/platform/autoscale-best-practices) about best practices.
| **VMSSAUTOSCALEROUTINCREASE** | 1 | 1 | 2 | 3 | How many new instances will be added to the scale set upon being triggered the scale out request.
| **VMSSAUTOSCALERINDECREASE** | 1 | 1 | 2 | 3 | How many instances will be removed from  the scale set upon being triggered the scale in request.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Storage and container deployment, you can download the full Bash [9-create-autoscaler.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/9-create-autoscaler.sh) or Windows Batch [9-create-autoscaler.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/9-create-autoscaler.bat) scripts to save you time.

#### Initialize variables

```bat
SET VMSSAUTOSCALERNAME=%PREFIX%Autoscaler
SET VMSSAUTOSCALERCRITERIA=Percentage CPU
SET VMSSAUTOSCALERMAXCOUNT=10
SET VMSSAUTOSCALERMINCOUNT=%VMSSVMTOCREATE%
SET VMSSAUTOSCALERUPTRIGGER=50 avg 5m
SET VMSSAUTOSCALERDOWNTRIGGER=30 avg 5m
SET VMSSAUTOSCALEROUTINCREASE=1
SET VMSSAUTOSCALERINDECREASE=1
```

#### Define the autoscaling profile

[Learn more about this command](https://docs.microsoft.com/cli/azure/monitor/autoscale?view=azure-cli-latest#az-monitor-autoscale-create).

```bat
CALL az monitor autoscale create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --resource %VMSSNAME% ^
 --resource-type Microsoft.Compute/virtualMachineScaleSets ^
 --name %VMSSAUTOSCALERNAME% ^
 --min-count %VMSSAUTOSCALERMINCOUNT% ^
 --max-count %VMSSAUTOSCALERMAXCOUNT% ^
 --count %VMSSVMTOCREATE%
```

#### Enable virtual machine autoscaler for scaling out

[Learn more about this command](https://docs.microsoft.com/cli/azure/monitor/autoscale/rule?view=azure-cli-latest#az-monitor-autoscale-rule-create).

```bat
CALL az monitor autoscale rule create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --autoscale-name %VMSSAUTOSCALERNAME% ^
 --condition "%VMSSAUTOSCALERCRITERIA% > %VMSSAUTOSCALERUPTRIGGER%" ^
 --scale out %VMSSAUTOSCALEROUTINCREASE%
```

#### Enable virtual machine autoscaler for scaling in

```bat
CALL az monitor autoscale rule create ^
 --resource-group %RESOURCEGROUPNAME% ^
 --autoscale-name %VMSSAUTOSCALERNAME% ^
 --condition "%VMSSAUTOSCALERCRITERIA% < %VMSSAUTOSCALERDOWNTRIGGER%" ^
 --scale in %VMSSAUTOSCALERINDECREASE%
```

### Azure Resource Manager template

TBD

### Azure Portal

Refer to [Automatically scale a virtual machine scale set in the Azure portal](https://docs.microsoft.com/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-portal), showing you how to create autoscale rules in the Azure Portal.

## Update the Virtual Machine instances from the Virtual Machine Scale Set

In this document it's covered a simple way to upload a bunch of PHP files to the remote virtual machine instances. You may want to use a more sophisticated system like Azure DevOps, in that case you could start with a couple of good documents: [Deploying Applications to Azure Virtual Machine Scale Sets](https://devblogs.microsoft.com/devops/deploying-applications-to-azure-vm-scale-sets/) or [Using Azure DevOps to Deploy Web Applications to Virtual Machines](https://devblogs.microsoft.com/premier-developer/using-azure-devops-to-deploy-web-applications-to-virtual-machines/).

### Command line approach using Azure CLI

On top of the previously defined variables, the following variables are also being used:

|Variable|Default value|Description|
|----------|----------|-----------|
| **BLOBSOURCEURI** | app\\package.tar.gz | The path to the compressed file (TAR GZIP) containing the PHP files.
| **BLOBFILEDESTINATIONNAME** | package.tar.gz | The filename of the compressed file in the destination.
| **SCRIPTUPDATESOURCEURI** | scripts\\update-app.sh | The path to the bash script that will decompress the PHP file remotely.
| **SCRIPTUPDATEFILEDESTINATIONAME** | update-app.sh | The filename of the update script file in the destination.
| **DESTINATIONFOLDER** | /var/www/html | Directory where the PHP files will be decompressed into.
| **SERVICETORESTART** | apache2.service | Service to restart after the PHP files are decompressed.

> [!TIP]
> In addition to the following documented individual commands and the order of execution, for you to understand each portion of the Azure Storage and container deployment, you can download the full Bash [10-update-app.sh](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/bash/10-update-app].sh) or Windows Batch [10-update-app.bat](https://github.com/Azure-Samples/gaming-lamp/blob/master/azurecli/windowsbatch/10-update-app.bat) scripts to save you time.

#### Initialize variables

```bat
SET BLOBSOURCEURI=app\\package.tar.gz
SET BLOBFILEDESTINATIONNAME=package.tar.gz
SET SCRIPTUPDATESOURCEURI=scripts\\update-app.sh
SET SCRIPTUPDATEFILEDESTINATIONAME=update-app.sh
SET DESTINATIONFOLDER=/var/www/html
SET SERVICETORESTART=apache2.service
```

#### Get the connection string from the storage account

[Learn more about this command](https://docs.microsoft.com/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-show-connection-string).

```bat
CALL az storage account show-connection-string -n %STORAGENAME% -g %RESOURCEGROUPNAME% --query connectionString -o tsv > connectionstring.tmp
SET /p STORAGECONNECTIONSTRING=<connectionstring.tmp
CALL DEL connectionstring.tmp
```

#### Upload both the application files and update application script to the blob storage

[Learn more about this command](https://docs.microsoft.com/cli/azure/storage/blob?view=azure-cli-latest#az-storage-blob-upload).

```bat
CALL az storage blob upload ^
 -c %STORAGECONTAINERNAME% ^
 -f %BLOBSOURCEURI% ^
 -n %BLOBFILEDESTINATIONNAME% ^
 --connection-string %STORAGECONNECTIONSTRING%

CALL az storage blob upload ^
 -c %STORAGECONTAINERNAME% ^
 -f %SCRIPTUPDATESOURCEURI% ^
 -n %SCRIPTUPDATEFILEDESTINATIONAME% ^
 --connection-string %STORAGECONNECTIONSTRING%
```

#### Get the URLs from the uploaded files

[Learn more about this command](https://docs.microsoft.com/cli/azure/storage/blob?view=azure-cli-latest#az-storage-blob-url)

```bat
CALL az storage blob url -c %STORAGECONTAINERNAME% -n %BLOBFILEDESTINATIONNAME% -o tsv --connection-string %STORAGECONNECTIONSTRING% > bloburl.tmp
SET /p BLOBURL=<bloburl.tmp
CALL DEL bloburl.tmp

CALL az storage blob url -c %STORAGECONTAINERNAME% -n %SCRIPTUPDATEFILEDESTINATIONAME% -o tsv --connection-string %STORAGECONNECTIONSTRING% > scripturl.tmp
SET /p SCRIPTURL=<scripturl.tmp
CALL DEL scripturl.tmp
```

#### Build the Protected Settings JSON string

It'll be used by the Custom Script Extension to download the file or files from the storage account. [Learn more about this command](https://docs.microsoft.com/cli/azure/storage/account/keys?view=azure-cli-latest#az-storage-account-keys-list)

```bat
CALL az storage account keys list --account-name %STORAGENAME% --resource-group %RESOURCEGROUPNAME% --query [0].value --output tsv > storagekey.tmp
SET /p STORAGEKEY=<storagekey.tmp
CALL DEL storagekey.tmp

SET PROTECTEDSETTINGS="{\"storageAccountName\":\"%STORAGENAME%\",\"storageAccountKey\":\"%STORAGEKEY%\"}"
SET SETTINGS="{\"fileUris\":[\"%BLOBURL%\",\"%SCRIPTURL%\"],\"commandToExecute\":\"bash %SCRIPTUPDATEFILEDESTINATIONAME% %BLOBFILEDESTINATIONNAME% %DESTINATIONFOLDER% %SERVICETORESTART%\"}"
```

#### Update the configuration file from the scale set

[Learn more about this command](https://docs.microsoft.com/cli/azure/vmss/extension?view=azure-cli-latest#az-vmss-extension-set).

```bat
CALL az vmss extension set ^
 --resource-group %RESOURCEGROUPNAME% ^
 --vmss-name %VMSSNAME% ^
 --publisher Microsoft.Azure.Extensions ^
 --name CustomScript ^
 --version 2.0 ^
 --settings %SETTINGS% ^
 --force-update ^
 --protected-settings %PROTECTEDSETTINGS%
```

#### Update all the instances from the scale set

This will ensure that in the next update round they download and install the files uploaded to the storage account. [Learn more about this command](https://docs.microsoft.com/cli/azure/vmss?view=azure-cli-latest#az-vmss-update-instances).

```bat
CALL az vmss update-instances ^
 --instance-ids * ^
 --name %VMSSNAME% ^
 --resource-group %RESOURCEGROUPNAME%
```

### Azure Resource Manager template

TBD

### Azure Portal

TBD