---
title: Choosing the right GPU SKU and optimizing costs
description: Determine which GPU SKU is right for you.
author: meaghanlewis
ms.topic: overview
ms.date: 03/11/2022
ms.author: mosagie
ms.prod: azure-gaming
---

# Choosing the right GPU SKU and optimizing costs

> [!IMPORTANT]
> [!INCLUDE [reminder](includes/game-dev-banner.md)]

With the Game Development Virtual Machine, game developers have a workstation in the cloud, leveraging the most powerful GPU machines to meet the high-performance requirements of game development. In this document we discuss how to determine which GPU SKU is right for you. Currently the Azure Game Development Virtual Machine supports four families of [GPU – accelerated compute](/azure/virtual-machines/sizes-gpu) SKUs, which are the [NV-series](/azure/virtual-machines/nv-series),  [NVv3-series](/azure/virtual-machines/nvv3-series), [NCasT4_v3-series](/azure/virtual-machines/nct4-v3-series) and [NVadsA10 v5-series](/azure/virtual-machines/nva10v5-series). Because they support [NVIDIA GRID drivers](/azure/virtual-machines/windows/n-series-driver-setup#nvidia-grid-drivers) that are used for 3D applications and content with virtual desktop infrastructure (VDI) solution in the cloud. Please refer to the respective documentation for detailed specifications of each VM series. The Azure VM family is growing, with more powerful GPU virtual machine SKUs on the Azure roadmap. As new GPU SKUs are released, the Game Development Virtual Machine will support them once they’re available.

## GPU Quota on Azure

Before creating Game Development Virtual Machines (VMs) that use high demand GPUs, check that you have the requested quota for the desired GPU SKU. Otherwise, you'll receive a quota error when deploying the VM.

A quick way to validate if you have quota in a specific region is to use the Azure command-line interface (Azure CLI) to run the following commands.

1. See [Quickstart for Bash in Azure Cloud Shell](/azure/cloud-shell/quickstart) for instructions to start the Cloud Shell and select the Bash environment.
1. Use **az account** to change the context of the bash session, if needed.
1. Run the following [az vm list-usage](/azure/virtual-machines/linux/cli-manage) command to list the GPU quota in the US West region.

```azurecli-interactive
az account show
az vm list-usage -l westus -o table | grep "Name\b\|---\|\(NV\|NVSv3\|NCASv3_T4\|A10v5\)\b" | (sed -u 2q; sort)
```

**TIP:**
If you are an independent game developer or startup interested in working with the Game Development VM, we strongly recommend that you join the [ID@Azure](https://www.azure.com/id) program. This program provides free access to developer tools and support from industry experts. If you have questions, see [Troubleshooting and Support](/azure/game-dev-virtual-machine/troubleshoot-support) or reach out to us on Game Dev VM [Discord Channel](https://aka.ms/gamedevVMdiscord).

## Determining the right GPU SKU to use

When trying to decide the right GPU SKU to use when deploying the Game Development Virtual Machine, there are a few factors that you’ll want to consider when ensuring the chosen GPU SKU meets your requirements. These are performance, capabilities (i.e., ray tracing requirements), region availability/capacity and price requirements. Let’s review each of these factors to help inform your decision on the right GPU SKU for your specific needs.

## Performance

Currently the most performant GPU SKUs in Azure for a game development workstation are the NCasT4_v3-series, especially if you need real-time ray tracing. These SKUs are powered by [Nvidia Tesla T4](https://www.nvidia.com/en-us/data-center/tesla-t4/) and uses NVIDIA's GRID driver and virtual GPU technology. The NVv3-series is a good second choice if real-time ray tracing is not required. Both the NCasT4_v3-series and the NVv3-series support [Premium Storage](/azure/virtual-machines/premium-storage-performance), which is highly recommend for game development and production workloads due to the low latency, faster IOPS and disk reliability. The newer VM SKUs that support premium storage can also be faster to spin up versus the NV-series VMs and have more consistency on the spin up times.

**Important**: Be sure to select at least 2 1TB+ SSD disks to be striped together (linearly increases IOPS/throughput) when deploying the Game Dev VM, and if you deploy through the Azure Portal, you can use the slider under the **Data Storage** tab to automatically calculate the expected IOPS/throughput for that striped disk. Additionally, ensure that you install your version control game assets on the striped disk for the best game engine editor and build performance, as well as avoiding running out of space on the smaller OS disk. As each VM SKU instance has a max IOPS and throughput for that VM, you might want to upgrade to a VM with higher cores to reach a higher IOPS/throughput limit (e.g., NV24s_v3 versus NV12s_v3).

If you decide to upgrade your current VM to a more powerful SKU, the VM can be re-sized after the initial creation (e.g., NV12s_v3 to NV24s_v3). Learn more about how to [change the size of a virtual machine](/azure/virtual-machines/resize-vm?tabs=portal).

> [!NOTE]
> Azure Game Dev VM started to support a more powerful GPU SKU [NVadsA10 v5-series](/azure/virtual-machines/nva10v5-series) since July, 2022. It is powered by [NVIDIA A10](https://www.nvidia.com/en-us/data-center/products/a10-gpu/) GPU, which has better performance than T4. For more details about this VM size, please refer to the [generally available blog post](https://azure.microsoft.com/blog/choose-the-right-size-for-your-workload-with-nvads-a10-v5-virtual-machines-now-generally-available/).

## Capabilities

For developers requiring ray tracing on their workstations, please leverage our NCasT4_v3-series or NVadsA10 v5-series VMs, which has [RT Cores](https://developer.nvidia.com/rtx/ray-tracing) that use the NVIDIA RTX™ technology you are likely familiar with in the consumer gaming cards. More powerful GPUs will continue to be released in Azure, which will further close the performance and capabilities gaps when coming from a local developer workstation.

## Region Availability

There are some regions that do not have the VM SKUs currently supported by the Game Development Virtual Machine, and you should refer to the [VM SKUs pricing page](https://azure.microsoft.com/pricing/details/virtual-machines/windows/) to verify which GPU SKUs are available in your desired region. You may also choose to use alternative VM SKUs where availability/capacity aren’t available for the more powerful SKUs, as GPU SKUs are in high demand, and depending on the region they can be more scarce at times.

## Pricing

Though your decision to choose a particular GPU SKU might be more around capability and performance versus price, it’s good to review the differences in pricing between the different GPU SKU families, especially as you increase in core count within that family. To review the pricing differences, please refer to the [Azure Pricing Calculator](https://azure.microsoft.com/pricing/calculator/).  If your requirements would allow for using Azure Spot Virtual Machines, major discounts can be had. Some VM sizes in certain regions have up to 90% discount. Please check [Azure Spot Virtual Machines pricing](https://azure.microsoft.com/pricing/spot-advisor/) page for region availability and discount rates.  

## Ensure you have GPU quota

Before deploying the Game Development Virtual Machine, please check if your Azure subscription has enough quota to deploy GPU-intensive VMs. In the Azure Portal under the **Subscriptions > <_SUBSCRIPTION_> -> Usage + Quotas** blade, select the Location(s) you want to ensure you have quota, and just choose _“Microsoft.Compute”_ as the Provider. This will display a list of CPU families, where you can either scroll down to find the desired family supported by the Game Dev VM, or quickly filter the list by putting one of the following supported CPU families in the Search box: Standard NV Family vCPUs, Standard NVSv3 Family vCPUs, Standard NCASv3_T4 Family vCPUs and Standard NVADSA10v5 Family vCPUs. When deploying the VM, if you see a message that a usage limitation has been reached, or an exclamation mark saying “VM size currently unavailable”, you can follow the steps above or create a support request to increase your quota.  

## NV Series Deprecated on August 31st, 2023

Currently the Game Development Virtual Machine supports the NV series of GPUs but be aware that these GPUs will be deprecated on August 31st, 2023. Learn more about the latest updates, expectations and migration steps on the [NV-series page](/azure/virtual-machines/nv-series). As the NV series is an older generation, it’s recommended that you use the newer NVv3-series VMs where possible, unless there was a capacity limitation in your region due to a large demand for the NVv3-series.  

## GPU Drivers are already pre-installed

The GPU and NVIDIA GRID drivers are pre-installed on the Azure Game Dev VM, so you don’t need to install them or add the NVIDIA Azure extension when deploying the VM. This helps to reduce initial spin up times and removes the chances for needed restarts or potential install errors.

## Next steps

[Create a Game Development Virtual Machine with Unreal Engine.](./create-game-development-vm-for-unreal.md)
