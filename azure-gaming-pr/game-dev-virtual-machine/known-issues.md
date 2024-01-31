---
title: Known issues and FAQs with the Azure Game Development Virtual Machine
description: Learn about known issues and FAQ when you work with the Azure Game Development Virtual Machine.
author: meaghanlewis
ms.topic: troubleshooting
ms.date: 04/21/2022
ms.author: mosagie
ms.service: azure-gaming
---

# Known issues and FAQs with the Azure Game Development Virtual Machine

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

This article discusses known issues to be aware of when you work with Azure Game Development VM. If you have suggestions about this product, please submit your ideas via this <a href="https://forms.office.com/r/VHK5iEqeBm" target="_blank">survey</a>. Or join this <a href="https://aka.ms/gamedevVMdiscord" target="_blank">Game Dev VM Discord channel</a> to connect with thousands of game developers around the world. Microsoft monitors the user feedback closely and listens to the voice of customers so that we can improve the service.

> [!NOTE]
> This article isn't a comprehensive list of known issues. If you know of an issue that isn't listed, provide feedback at the bottom of the page.

## Deploying the VM

### No VM Size available

You may receive the following error when you deploy the VM: "Operation could not be completed as it results in exceeding approved size Family Cores quota." Or you cannot find the VM Size when you attempt to create this Game Development VM.

This is true when your subscription reaches the VM usage limits, especially the [GPU – accelerated compute](/azure/virtual-machines/sizes-gpu) SKUs which are not always enabled by default. You can try different regions, or [check resource usage against limits](/azure/networking/check-usage-against-limits) and increase the limit. It is also possible that your Azure offer doesn't support GPU. Please refer to this <a href="./offer-types.md" target="_blank">offer types chart</a> to check if your Azure subscription falls under the supported offers.

> [!TIP] 
> If you are an independent game developer or startup interested in working with the Game Dev VM, joining the <a href="https://www.azure.com/id" target="_blank">ID@Azure</a> program is strongly recommended. This program will help with your quota ask; plus you get free access to developer tools and support from industry experts.

## Accessing the VM

### Slow first-time sign-in

You may see the following “Preparing Windows” screen stay for about 1 to 2 mins after your first sign-in. This is expected.  

:::image type="content" source="./media/known-issues/preparing-windows.png" alt-text="Screenshot showing Preparing Windows message during sign in":::

### Teradici "Connect Insecurely" message

As Teradici uses self-signed certificates as the default, you may notice a window pop up that warns you that the certificate cannot be verified due to the certificate not being registered with a trusted certificate authority.

:::image type="content" source="./media/remote-to-vm-with-teradici/teradici-connect-insecurely.png" alt-text="Screenshot showing the window to connect to Teradici insecurely":::

