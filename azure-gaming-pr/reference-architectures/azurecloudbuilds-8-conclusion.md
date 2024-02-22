---
title: Azure Cloud Build Pipelines - Conclusion
description: This document summarizes the demo build pipelines and sets out further steps a reader may take. It also contains links to other publicly-available example setups from third-party game studios.
author: whenslunch
keywords: 
ms.topic: reference-architecture
ms.date: 3/18/2022
ms.author: tzong
ms.service: azure-gaming
---
# Conclusion

Hopefully you've gone through all the steps and now have a basic pipeline that's working. If so, congratulations!

## Clean up

If you used the Azure Game Development VMs in the pipeline, you might want to stop and deallocate them when not in use - the NV-class VMs can be costly.  

---

## The journey begins here

Of course, this is **not** a production-grade pipeline yet! Build on this pipeline and make it really work for your studio's situation.

Below are just a few other areas to explore with the pipeline you've just built. 
Please contact us at azuregamedevs@microsoft.com for any feedback or inquiries.

## Identity Management

Azure Active Directory can be integrated with Perforce's Helix Authentication Service to provide  convenient, secure Single-Sign On capability for your Perforce users and the build system. Please take a look at how to [integrate AAD together with Perforce](./perforce-using-aad-together-with-perforce.md) and boost security in your workflow.

For more on this, see how to [use AAD Managed Identities with Perforce](./perforce-logging-on-with-a-managed-identity.md).

## Perforce Proxies

In this example pipeline, we have a single Perforce commit server. In real-world situations, however, a more advanced Perforce deployment topology is probably required to address the geographical locality of your development staff.  

Proxies are a way of balancing the load between local depots and the main commit server. Check out how to [set up a Perforce Proxy in Azure](./perforce-set-up-a-proxy-in-azure.md).

## Perforce Changelist Number
In this example pipeline, you saw the Azure DevOps Perforce extension used, as seen in this script fragment:
```yaml
- task: Perforce@1
  displayName: Sync Perforce to CL
  inputs:
  ...
```
This is useful for a build trigger per-check in, but in many cases, you will also want a regular timed build, e.g. nightly builds.

There is another Azure DevOps Perforce extension called FindPerforceChangeListNumber. This extension lets you specify the changelist number from within the Azure DevOps pipeline, including the head of a given branch. To do this, define a pipeline variable $(ChangelistNumber) that will contain the changelist that FindPerforceChangeListNumber will write into, then have the other Perforce extension consume that variable. For instance:

```yaml
- task: FindPerforceChangelistNumber@1
  displayName: Find Perforce CL Number
  inputs:
    STATUS: 'submitted'
    DEPOTPATH: '$(p4depotpath)'
    P4PORT: '$(p4port)'
    ssl: true
    fingerprint: '$(fingerprint)'
    P4USER: '$(p4user)'
    P4PASSWD: '$(p4pass)'
- task: Perforce@1
  displayName: Sync Perforce to CL
  inputs:
    UseDepotPath: true
    DepotPath: '$(p4depotpath)'
    P4ROOT: '$(p4root)'
    CreateWorkspace: false
    Sync: true
    Clean: false
    Force: false
    synctype: 'changelist'
    CHANGELIST: '$(ChangelistNumber)'
    P4CLIENT: '$(p4client)'
    P4PORT: '$(p4port)'
    ssl: true
    fingerprint: '$(fingerprint)'
    P4USER: '$(p4user)'
    P4PASSWD: '$(p4pass)'
```

This extension makes Perforce changelist operations more flexible from within Azure DevOps, so please check it out.


## Pixel Streaming

Azure Gaming and Epic Games have collaborated to integrate Unreal Engine Pixel Streaming capabilities into Azure. Check out [this guide](./unreal-pixel-streaming-at-scale.md) to see how you can enable builds to be streamed to production and QA staff for quick reviews, post-build.

## Customer examples

Other game developers are benefitting from Azure cloud build pipelines today. Please take a look at these examples:

[Bandai Namco Studios: Building a Cloud Native CI/CD Pipeline for Unity Games](https://www.youtube.com/watch?v=zT5r_B1ZtUc)

[Game Studio Inc. uses Azure to cloudify its development](https://developer.microsoft.com/games/customer-stories/game-studio-inc-uses-azure-to-cloudify-its-development/)

[Cloud Build Pipelines: The Coalition](https://developer.microsoft.com/games/customer-stories/cloud-build-pipelines-the-coalition/)

## Next steps

Return to the [Introduction](./azurecloudbuilds-0-intro.md).

Troubleshooting page is [here](./azurecloudbuilds-9-troubleshooting.md).
