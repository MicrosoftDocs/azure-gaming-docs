---
title: 'Quickstart: Remoting into the Game Development Virtual Machine with Parsec' 
description: Access the Azure Game Development Virtual Machine using Parsec.
author: meaghanlewis
ms.topic: quickstart
ms.date: 03/02/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Quickstart: Remoting into the Game Development Virtual Machine with Parsec

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

Access the Azure Game Development Virtual Machine using Parsec. Parsec is a high-performance remote access technology built for the Media & Entertainment industry. It provides high fidelity and low latency remote access experience. Parsec’s agent is pre-installed on the VM and requires a [Parsec client](https://parsec.app/downloads) to connect to the VM.

## Prerequisites

Check with your Parsec team administrator to ensure that there is an Azure Game Development Virtual Machine matched to your Parsec account in the Parsec admin panel. Usually, your team administrator invites you through the Parsec team’s dashboard. And you create your own account. For more information, please read [Parsec for Teams Onboarding Guide](https://pages.parsec.app/hubfs/AWS%20AMI%20marketplace/Parsec%20for%20Teams%20Onboarding%20Guide.pdf).

> [!NOTE]
> This VM does not support Parsec Individuals (Warp) plan. You need to have a Teams plan or Enterprise plan to use Parsec. [Review the different Parsec plans](https://parsec.app/pricing) to learn more.

## Connect Parsec with your Game Development Virtual Machine

1. Download and install the [Parsec app](https://parsec.app/downloads) on the device where you will initiate the remote connection from.
2. Launch the Parsec app. You will be prompted to Log In. If you don’t know what credentials to use, check with your Parsec team administrator from step 1. This is NOT your Azure VM credentials.

:::image type="content" source="./media/remote-to-vm-with-parsec/Parsec-sign-in.png" alt-text="Screenshot showing the Parsec app sign in form":::

3. Upon logging in successfully, you should be able to see the Azure Game Development VM which is assigned to you. Please make sure the VM is turned on, or else you won’t see it in the App. You may need click Reload button on the top right corner to see your VM.

:::image type="content" source="./media/remote-to-vm-with-parsec/Parsec-connect-to-vm.png" alt-text="Screenshot showing the Parsec app connect to VM button":::

4. Click Connect button in the Your Computer icon.
5. Sign on using your Azure VM credentials.

> [!NOTE]
> If this is the first time you sign on to this VM, you’ll be prompted immediately to accept Epic Games store End User License Agreement (EULA) by using your EPIC account.

6. You’ll be ready to start using the tools that are installed and configured on the VM. Many of the tools can be accessed through Start menu tiles and desktop icons.

For more details about how to use Parsec app, please watch this video: [Parsec for Teams: Walking through the app](https://www.youtube.com/watch?v=OaMl_p64zak). If you need additional information about tips, common issues, and advanced settings of using Parsec, please visit [Parsec support](https://support.parsec.app) or read [Parsec for Teams Onboarding Guide](https://pages.parsec.app/hubfs/AWS%20AMI%20marketplace/Parsec%20for%20Teams%20Onboarding%20Guide.pdf).  

## Clean up resources

None.

## Next steps

- Explore the tools on the Game Dev VM by opening the Start menu.
- Learn about the Game development on Azure by reading [Azure for Gaming](/gaming/azure/) and trying out tutorials.
- Read the article about the [Game Development Virtual Machine](./overview.md).
