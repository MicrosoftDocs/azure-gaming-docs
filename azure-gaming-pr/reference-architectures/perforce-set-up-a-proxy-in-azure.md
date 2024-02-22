---
title: Set up a Perforce Proxy in Azure
description: Learn how to deploy a Perforce Proxy in Azure
author: erik-jansen
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2022
ms.author: erikja
ms.service: azure-gaming
---

# Set up a Perforce Proxy in Azure

In this guide we’ll show how to set up a Perforce Proxy in Azure. A scenario where a Perforce Proxy is beneficial is to bring Perforce closer to its users, whether that is for working from home scenarios or teams that span the globe. The Perforce Proxy can be used for on premises deployments of the Helix Core commit server, or Azure deployed Helix Core commit servers.

## Prerequisites

To use the Perforce Proxy, a Helix Core commit server needs to be available. For this guide we have installed Helix Core through Perforce’s Enhanced Studio Pack in the Azure Marketplace. It contains Helix Core, Swarm, Hansoft and a Windows workstation, with 5 licenses to go with that.

## End state

[![Perforce Proxy in Azure](media/cloud-build-pipeline/perforce-proxy-architecture.png)](media/cloud-build-pipeline/perforce-proxy-architecture.png)

## Setting up the Proxy machine

The machine’s specifications can be lower than the commit server. For this example, we’ll use a Standard D2s v4. Follow these instructions to set up the Virtual Machine:

