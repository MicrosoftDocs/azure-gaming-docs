---
title: "What's happening to Azure Game Development Virtual Machine"
description: "Learn about Azure Game Development Virtual Machine retirement and the new game development options."
author:  joannaleecy
ms.author: joanlee
ms.date: 11/01/2023
ms.topic: overview
ms.prod: azure-gaming
---

# What's happening to Azure Game Development Virtual Machine?

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

Azure Game Development Virtual Machine is scheduled for retirement by February 1st, 2024. While the decision has been made to retire this Azure Marketplace Application, we remain committed to our game developers with more [GPU optimized virtual machine options](/azure/virtual-machines/sizes-gpu), which are tailored to 3D game creation.  

In addition, we’re providing the scripts for building Azure VM for game development purposes as open source in [Azure Virtual Directory (AVD) Landing Zone Accelerator repository](https://github.com/Azure/avdaccelerator/tree/main/workload/terraform/example/gamedevwm/packer).

In scenarios where dedicated GPUs are not required, consider using [Microsoft Dev Box](https://azure.microsoft.com/products/dev-box/), which is a managed Azure service that enables developers to create on-demand, high-performance, secure, ready-to-code, project-specific workstations in the cloud.

## Support timeline

The following notes outline the timeline for support.

|Key dates |What happens |
|----------|-------------|
|On or before Jan 2nd, 2024 |No substantial product changes. Microsoft will still support the product without adding new features. Users can still:  <br/><ul><li>Create new Game Dev VMs. <br/><li>Use existing Game Dev VMs.<ul/><br/>          |
|After Jan 2nd, 2024, and on or before Feb 1st, 2024 | The Game Dev VM marketplace application will not be visible for new Azure subscriptions. <br/><ul><li>Creation of new Azure Game Development Virtual Machines will only be available for existing Game Dev VM users. <br/><li>Any Game Dev VM created before Jan 2nd, 2024, will continue to run.  |
|After Feb 1st, 2024 | The Game Dev VM marketplace application will no longer be listed in Azure marketplace. <br/><ul><li>The creation of new Azure Game Development Virtual Machines will not be available for any users. <br/><li>Existing virtual machines will continue to run for as long as you keep those resources provisioned in your Azure subscription.|

## Next steps

- Use Azure Virtual Machines with [GPU SKUs](/azure/virtual-machines/sizes-gpu) and explore building your own game dev environment [using Azure Virtual Directory (AVD) Landing Zone Accelerator repository](https://github.com/Azure/avdaccelerator/tree/main/workload/terraform/example/gamedevwm/packer).  

- If you have a support plan and need technical help, create a [support request](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/overview) by following instructions on [Troubleshoot Azure Game Development Virtual Machine and Get Support](/gaming/azure/game-dev-virtual-machine/troubleshoot-support).
