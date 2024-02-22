---
title: Scale out your cloud builds with Incredibuild
description: Learn how to set up Incredibuild in your build pipeline
author: whenslunch
keywords: 
ms.topic: reference-architecture
ms.date: 3/14/2022
ms.author: tzong
ms.service: azure-gaming
---

# Set up Incredibuild on Azure DevOps build agents

This guide will show how to set up Incredibuild on an Azure DevOps Windows build agent as part of an Azure-powered build pipeline from the related GDC 2022 talk . This document is not intended as a substitute for Incredibuild's documentation, rather, it aims to give the context of Incredibuild setup within an Azure DevOps build pipeline. We will refer to the Incredibuild documentation where appropriate.

This document also assumes the reader has some fundamental knowledge of:

- working in the Azure Portal, with the ability to create VMs and simple networking
- working in Azure DevOps, with the ability to set up build agents (not to be confused with Incredibuild agents)

[![Incredibuild build server scalling on Azure](media/cloud-build-pipeline/incredibuild-architecture.png)](media/cloud-build-pipeline/incredibuild-architecture.png)

## What is Incredibuild?

Incredibuild is a suite of compilation acceleration software that takes advantage of idle compute resources within an organization to spread compute load. You can find out more about the company, its products and licensing at **[https://www.incredibuild.com/](https://www.incredibuild.com/)**

Incredibuild has been used for more than two decades and is a popular choice in game development to speed up build workflows in game studios.

## Where can I use it?

### On your development desktop

Incredibuild is commonly used on the desktop, integrated with IDEs like Visual Studio or command-line tools like make/cmake. Once installed and configured correctly, Incredibuild will accelerate compilation behind the scenes.

### In your CI pipeline

The same is true for CI/CD build pipelines. Once installed correctly, Incredibuild can accelerate game builds and will automatically scale up and down compute resources during the compile phases of your build.

## Quick overview of Incredibuild architecture

This section is a summary of the **[full Incredibuild documentation](https://docs.incredibuild.com/win/latest/windows/components_and_architecture.html)**, and points out the bare basics needed to get started. Incredibuild basically has two components: Agents and Coordinators.

### Agents

Agents are the machine instances that run compile workloads. The initiator agent is the first thing to get activated to assess the workload. Then helper agents assist the initiators. Together, they are the computing resources that receive packages of work to execute, then send back the results.

Agents can be on-premises machines, VMs in the cloud, or a mix of both. Once an Incredibuild software agent is installed on a computer and configured to point to a Coordinator, it then gets registered as an Agent.

### Coordinator

Coordinators, as the name implies, coordinate the sending and receiving of work processes between all agents. They do this after receiving a request from the initiator Agent that tells it how many cores it needs. Among other tasks, Coordinators also track availability, state, quality of service and package update status of all agents.  

The coordinator software is simply installed on the machine designated for that task.
There is usually one Coordinator to handle multiple agents. Of course, you can also set up redundant Coordinators for fault tolerance.  
Coordinators require a static IP address or a configured DNS name for all agents to locate it.

## Prerequisites and considerations

There are a few considerations for how to set up your Incredibuild system. What machine will you use for the Coordinator and Initiator Agent?  What machines will you use for your helper Agents?  If you are using Incredibuild Cloud agents, what kind of VMs will you need, and how many? Do you already have an Azure Subscription you can use?

Much will depend on your current setup and your budgets. Here are some suggestions for you to consider.

### Complete Azure setup

In this case, we assume:

- Coordinators and Agents will all be VMs in Azure
- The Azure DevOps pipeline will use a custom build agent that will act as the Coordinator and Initiator Agent

Also consider where your Helper Agents will come from. During the Incredibuild Cloud setup, you will be asked to specify what type(s) of VMs you would like to use, and the maximum number of VMs to scale.

You also have the option of using Spot VM instances as agents. Spot instances allow you to take advantage of Azure's unused capacity at a significant cost savings. The trade-off is that when Azure needs the capacity back, the Azure infrastructure will evict these Spot Virtual Machines. However, because of the way Incredibuild divides the compute tasks, Azure Spot VM remain a viable option for a build pipeline. For more information on Azure Spot VMs, please see the **[documentation](/azure/virtual-machines/spot-vms)**.

You should also check your Azure core quota. You cannot allocate more VMs of a certain type than your Subscriptions' quotas allow, so make sure that you confirm you can allocate as many as you need. You can **[request more quota](/azure/azure-portal/supportability/per-vm-quota-requests)** of a certain VM SKU if needed.

### Hybrid setup

You may already have an on-premises compute farm that you would like to use as part of the machine pool. Incredibuild deals with both on-premises machines and cloud machines, so you would just have to configure them accordingly.

One consideration is to use your on-premises resources as the mainstay of your build pipeline, and then use Azure VMs to burst when needed.

Don't forget to check your Azure core quota as well. If you don't have enough quota of a certain SKU of interest, in certain Azure regions, you should apply ahead of time for a quota increase as it could take some time.

### Licensing

You will need a working Incredibuild license. You can obtain a **[free trial license](https://www.incredibuild.com/free-trial)** from Incredibuild. This will allow you to run a limited number of agent cores to get started. For a full license, please contact Incredibuild.

### Azure Subscription

Assuming you will use Azure VMs to scale computation, you will need an Azure Subscription with a suitable RBAC role for yourself, such that you can create an **[App Registration](/azure/active-directory/develop/app-objects-and-service-principals)**. This will be needed in the Incredibuild Cloud setup phase.

### Full Incredibuild requirements

Incredibuild specifies requirements for open ports, networking, storage, operating system and more, in their **[documentation here](https://docs.incredibuild.com/win/latest/windows/system_requirements.html)**.

## Setting up Incredibuild

The Azure Marketplace offers a standalone Incredibuild Cloud offering as well as the new [Azure Game Developer VM](/gaming/azure/game-dev-virtual-machine). These have Incredibuild already installed, so we’ll go through the configuration steps to get it running.

You might also have an existing machine on which you want to install Incredibuild directly. The only difference is here is you must first obtain the Incredibuild installer and install the software, and the rest of the steps will be the same. Please contact Incredibuild to get their installer package.

### Starting with the Azure Marketplace Incredibuild VM

#### 1. Choose your deployment

Here, you can choose from a range of suggested VM types to deploy your Incredibuild main machine (the one containing the initiator Agent and a Coordinator), including some pre-set options. If you're not sure what to choose, try a pre-set option of a General Purpose D-series default VM, that should give you a good balance of compute, networking and storage performance to begin with.  

#### 2. Create the Incredibuild VM

After setting up your VM with the appropriate data like Subscription, Resource Group, VM name, disks and so on, click on Create to create the VM.

#### 3. RDP into the system and continue setup

Once done, open a Remote Desktop Session to the VM to continue setup of the system. The first thing you will want to do is install your Incredibuild license; In the Coordinator window, click on Tools -> Coordinator settings -> License. In the Settings window, install your license file.

#### 4. Configure according to Incredibuild instructions

Read about Incredibuild Cloud [here](https://docs.incredibuild.com/cloud/cloud_understanding.html), and start setting up [here](https://docs.incredibuild.com/cloud/cloud_setup_standard.html).

You will need access to an Azure Subscription, Azure Active Directory to perform App Registration, and necessary VM quotas. All these are explained in the Incredibuild documentation.

## Testing

Once you set everything up, start up the Coordinator Monitor. You should see all the agent machines, both on-prem and in the cloud, appear in the Coordinator Monitor window.
If you are using Azure Spot VMs, please note that the VMs will not appear in the Coordinator Monitor until they have been requisitioned, allocated and put into active service. This is due the on-demand nature of this particular VM offering.

You can do a simple verification that your setup works by starting up a compilation job on the Initiator agent or coordinator (often the same machine). You could load up a Visual Studio project that you own for this purpose, but you want to make sure the job is a large one, otherwise it will simply finish on the Initiator agent without Incredibuild needing to spread out the work. The Build Monitor will show the cores in action during compilation stages.

## Conclusion

Incredibuild setup can be a bit involved but with some investment in planning, it can go smoothly. Many game studios use this to accelerate their compilation and builds today, so consider using this especially to deal with peak production phases where smooth, fast builds can really make a difference in making a ship date.

## References

The full Incredibuild documentation can be found [here](https://docs.incredibuild.com/win/latest/windows/components_and_architecture.html).
