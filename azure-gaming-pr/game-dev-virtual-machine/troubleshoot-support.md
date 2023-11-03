---
title: Troubleshoot Azure Game Development Virtual Machine and Get Support
description: View common support issues that customers might experience using the Azure Game Development Virtual Machine and steps to resolve those issues.
author: meaghanlewis
ms.topic: troubleshooting
ms.date: 04/05/2022
ms.author: mosagie
ms.prod: azure-gaming
---

# Troubleshoot Azure Game Development Virtual Machine and Get Support

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

The Game Development Virtual Machine is built on top of common Windows images currently available in Azure, which allows you to use much of the same troubleshooting guidance found for general Azure VMs. For more details about how to get support with your Azure VMs, please visit the [Azure support FAQ](https://azure.microsoft.com/support/faq/#support-overview).

We are very eager to hear your feedback and invite you to share your experiences and feedback in our <a href="https://forms.office.com/r/VHK5iEqeBm" target="_blank">survey</a>; however, be sure to [create a ticket](https://azure.microsoft.com/support/create-ticket/) in the Azure Portal if you run into issues that require support to help unblock you. Additionally, join our <a href="https://aka.ms/gamedevVMdiscord" target="_blank">Game Dev Discord channel</a> to connect with thousands of game developers around the world and talk with the Game Development Virtual Machine team.

This article describes common support issues that customers might experience, and guidance from us about how to resolve them.

## VM deployment support

### Reached quota limits

> [!TIP] 
> If you are an independent game developer or startup interested in working with the Game Dev VM, joining the <a href="https://www.azure.com/id" target="_blank">ID@Azure</a> program is strongly recommended. This program will help with your quota ask; plus you get free access to developer tools and support from industry experts.

The most common issue when deploying the Game Development Virtual Machine is when the Azure region you are deploying to does not have the desired quota, especially for GPU SKUs which are high in demand. You may receive a similar error like the following when you deploy the VM: _"Operation could not be completed as it results in exceeding approved standardNVSv3Family Cores quota."_

If you can’t find your desired VM SKUs when you attempt to create the VM, it could be possible that your Azure subscription doesn’t have enough quota to deploy GPU-intensive VMs, or the region doesn’t support the SKUs, or your Azure offer doesn't support GPU. To check what Azure offers support GPU SKUs, please refer to this <a href="./offer-types.md" target="_blank">offer types chart</a>.
	
:::image type="content" source="./media/troubleshoot-support/select-vm-size.png" alt-text="Screenshot showing the Azure Portal dashboard to select a VM size":::

To check if your region supports the required SKUs, please go to [Azure Products by Region](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines) and examine if **NCasT4v3-series**, **NV-series**, **NVv3-series** or **NVadsA10 v5-series** is available. You can also [check resource usage against limits](/azure/networking/check-usage-against-limits) if you believe you need more quota. Select the location(s) you want to deploy the VM. And choose **Microsoft.Compute** as the Provider. Confirm that the VM sizes: **Standard NV Family vCPUs**, **Standard NVSv3 Family vCPUs**, **Standard NCASv3_T4 Family vCPUs** and **Standard NVADSA10v5 Family vCPUs** are available since these are the sizes supported by the Game Development VM.

:::image type="content" source="./media/troubleshoot-support/request-increase.png" alt-text="Screenshot showing the available VM sizes and how to request quota increases":::

If you see the Usage is full, or an error message saying _"VM size currently unavailable"_, you need click the icon to create a support request to increase quota or activate the unavailable VM size(s).

:::image type="content" source="./media/troubleshoot-support/request-increase-icon.png" alt-text="Screenshot showing how to request a quota increase":::

In some cases, if you see a pencil icon, then you click it and set up a [new quota limit](/azure/azure-portal/supportability/per-vm-quota-requests) by yourself.

:::image type="content" source="./media/troubleshoot-support/edit-icon.png" alt-text="Screenshot showing how to set up a quota limit":::

> [!TIP]
> If you prefer command line, you can run <a href="/azure/cloud-shell/quickstart" target="_blank">Bash in Azure Cloud Shell</a> in the Azure portal. <a href="/cli/azure/vm?view=azure-cli-latest#az-vm-list-skus&preserve-view=true" target="_blank">az vm list-skus</a>, and <a href="/cli/azure/vm?view=azure-cli-latest#az-vm-list-usage&preserve-view=true" target="_blank">az vm list-usage</a> are useful commands to start with.

**Example 1:** Find out which regions have supported VM sizes for Game Dev VM, and whether those sizes are available for your Azure subscription.
```azurecli-interactive
az vm list-skus --all --output table | grep "Locations\b\|---\|\([[:alnum:]]\+[[:space:]]\+\)Standard_N\(V[[:digit:]]\+\(s_v3\)\?\|C[[:digit:]]\+as_T4_v3\)\b" | (sed -u 2q; sort)
```
**Example 2:** Get the compute resource usage, which supports Game Dev VM for the West US region in your Azure subscription.
```azurecli-interactive
az vm list-usage --location westus -o table | grep "Name\b\|---\|\(NV\|NVSv3\|NCASv3_T4\|A10v5\)\b" | (sed -u 2q; sort)
```

### Validation failure

At the end of Game Development VM creation, the Azure Portal will validate whether all the required information has been filled in. If any information is missing or incorrect, you will see the error message: _"Validation failed. Required information is missing or not valid."_

:::image type="content" source="./media/troubleshoot-support/validation-failed-error.png" alt-text="Screenshot showing the validation error during VM creation":::

The Azure Portal highlights which tabs are missing information and what needs to be changed, by putting a red circle above the tab that requires attention. You need to go back to the tabs and complete the missing fields or make the correction.

### Deployment failure

It usually takes about 10 to 15 minutes to deploy the VM. If deployment fails, Azure will return the error message in the Azure Portal notifications bar, which will display the reason for the failure, so you know how to resolve the issue. For more common VM deployment failure issues, refer to the [troubleshooting VM deployments in Azure article](/troubleshoot/azure/virtual-machines/troubleshoot-deployment-new-vm-windows).

:::image type="content" source="./media/troubleshoot-support/deployment-failed-details.png" alt-text="Screenshot showing how to see why VM deployment failed":::

### Teradici or Parsec license failure

If you choose Parsec or Teradici as the remoting tool into the VM, and the license information is incorrect or can’t be verified, you may notice the following failure with an error message at the end of deployment, when clicking on the **Operation details** link: _"VM has reported a failure when processing extension 'CustomScriptExtension-Teradici'. Error message: \"Command execution finished, but failed because it returned a non-zero exit code of: '1'. The command had an error output of: 'No license available.\r\nFailed to register with the cloud license server \"https://teradici.compliance.flexnetoperations.com/instances/.../request\"_.

An Azure Custom Script Extension is used to verify your license information with each partner. If the information is wrong or the verification can’t finish because of a network connection issue, the above failure occurs. You can still access the VM using RDP and manually register your remoting solution with the correct information. You don’t need to restart the whole deployment.

If none of the above scenarios are applicable to your VM Deployment issue, please <a href="https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/%7B%0D%0A%09%22pesId%22%3A+%226f16735c-b0ae-b275-ad3a-03479cfa1396%22%2C%0D%0A%09%22supportTopicId%22%3A+%22e2212b28-7a50-21a0-2346-d238a1010cfb%22%2C%0D%0A%09%22contextInfo%22%3A+%22Deployment+Issues+with+Game+Developer+VM%22%2C%0D%0A%09%22severity%22%3A+%224%22%0D%0A%7D" target="_blank">open a support request</a>. The request should have the Service type: Virtual Machine running Windows, Problem type: Cannot create a VM, and Problem subtype: Troubleshoot marketplace image deployment failures.

:::image type="content" source="./media/troubleshoot-support/cannot-create-vm.png" alt-text="Screenshot showing what to include in support request when VM creation fails":::

## VM access support

### RDP

There are three methods to access this VM. RDP, Parsec or Teradici. RDP is the default option, and it is always available. If you have any VM Access issue, the first step is to [troubleshoot the default RDP option](/troubleshoot/azure/virtual-machines/troubleshoot-rdp-connection). If you need other support with RDP, please <a href="https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/%7B%0D%0A%09%22pesId%22%3A+%226f16735c-b0ae-b275-ad3a-03479cfa1396%22%2C%0D%0A%09%22supportTopicId%22%3A+%2292c2396d-b703-973f-1bca-2eea9425b21a%22%2C%0D%0A%09%22contextInfo%22%3A+%22RDP+or+SSH+connect+Failures+Issues+with+Game+Developer+VM%22%2C%0D%0A%09%22severity%22%3A+%224%22%0D%0A%7D" target="_blank">open a support request</a>. The request should have the Service type: Virtual Machine running Windows, Problem type: Cannot connect to my VM, and Problem subtype: Failure to connect using RDP or SSH port.

:::image type="content" source="./media/troubleshoot-support/cannot-connect-vm.png" alt-text="Screenshot showing what to include in support request when VM connection fails with RDP":::

### Parsec

The pre-installed Parsec component in this Game Development VM is the [Parsec application](https://parsec.app/downloads). No additional network port configuration or NSG is required to make Parsec work. If you can access the VM via RDP but not Parsec, please reach out to [Parsec support](https://support.parsec.app/hc/)
Here are some additional Parsec technical resources:

- [Remote to Game Development Virtual Machine with Parsec](./remote-to-vm-with-parsec.md)
- [Parsec for Teams Onboarding Guide](https://pages.parsec.app/hubfs/AWS%20AMI%20marketplace/Parsec%20for%20Teams%20Onboarding%20Guide.pdf)
- [Getting Started with Parsec For Teams](https://support.parsec.app/hc/articles/360040562872-Getting-Started-With-Parsec-For-Teams)

### Teradici

The pre-installed Teradici component in the Game Development VM is the [Teradici PCoIP Graphics Agent for Windows 21.07](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/21.07/). If you need, first follow the [instructions to remote into the VM with Teradici](./remote-to-vm-with-teradici.md). If you can access the VM via RDP but not Teradici, please make sure the ports: 4172, 443 and 60443 are not blocked by a firewall in your network environment.

:::image type="content" source="./media/troubleshoot-support/teradici-rdp.png" alt-text="Screenshot showing Teradici remote connections to a VM":::

Check if there’s any other NSG rules taking precedence. If networking is excluded as the cause of the issue, it’s possible the issue roots from software itself, and you’ll need to contact [Teradici support](https://www.teradici.com/web-help/pcoip_agent/graphics_agent/windows/21.07/admin-guide/support/contacting-support/).

### Azure Active Directory

This Game Development VM supports Azure Active Directory (Azure AD) authentication when you enable Azure AD during VM creation. The user experience of Azure AD on Game Development VM is identical to regular Azure VM. If you have sign-in issues with Azure AD, please follow the [troubleshooting steps](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#troubleshoot-sign-in-issues).

The most common issue is that Azure role assignment is not configured after VM is created. You need to set up either **Virtual Machine Administrator Login** or **Virtual Machine User Login** role on the Game Development VM to [authorize Azure AD login](/azure/active-directory/devices/howto-vm-sign-in-azure-ad-windows#configure-role-assignments-for-the-vm).

If the troubleshooting methods in the above links can’t resolve your issue, please <a href="https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/%7B%0D%0A%09%22pesId%22%3A+%226f16735c-b0ae-b275-ad3a-03479cfa1396%22%2C%0D%0A%09%22supportTopicId%22%3A+%220969c3f5-eae0-7620-93c2-609254bcac83%22%2C%0D%0A%09%22contextInfo%22%3A+%22AAD+Login+Extension+Issues+with+Game+Developer+VM%22%2C%0D%0A%09%22severity%22%3A+%224%22%0D%0A%7D" target="_blank">open a support request</a>. The request should have the Service type: Virtual Machine running Windows, Problem type: VM Extensions not operating correctly, and Problem subtype: Azure Active Directory Login extension issue.

:::image type="content" source="./media/troubleshoot-support/vm-extensions-not-operating.png" alt-text="Screenshot showing what to include in support request when VM extensions not operating correctly":::

## Issue with pre-installed tools

This VM comes pre-installed with [many different tools](./tools-included-azure-game-dev-kit.md). If you have an issue with a partner’s tooling, please reach out directly to that partner for support on their specific solution. One common issue you may notice with pre-installed tools can be related to network restriction, as some tools use different network ports to communicate with other devices or services. The Azure Game Development VM doesn’t block those ports. However, you may have your own security policies which block certain ports on Azure. You need to check with your network administrator and make sure the network Access Control List (ACL) or other security mechanisms like Blueprint policies in your environment don’t block the required network connection.

To know which tools were successfully installed on your VM, you can find a log file named **INSTALLED SOFTWARE.txt** on the desktop after VM is deployed. This file lists all the pre-installed software and the installation status. This can be a valuable resource to troubleshoot pre-installed tooling related issues.
If the issue you find relates to the pre-install tools, you should reach the respective support channels or leverage community resources, like our [Discord channel](https://aka.ms/gamedevVMdiscord), [Microsoft Q&A](https://techcommunity.microsoft.com/), or [Stack Overflow](https://stackoverflow.com/search?q=gdvm) to get help. However, if you believe this is a Game Development VM related issue, please <a href="https://ms.portal.azure.com/#create/Microsoft.Support/Parameters/%7B%0D%0A%09%22pesId%22%3A+%22e00b1ed8-fc24-fef4-6f4c-36d963708ae1%22%2C%0D%0A%09%22supportTopicId%22%3A+%229cfdee6c-9044-62fa-6a3f-5f3dbded7124%22%2C%0D%0A%09%22contextInfo%22%3A+%22Virtual+Maxhine+Issues+%28N+or+H+Series%29+On+Game+Developer+VM%22%2C%0D%0A%09%22severity%22%3A+%224%22%0D%0A%7D" target="_blank">open a support request</a> with the Service type: High Performance Computing (HPC), Problem type: HPC VM (N or H series), and the most relevant problem subtype.

:::image type="content" source="./media/troubleshoot-support/hpc-issue.png" alt-text="Screenshot showing what to include in support request when there is an HPC issue":::
