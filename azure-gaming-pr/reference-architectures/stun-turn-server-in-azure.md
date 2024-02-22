---
title: Setting up a STUN/TURN Server in Azure
description: Learn how to set up a STUN/TURN Server in Azure
author: Erik Jansen
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2022
ms.author: erikja
ms.service: azure-gaming
---

# Setting up a STUN/TURN Server in Azure

## Overview

When deploying Unreal Pixel Streaming, some of your clients might be on networks that have settings that prevent a successful WebRTC connection. In these cases, Epic suggests using a STUN server, or a TURN server. More information can be found in the [Epic Unreal Pixel Streaming documentation](https://docs.unrealengine.com/4.27/en-US/SharingAndReleasing/PixelStreaming/Hosting/).

## Setting up the machine

The walkthrough assumes a deployed Ubuntu 18.04 machine in Azure. If you are looking how to deploy a Linux VM in Azure, have a look at [this quickstart](/azure/virtual-machines/linux/quick-create-portal) on the Microsoft Docs site.

## Setting up the STUN/TURN Server

[Coturn](https://github.com/coturn/coturn) is an open-source STUN/TURN Server project, with many configuration options. To deploy Coturn, please have a look at [this blogpost on OurCodeWorld](https://ourcodeworld.com/articles/read/1175/how-to-create-and-configure-your-own-stun-turn-server-with-coturn-in-ubuntu-18-04) that perfectly describes the process.

## DNS Option

In the walkthrough DNS is configured. In case you need a domain name, you can [register one in Azure](/azure/app-service/manage-custom-dns-buy-domain), and use [Azure DNS](/azure/dns/dns-overview) to host the DNS for your domain.

## SSL Certificate Option

There are many ways to obtain an SSL certificate. An easy and cheap way is to use [LetsEncrypt](https://letsencrypt.org/). Setting up LetsEncrypt for a server that uses port 80 for a webserver is extremely easy. Since the Coturn server does use port 80, a manual request with a DNS challenge is easiest:

1. Install CertBot

```bash
sudo snap install core; sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

2. Request the certificate

```bash
sudo certbot -d *.yourdomain.com --manual --preferred-challenges dns certonly
```

Follow the steps certbot is guiding you through.

3. Update the Coturn configuration to point to your newly created certificates and restart the service:

```bash
sudo nano /etc/turnserver.conf
sudo systemctl restart coturn
```

## Opening network ports

Depending on what you configure, you’ll need to open some network ports for the STUN and/or TURN Server to be reachable. For TLS the default port is 5349, and it needs to be open for UDP and TCP. To do this, follow these steps:

1. Go to the **Azure Portal** at [https://portal.azure.com](https://portal.azure.com)
2. Go to the **Resource Group** that contains your STUN/TURN Server deployment
3. Select the **Network Security Group**
4. Click **Inbound security rules** on the left
5. Click **+ Add** on the top of the new blade
6. Use the following settings:
    1. Source: Any
    2. Source port ranges: *
    3. Destination: Any
    4. Service: Custom
    5. Destination port ranges: 5349
    6. Protocol: Any
    7. Action: Allow
    8. Priority: Lower than 65000
    9. Name: A descriptive name like Stun-Turn-Port
7. Click **Add** to save the rule

After this your STUN and TURN Server is up and running.