1. Sign into the Azure portal at [https://portal.azure.com](https://portal.azure.com).
2. Type **virtual machines** in the search box.
3. Under **Servers**, select **Virtual Machines**.
4. In the **Virtual machines** page, select **Create** -&gt; **Virtual machine** at the top left.
5. Fill out the **Basics**. A few fields to pay attention to:
    1. Choose the correct **Region** you want the proxy to be deployed to
    2. For Image, select **CentOS-based 7.9 - Gen2**. Other distributions can be chosen as well, but here we choose CentOS to keep the proxy the same as the Enhanced Studio Pack.
    3. For size choose **Standard D2s v4**
6. At the **Disks** blade, we’ll add a disk where the cached content can be placed upon
    1. Click **Create and attach a new disk**
    2. Choose the size you need for your cache
7. In the **Networking** blade, it is important to create a Virtual Network that does not have a conflicting address space with the Helix Core Virtual Network.
8. All the input fields on the other blades can be left to their defaults

After the Virtual Machine has been created, we can open the port to allow users to connect to the Proxy.

1. Navigate to the newly created **Resource Group**
2. Select the **Network Security Group**
3. Click on **Inbound security rules** on the left
4. Click **+ Add** on the top of the blade
5. Fill out the input fields:
    1. Source: IP Addresses
    2. Source IP addresses/CIDR ranges: the IP address (range) you are connecting from
    3. Source Port ranges: *
    4. Destination: Any
    5. Service: Custom
    6. Destination port ranges: 1666
    7. Protocol: Any
    8. Action: Allow
    9. Name: Perforce

## Setting up Virtual Network Peering

To have the Proxy communicate with the commit server, we need to make communication possible between the two Virtual Networks. In Azure this is easily done through Vnet Peering. Follow these steps to setup Vnet Peering:

1. In the Azure Portal, find the **Virtual Network** that was created during setting up the Proxy Virtual Machine.
2. Select **Peerings** from the menu on the left
3. Click on **+ Add** on the top of the blade
4. Fill out the input fields
    1. Leave the radio buttons to their defaults
    2. Select the **Virtual Network** from the commit server through the selector (the default name is HXVNET)
    3. Click **Add** to create the peering connection

After creating the peering, you will see that the Peering Status is set to **Updating**. After about 30 seconds it should be in the **Connected** state.

[![Peering result from proxy to commit](media/cloud-build-pipeline/perforce-proxy-vnetpeering-connected-1.png)](media/cloud-build-pipeline/perforce-proxy-vnetpeering-connected-1.png)

When you navigate to the commit server Vnet, you will also be able to see the peering that was created.

[![Peering result from commit to proxy](media/cloud-build-pipeline/perforce-proxy-vnetpeering-connected-2.png)](media/cloud-build-pipeline/perforce-proxy-vnetpeering-connected-2.png)

## Initializing the Cache Disk

Although the disk is attached to the Virtual Machine, we will still need to partition it and make it available for the Operating System. More information on this can be found here. The following steps partition and mount the disk:

1. SSH into the Virtual Machine
2. Move to super user

```bash
sudo su
```

3. Partition the disk

```bash
parted /dev/sdb --script mklabel gpt mkpart xfspart xfs 0% 100%
mkfs.xfs /dev/sdb1
partprobe /dev/sdb1
```

4. Mount the disk

```bash
mkdir /hxproxy
mount /dev/sdb1 /hxproxy
```

5. Configure the Operating System to make the disk available after reboots
    1. Find the UUID of the disk (look for the line starting with /dev/sdb1)

    ```bash
    blkid
    ```
    [![blkid output](media/cloud-build-pipeline/perforce-proxy-disks.png)](media/cloud-build-pipeline/perforce-proxy-disks.png)

    2. Edit fstab

    ```bash
    vi /etc/fstab
    ```

    3. Add a line like this, replacing the UUID found above

    ```bash
    UUID=<<UUID>>       /hxproxy        xfs     defaults,nofail 1       2
    ```

6. Create the Proxy directories

```bash
mkdir /hxproxy/cache
mkdir /hxproxy/logs
```

## Installing Perforce Proxy

On CentOS we can install the Proxy through YUM, but first we must add the package repository to the YUM configuration. The official Perforce documentation on installing Perforce components can be found here.

1. SSH into the Virtual Machine
2. Move to super user

```bash
sudo su
```

3. Import the Perforce repository key

```bash
rpm --import https://package.perforce.com/perforce.pubkey
```

4. Add Perforce repository to YUM config

```bash
vi /etc/yum.repos.d/perforce.repo
```

5. Contents:

```ini
[perforce]
name=Perforce
baseurl=https://package.perforce.com/yum/rhel/7/x86_64
enabled=1
gpgcheck=1
```

6. Install the Proxy

```bash
yum install helix-proxy
```

The configuration of SSL is optional, although in many cases it is considered a requirement. A blogpost on the Perforce website discussing this can be found [here](https://community.perforce.com/s/article/2596). Setting up SSL for the Proxy is done through the following steps:

```bash
# setting up the folder
mkdir /hxproxy/sslkeys
chmod 700 /hxproxy/sslkeys

# configure P4 to know where the certs are
export P4SSLDIR=/hxproxy/sslkeys

# generate key and certificate path
p4p -Gc

# generate fingerprint
p4p -Gf

# trust the commit server's certs
p4 -p ssl:<<COMMIT SERVER PRIVATE IP>>:1666 trust -y
```

## Making Perforce Proxy run on startup

After you have successfully completed all the steps above, the Proxy is ready to run. However, when the machine reboots it does not restart the Proxy process. We need to add the Proxy to the list of services to start during boot-up:

1. SSH into the Virtual Machine
2. Move to super user

```bash
sudo su
```

2. Create a new service definition for p4p deamon

```bash
vi /etc/systemd/system/p4pd.service
```

3. Contents (replace the IP addresses in the ExecStart command):

```ini
[Unit]
Description=P4P Proxy Service
After=network.target
StartLimitIntervalSec=0

[Service]
Type=simple
Restart=always
RestartSec=1
User=root
Environment="P4SSLDIR=/hxproxy/sslkeys"
ExecStart=/opt/perforce/sbin/p4p -p ssl:<<LOCAL PRIVATE IP>>:1666 -t ssl:<<COMMIT SERVER PRIVATE IP>>:1666 -r /hxproxy/cache -L /hxproxy/log/p4p.log -v server=3 -u perforce

[Install]
WantedBy=multi-user.target
```

5. Save the file
6. Start the service

```bash
systemctl start p4pd
```

7. Enable the service so it will automatically start after boot-up

```bash
systemctl enable p4pd
```

## Connect to the Proxy

After you have implemented the steps outlined above, the Proxy is ready to serve users. Using a machine that has the Perforce client tools installed and has its IP address listed in the Network Security Group, we can now connect to the commit server using the proxy. The users and group settings remain the same, just change the IP address to point to the Proxy.

## Troubleshooting

If you cannot connect to the proxy using the Perforce client tools, make sure the network and Network Security Groups are set up correctly.
If the network is configured correctly, make sure the proxy is running by executing:

```bash
sudo systemctl status p4pd
```

If it is not in a running state, have a look at the journal to see if there is an error message that explains what is going on:

```bash
journalctl -u p4pd.service
```

The application is configured to create a logfile called /hxproxy/log/p4p.log. Any issues between the proxy and the commit server will be logged there. 