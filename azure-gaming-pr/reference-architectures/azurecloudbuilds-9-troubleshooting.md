---
title: Azure Cloud Build Pipelines - Troubleshooting
description: This document contains troubleshooting FAQs related to the set up of the build pipeline.
author: whenslunch
keywords: 
ms.topic: reference-architecture
ms.date: 3/18/2022
ms.author: tzong
ms.service: azure-gaming
---
# Troubleshooting

**“A problem is a chance for you to do your best.” Duke Ellington**

### Can't allocate NV-SKU Virtual Machines

Please check your quota for the SKU and for the region you are deploying in. More info [here](/azure/azure-portal/supportability/per-vm-quota-requests)

### Can't find Perforce extensions in Azure DevOps

At the time of publishing, these extensions were in Private Preview, so please contact us with your Azure DevOps Enterprise or Organization name to be specifically included. Please email us at {address}.

### Personal Access Token (PAT) info lost, or has expired

PATs are generated once, then hidden forever. If you do not have the actual token copied somewhere temporarily, you will have to regenerate a new one.
PATs can expire, and as rule of thumb they should be valid for only as long as they are needed, then renewed as needed. Renewals can be done in the same Azure DevOps settings menu where the PATs were generated.

### Build of ShooterGame seems to take abnormally long on the build machine

If you have Incredibuild installed but no license, Incredibuild restricts execution to only a single core, even if your machine has multiple cores available. This can result in very slow compilation times. There are some articles on the Internet that talk about using Unreal Engine configuration files to disable Incredibuild in, for instance, the shader compilation phase, but possibly the best way to guarantee better performance in this case is to uninstall Incredibuild until such time you are able to secure at least a trial license.

### Azure DevOps service connection steps not showing up my Azure Subscriptions

As part of the Azure DevOps setup, you will create an Azure Resource Manager service connection. The first two choices in the menu are to do so either automatically, or manually.

The automatic way should auto-detect all the subscriptions associated with your login account and populate them on screen for you to choose. If it doesn't do that, try the manual method instead, whereby you have to specify the Subscription ID, etc.

Also make sure that the account you are logged into Azure DevOps also exists in the Azure Subscription.

### Azure DevOps is unable to copy the build to the storage account

This could be due to a fault service connection. In Azure DevOps, in Project settings, you can click into service connections and verify that it works. Please review the following docs carefully:
[Creating a service principal in Azure](/azure/active-directory/develop/howto-create-service-principal-portal).

[Create an Azure Resource Manager service connection](/azure/devops/pipelines/library/connect-to-azure?view=azure-devops#create-an-azure-resource-manager-service-connection-with-an-existing-service-principal)

### When building, Perforce is complaining that it cannot find the workspace on the build server

It seems like the workspace, also known as the client, is incorrectly configured, or was somehow entered wrongly as a variable value.
One thing to try is, on the build machine, run p4v and do a manual sync and rectify any errors there first.

### When building the code, the build agent has an error CS0042... Access is denied

When setting up the build workspace, set the option 'allwrite'. Check this link for more info: [Prevent Perforce from marking files as read-only](https://stackoverflow.com/questions/48195633/prevent-perforce-from-marking-files-as-read-only)

### During build, the pipeline complains that it cannot find the Unreal Automation Tool / RunUAT.bat

The pipeline code under task: PowerShell@2 is hard-coded to where the Azure Game Development VM installs Unreal Engine by default. If you are using your own installation, please replace the relevant code snippet to reflect where your installation of Unreal Engine.

### Out of disk space on the build machine

Because this is a demo, the VM was purposely set up with only one C: drive, to make the process faster and overall less costly. 
You can always attach a larger data drive to the VM. However, take care to re-configure the workspace and the Perforce root accordingly.

### I cannot access my Perforce commit server/build server

Make sure your network security groups are not blocking the relevant means of access (e.g. ports 22, 3389 for ssh and RDP access respectively). The VM itself has a firewall that you need to check.
A good way to ensure access is to either use [Just-In-Time VM access](/azure/defender-for-cloud/just-in-time-access-usage?tabs=jit-config-asc%2Cjit-request-asc) (if your Azure Security tier allows it) or use [Azure Bastion hosts](https://azure.microsoft.com/services/azure-bastion/#features) to access them.

### I can't find the fingerprint of my Perforce commit server

It appears when you first connect to the Perforce server from a new machine. The problem is that the pop-up it appears in does not allow you to copy and paste the fingerprint data, making it easy to lose it later. There are various resources on line that will show you how to do this within the server, please search in your favorite search engine.

### I can't add more users to my Perforce server

The free Perforce trial allows up to five users to be created on the system. Either delete existing users, or contact Perforce for a paid license.

## Next steps

Return to the [Introduction](./azurecloudbuilds-0-intro.md).
