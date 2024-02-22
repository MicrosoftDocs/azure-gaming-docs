---
title: Create a Game Development Virtual Machine with other Game Engines
description: Get up and running with a Windows 11 or Server 2022 Game Development Virtual Machine that includes common game development tools pre-installed. 
author: cshea15
ms.topic: quickstart
ms.date: 12/1/2022
ms.author: chashea
ms.service: azure-gaming
---
# Quickstart: Create a Game Development Virtual Machine with other game engines

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

If you don't need have an Unreal Engine pre-installed, you can still leverage the Game Development Virtual Machine with either Windows 11 or Windows Server 2022 that bundles common game development tools, and then install your game engine of choice after deploying this VM. Some popular game engines in the market which are worth mentioning but not limited to are: [CryENGINE](https://www.cryengine.com/),  [GameMaker](https://gamemaker.io/en), [Godot](https://godotengine.org/), [O3DE](https://www.o3de.org/), and [Unity](https://unity.com/).

> [!NOTE]
> Game Dev VM supports Windows 10 and Windows Server 2019 too, and can be deployed via ARM template. Please refer to <a href="./create-game-development-vm-arm-template.md" target="_blank">Create a Virtual Machine with an ARM template</a> for steps.

## Prerequisites

- An Azure account with an active subscription. [Create an account for free](https://azure.com/free). Please note Azure free accounts do not support GPU enabled virtual machine SKUs. To understand what Azure subscriptions support GPU SKUs, please refer to this <a href="./offer-types.md" target="_blank">offer types chart</a>.
- Multi-tenant Hosting Rights for Windows 11 is required. Verify you have an [eligible Windows 11 subscription license](/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment#subscription-licenses-that-qualify-for-multitenant-hosting-rights) or a [Visual Studio subscription](/azure/virtual-machines/windows/client-images) for dev/test scenarios.

## Deploy your Game Development Virtual Machine

### Create a Game Dev VM instance

1. Go to the [Azure portal](https://portal.azure.com/). Or, the [Azure Game Development VM](https://aka.ms/gamedevvm) marketplace page, click "Get It Now" under the Azure icon, click "Continue" in the pop up window to give consent and then skip the next step .

1. In the Portal, go to the Azure Marketplace and search **Azure Game Development VM**

1. Select the  **Create**  button.

On the **Basics** tab, complete the following information, then select **Next: Game Developer Tools >**:

| Parameters | Value/Description |
|--|--|
| Subscription | Choose the subscription from the drop-down list|
| Resource Group | Create a new or select an existing Resource Group |
| Region | Select the Azure region that's most appropriate. For the best user experience choose the Azure Region closest to the users. Learn more about [Azure Regions](https://azure.microsoft.com/global-infrastructure/regions/) |
| Use as a build server | This is optional. If you do not expect to use 3D applications or work with 3D content on this VM, instead you want to use it for game build purpose, as illustrated in this [build servers example](/azure-gaming-pr/game-dev-virtual-machine/overview.md#using-as-build-servers), you can check this box. It gives you more VM size options since GPU is not required |
| VM Size | This VM currently supports the sizes [NV](/azure/virtual-machines/nv-series.md), [NVv3](/azure/virtual-machines/nvv3-series.md),[T4](/azure/virtual-machines/nct4-v3-series.md),[A10](/azure/virtual-machines/nva10v5-series.md). Choose a size that is appropriate for your workload. Read more about choosing the right GPU SKU size. |
| Virtual machine name | Enter the name of the virtual machine |
| Admin Creds | Enter the local username and password |
| Operating System | Windows 11 or Windows Server 22 |

On the **Game Development Tools** tab complete the following information, then select **Next Remote Access Configuration >**:

| Parameters | Value/Description |
|--|--|
| Game Engine | Choose No game engine installed |
| Perforce Depot | Connect to and sync a Perforce depot if you already have a Perforce Helix Core version control server in place. Learn more [Integrate with a Perforce Depot](./integrate-perforce-depot.md) |
| Game SDK Installed  | Choose which version of GDK to install, Xbox console development, there will need to be additional [steps to enable this development](/gaming/gdk/_content/gc/tools-console/gc-tools-console-toc.md), as specified in the NDA developer program |
| Incredibuild | To install Incredibuild select the free trial or upload the license |

On the **Remote Access Configuration** tab complete the following information, then select **Next: VM Network >**:

| Parameters | Value/Description |
|--|--|
| Remote Access Technology | Choose RDP, Teradici or Parasec |
| (If you chose RDP) Azure Virtual Desktop Integration | Click the box to [integrate with Azure Virtual Desktop](./integrate-azure-virtual-desktop.md), then select a Host Pool |
| (If you chose Teradici) Teridici Configuration | Add CAS+ Registration Key |
| (If you chose Parsec) Parsec Configuration | Add the Parsec information required |

On the **VM Network** tab complete the following information, then select **Next: Data Storage >**:

| Parameters | Value/Description |
|--|--|
| Public IP Address | Create or select existing public IP name, if you do not want use a public IP then select **None**  |
| DNS Prefix | Give this VM a unique DNS Prefix or use the defauly prefix for the public IP |
| Virtual Network | Create a new or select an existing Virtual Network space the VM |
| Subnet Name | Choose the subnet name you want to assign to this VM. The Private IP address is the next available IP address in the subnet |

> [!IMPORTANT]
> It is best practice to not expose Virtual Machines directly to the internet with a public IP. [Plan for inbound and outbound internet connectivity](/azure/cloud-adoption-framework/ready/azure-best-practices/plan-for-inbound-and-outbound-internet-connectivity#design-recommendations.md)

On the **Data Storage** tab complete the following information, then select **Next: Management >**:

| Parameters | Value/Description |
|--|--|
| Number of Data Disks | Choose the number of data disks needed for this VM,  |
| Data Disk Size | Select the size of the data disk. Each disk will have the same size |
| Azure Storage File Share Configuration | Click the check box if you need to mount an exisiting azure file share |

> [!NOTE]
>
> - If you don’t want to use the striped disk, please set the number of data disks to be 1. Set the number to 0 if you don’t want to attach a data disk at all. This VM will still have an OS disk and a temporary data disk as the default configuration.
> - It is highly recommended to choose at least 2 disks that will be striped together for the best IOPS performance, and to install your source control and game projects on that drive for the best experience and available space on the VM. The C drive defaults to 256GB and could quickly run out of space for medium to large projects.

On the **Management** tab complete the following information, then select **Next: Advanced >**:

| Parameters | Value/Description |
|--|--|
| Identity | Choose if you want to use System Assigned Managed Identity, learn more about [Managed Identity](/azure/active-directory/managed-identities-azure-resources/how-managed-identities-work-vm.md)|
| Azure AD | Choose if you want to enable AAD for your VM, learn more about [AAD](./integrate-vm-with-aad.md) |
| Guest OS updates | Select Automatic or Manuel updates for your VM |

On the **Advanced** tab complete the following information, then select **Next: Tags >**:

| Parameters | Value/Description |
|--|--|
| Support for custom image creation with sysprep | If you are planning on using this VM to create a custom image, select the check box |

On the **Tags** tab, complete the following information then select **Next: Review + create >**

Tags are used to categorize resources, usually for billing management purposes. If you don't need tags now, you can skip this page and check out [resource tagging](/azure/cloud-adoption-framework/ready/azure-best-practices/resource-tagging.md) to learn more. 

On the **Review + create** tab, ensure validation passes and review the information that will be used during deployment. If the validation has failed, review the previous sections.

Select **Create**.

Once the VM is deployed, RDP directly to the VM and install your engine of choice.

> [!NOTE]
>  
>- There is no additional charge for the software that comes pre-loaded on the virtual machine. You do pay the compute cost for the VM size that you chose in the  **Size**  step. If you choose Teradici or Parsec as your remote access tool, you bring your own license to the VM on Azure.
>- Provisioning takes 10 to 20 minutes. You can view the status of your VM on the Azure portal.

## Access the Game Development VM

After the VM is created and provisioned, there are three methods to access this VM, depending on the **Remote access technology** you chose before.

**Method 1:** RDP. This remote method is always available.

1. Follow the steps listed to [connect and sign on to Azure-based virtual machine](/azure/virtual-machines/windows/connect-logon.md). Use the credentials that you configured when created this virtual machine before. If you enable AAD for this VM, you can also use your corporate credentials for RDP access [if you meet the requirements](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#requirements.md).  

1. You're ready to start using the tools that are installed and configured on the VM. Many of the tools can be accessed through Start menu tiles and desktop icons.

> [!NOTE]
> You may see a command prompt window pop up which shows the Microsoft GDK or other components are being installed in the background. This may take up to 10 minutes. You can safely ignore it but leave the window open, as it will automatically close once all the tasks are finished.

Alternatively, you can follow the steps to remote into the Game Development Virtual Machine with either [Teradici](./remote-to-vm-with-teradici.md) or [Parsec](./remote-to-vm-with-parsec.md) depending on your chosen method of remote access technology.

## Install the Game Engine of your choice

After you access the Game Dev VM, you can install your preferred game engine on this machine. The following are the instructions from different game engine pages that you can start with. Each game engine has rich learning resources and supportive communities that will help you grow your game development skills.

- CryENGINE: [CRYENGINE Setup and Installation](https://www.cryengine.com/tutorials/view/sandbox-and-setup/setup-and-installation)
- GameMaker: [Getting Started with GameMaker](https://gamemaker.io/en/tutorials)
- Godot: [Introduction to Godot](https://docs.godotengine.org/en/stable/getting_started/introduction/index.html)
- O3DE: [Installing O3DE for Windows](https://www.o3de.org/docs/welcome-guide/setup/installing-windows/)
- Unity: [Install the Unity Hub and Editor](https://learn.unity.com/tutorial/install-the-unity-hub-and-editor)

## Clean up resources

When no longer needed, you can delete the resource group, virtual machine, and all related resources.

1. On the Overview page for the VM, select the Resource group link.
2. At the top of the page for the resource group, select Delete resource group.
3. A page will open warning you that you are about to delete resources. Type the name of the resource group and select Delete to finish deleting the resources and the resource group.

## Next steps

- Explore the tools on the Game Dev VM by opening the **Start** menu.
- Learn about customizing a [Game Dev VM](./customized-game-dev-vm.md)
- Learn about game development on Azure by reading [Azure for Gaming](/gaming/azure/) and trying out tutorials.
- Read more about the [Game Development Virtual Machine](./overview.md).
