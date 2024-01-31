---
title: 'Quickstart: Remoting into the Game Development Virtual Machine with Teradici' 
description: Access the Azure Game Development Virtual Machine using Teradici.
author: meaghanlewis
ms.topic: quickstart
ms.date: 03/10/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Quickstart: Remoting into the Game Development Virtual Machine with Teradici

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

Access the Game Development Virtual Machine using Teradici CAS (Cloud Access Software).

Using Teradici’s PCoIP protocol, connect to the VM for a highly responsive, color-accurate, lossless, and distortion-free experience. Teradici’s Windows PCoIP agent is pre-installed on the VM and requires a [Teradici client](https://docs.teradici.com/find/product/software-and-mobile-clients) to connect to the VM.

## Prerequisites

- Teradici CAS License
- Teradici PCoIP Client software
- An Azure Game Development Virtual Machine

## Connect Teradici with your Game Development Virtual Machine

1. Go to the Azure Portal and search for and select the Game Development Virtual Machine. Click the Create button.
2. As you fill out the desired VM configurations, be sure to select Teradici from the dropdown under the Remote Access Configuration tab and input your pre-purchased license in the Registration Key textbox.
3. Deploy the Game Development Virtual Machine by clicking the Create button at the bottom of the Review + create tab.
4. While the VM is deploying, [install the Teradici PCoIP Client software](https://docs.teradici.com/find/product/software-and-mobile-clients) on the device where you will initiate the remote connection from.
5. Start the Teradici PCoIP Client and wait until the game development VM is successfully deployed.
6. Once the VM is ready, to connect to it you must first input the Public IP address or DNS name of your Azure Game Development Virtual Machine in the Host Address or Code field.

:::image type="content" source="./media/remote-to-vm-with-teradici/teradici-azure-client-host-address.png" alt-text="Screenshot showing the Teradici PCoIP connection window":::

> [!NOTE]
> You can find IP address and DNS name of deployed VM from its Overview blade. Make sure your VM is in the Running Status. If this VM doesn’t have a Public IP address, you can use Private IP address instead. You need to ensure the machine which you initiate the remote connection from also has a private IP address that is routable to your Azure VM’s Private IP address.

7. Optional: Give a name of your connection in the Connection Name field.
8. Click NEXT
9. As Teradici uses self-signed certificates as the default, you may notice a window pop up that warns you that the certificate cannot be verified due to the certificate not being registered with a trusted certificate authority. It doesn’t mean the connection is insecure, as it is still using HTTPs encryption with a self-signed private key, and you can click the Connect Insecurely to login to the VM. Learn more about [configuring trusted signed certificates with Teradici](https://www.teradici.com/web-help/pcoip_connection_manager_security_gateway/19.08/security/creating_cmsg_cert/#default-certificate), which is recommended when deploying a production system.  

:::image type="content" source="./media/remote-to-vm-with-teradici/teradici-connect-insecurely.png" alt-text="Screenshot showing that Teradici PCoIP will connect insecurely since the connection cannot be verified":::

10. Type in your Azure VM credentials and click LOGIN.

> [!NOTE]
> If this is the first time signing into this VM, you’ll be prompted immediately to accept Epic Games store End User License Agreement (EULA) by using your Epic Games account.

:::image type="content" source="./media/remote-to-vm-with-teradici/teradici-pcoip-vm-credentials.png" alt-text="Screenshot showing the Teradici login window with username and password fields":::

11. You're ready to start using the tools that are installed and configured on the game development VM, while taking advantage of Teradici’s powerful technology to improve your game development experience with beautiful color accuracy, responsive interactive streaming and support for a wide array of USB peripherals critical in game creation.

## Clean up resources

None.

## Next steps

- Explore the tools on the Game Development Virtual Machine by opening the Start menu.
- Learn about the game development on Azure by reading [Azure for Gaming](/gaming/azure/) and trying out tutorials.
- Read the article about the [Game Development Virtual Machine](./overview.md).
- [Subscribe To Teradici CAS License for Azure Virtual Desktop](https://ms.portal.azure.com/#create/teradici.teradici_cloud_access_software)
- [Learn more about Teradici All Access, or request for a demo](https://connect.teradici.com/contact-us)