Teradici uses HTTPs encryption with a self-signed private key. You can click Connect Insecurely to login to the VM. Learn more here about [configuring trusted signed certificates with Teradici](https://www.teradici.com/web-help/pcoip_connection_manager_security_gateway/19.08/security/creating_cmsg_cert/#default-certificate), which is recommended when deploying a production system.

### Cannot sign in EPIC account with Google  

You will need to verify Unreal Engine End-user license agreement (EULA) on the very first connection to the Game Development VM before using it. This requires you to log in with your EPIC Games account. If you use  Google, you may notice an error that it couldn’t sign you in.

:::image type="content" source="./media/known-issues/couldnt-sign-you-in.png" alt-text="Screenshot showing EPIC Games sign in error with Google":::

This is a known issue that the engineering team is working towards a fix. The current workaround is to use other mechanisms to log in with your EPIC Games account. You only need verify EULA once for each Game Development VM. And you can still use Google to sign in EPIC Games Launcher to run Unreal Engine inside this VM without problem.

## Using the VM

### Can’t run Visual Studio after first-time sign-in

You may notice that you can’t open Visual Studio or other products immediately after first-time sign-in the Game Development VM. This is expected because GDK is still installing. You will need wait for a few minutes until GDK installation window disappears. Then you can use Visual Studio or other developer tools.

:::image type="content" source="./media/create-game-development-vm-for-unreal/user-configuration-tasks-messages-terminal.png" alt-text="Screenshot showing Microsoft GDK installing":::

### Can't use Unreal Engine or other 3D applications

You may see certain error message when you start 3D application on the Game Dev VM. For example, with Unreal Engine, you see message like *"A D3D11-compatible GPU (Feature Level 11.0, Shader Model 5.0) is required to run the engine."*

This is possibly because you intended to use this VM as a build server when you first deployed it. Therefore, the VM size you chose before didn't fall into [NV-series](/azure/virtual-machines/nv-series),  [NVv3-series](/azure/virtual-machines/nvv3-series) or [NCasT4_v3-series](/azure/virtual-machines/nct4-v3-series). Only these VM sizes support 3D applications and 3D content editing. Please check your VM size and redeploy with correct one which has GPU. You can still keep this VM for your build server as you originally planned.  

### Prompted warning about issues with graphic driver 

You may see a window pop up with message starts with *"The installed version of the NVIDIA graphics driver has known issues in D3D12."* 
This only happens when you run Unreal 5.1 engine which is still in preview on Game Dev VM. You can safely ignore that window and click "No" button to not install the recommended driver. 

### Slow performance when opening a project with Unreal Engine during compiling shaders

You may experience a long wait to open an Unreal Engine project and see the slow progress of compiling shaders.

:::image type="content" source="./media/known-issues/compiling-shaders.png" alt-text="Screenshot showing Unreal Editor is compiling shaders while loading project":::

This can happen if you have Incredibuild installed without an active license or a limited license for only a few CPU cores. In such a scenario, Unreal Engine doesn’t utilize all the CPU cores that are assigned to this VM. Instead, only one core or just a few cores out of many are being used. Therefore, you notice the slowness when using Unreal Engine.  

The solution is to have an active and [complete Incredibuild license](https://www.incredibuild.com/pricing). Or uninstall Incredibuild to remove the CPU core limitation when you use Unreal Engine on this VM.

## Other FAQs

#### How much does the Azure Game Development VM cost?

There is no additional charge for the Azure Game Development VM from what you would pay if you were spinning up a Windows VM in Azure, as all the software is either free or bring your own license (i.e., Parsec, Teradici and Incredibuild). There are configurations like adding striped SSD disk volumes and Azure Files for fast shared SMB caches, which would increase the cost of using the VM. The cost is also determined by the VM SKU you choose, which would increase as you choose a larger SKU which has more cores, memory, temp SSD storage and throughput. Please see the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/) to estimate costs.

#### I see prompts from certain pre-installed software (i.e., EPIC launcher, Unreal Engine, etc) which reminds me some updates are available. Why, and what should I do?

While Microsoft updates the Game Development VM image periodically, it can’t guarantee all software inside this VM is always up to date. End users can decide whether to install the software updates on their own. They have full control of the operating system and are responsible for maintaining and updating the software environment. See all the pre-installed software and [Tools included with the Azure Game Development Virtual Machine](./tools-included-azure-game-dev-kit.md).

#### How to update GPU drivers on the Game Dev VM?
The Game Dev VM with VM sizes in NV-series, NVv3-series and NCas_T4_v3 series comes with NVIDIA GRID drivers. These drivers are used specifically for virtual desktop infrastructure (VDI) solutions in cloud & data centers. Therefore, you need to update NVIDIA GRID drivers, instead of the regular Geforce drivers to avoid any unexpected issues. You can find the latest supported NVIDIA GRID drivers for Azure VM running windows on the website: <a href="/azure/virtual-machines/windows/n-series-driver-setup#nvidia-grid-drivers" target="_blank"> Azure N-series NVIDIA GPU driver setup for Windows - Azure Virtual Machines </a> .

#### How does support work?  

Microsoft supports its own included tooling and Azure infrastructure used to deploy the Game Dev VM. For any partner tooling, support will come directly from those partner support channels, which is specified when signing up for those products. If you have an issue when deploying the Game Dev VM, please try and look at the INSTALLED_SOFTWARE.txt file on the desktop which gives a log of what was installed, and you can always open a [support ticket](/azure/azure-portal/supportability/how-to-create-azure-support-request) with Azure.
> [!TIP] 
> If you are an independent game developer interested in working with the Game Dev VM, join the <a href="https://www.azure.com/id" target="_blank">ID@Azure</a> program to get free access to developer tools and support from industry experts. 

#### I’m using Windows 10 and can’t verify if this VM is utilizing the licensing benefits, is that a problem?  

As long as you have the [eligible windows 10 subscription licenses](/azure/virtual-machines/windows/windows-desktop-multitenant-hosting-deployment#subscription-licenses-that-qualify-for-multitenant-hosting-rights) that qualify for Multitenant Hosting Rights, or you deploy Windows 10 with your [Visual Studio (formerly MSDN) subscription](/azure/virtual-machines/windows/client-images) on Azure for dev/test purposes, you’re not violating Microsoft’s licensing restrictions.

#### What’re the differences between <a href="https://azure.microsoft.com/products/dev-box/" target="_blank">Microsoft Dev Box</a> and <a href="https://azure.microsoft.com/products/virtual-machines/game-development-virtual-machines/" target="_blank">Azure Game Dev VM</a>?

Microsoft Dev Box is a managed Azure service that enables developers to create on-demand, high-performance, secure, ready-to-code, project-specific workstations in the cloud. It is built on top of <a href="https://www.microsoft.com/windows-365" target="_blank">Windows 365</a>, leveraging it to integrate Dev Boxes with Intune to ensure unified management, security, and compliance stay in the hands of IT. 

Game Dev VM is an Azure VM with customized image which includes common game development tools. It specifically targets game professionals who want to use Azure for game production; and meets the challenges of distributed game development with GPU-powered computing resources. It is deployable via Azure Portal, Powershell, ARM template, CLI, Terraform, or even via Bicep to VMSS. It now supports the <a href="./integrate-azure-virtual-desktop.md" target="_blank">integration with Azure Virtual Desktop</a>. 

The Game Dev VM will be deployable on Dev Box in the future.

