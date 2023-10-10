---
title: Azure Cloud Build Pipelines - Testing
description: This section of the guide shows how to test the pipeline now that all components have been set up. This is part 8 of an 8 part series.
author: whenslunch
keywords: 
ms.topic: reference-architecture
ms.date: 3/18/2022
ms.author: tzong
ms.prod: azure-gaming
---

# Section 7: Testing

1. On the developer workstation, open a browser to your Azure DevOps project, navigate to your Pipelines.

2. In Unreal Editor, open the ShooterGame project and make a change, like add an object into the scene. Test the game in the Editor to make sure it runs fine.

3. Check in the change either directly in Unreal Editor or in p4v.

4. Refresh the browser showing your Azure DevOps pipelines. Your pipeline build should show as starting up.

>[!NOTE]
> Since this is the first time the pipeline is run, you may have to grant permissions for the pipeline to access resources. 
> A message will pop up with a message:
> "This pipeline needs permission to access a resource before this run can continue"
> Go ahead and click Permit.
> Again, you should only have to do this once for the project.

5. Wait for the build to be completed, archived and copied.

6. On your developer workstation, open Azure Storage Explorer. Sign in to your subscription and navigate to the storage container.

7. Download the build zip file to your dev workstation.

8. Unzip the build file, and click on ShooterGame.exe to run the game.

9. Verify your previous changes are in the build.

## Next steps

Next, go to Section 8: [Conclusion](./azurecloudbuilds-8-conclusion.md).

Or return to the [Introduction](./azurecloudbuilds-0-intro.md).

Troubleshooting page is [here](./azurecloudbuilds-9-troubleshooting.md).
