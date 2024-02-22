---
title: Connect from Visual Studio
description: Connect to Unreal Cloud DDC from Visual Studio  
author: dciborow
ms.topic: how-to
ms.date: 10/02/2022
ms.author: dciborow
ms.service: azure-gaming
---

# Connect to Unreal Cloud DDC from Visual Studio

[!INCLUDE [retire-banner](./includes/retire-banner.md)]

## Prerequisites

Software requirements for your device when connecting to Unreal DDC:

* User account must be added to the 'Enterprise Application' of the Application Service Principal as a `User` with `Cache` and `Storage` access.
* <a href="https://www.visualstudio.com/downloads/" target="_blank">Visual Studio 2022</a>
* Latest version of Unreal Engine and Unreal Editor. If you don't have Unreal Editor installed yet, see [Install the latest version of Unreal Editor](#install-the-latest-version-of-unreal-editor)

> [!TIP]
> You can also use a [Game Developer VM](integrate-with-gdvm.md) which comes with the above prerequisites installed.

## Install the latest version of Unreal Editor

1. Open PowerShell
1. Run the following example script

```powershell
git clone -b ue5-main https://github.com/EpicGames/UnrealEngine
cd .\UnrealEngine\
git checkout b5ce34c6e62294b6de6d3c5f35be0164a75cb3ba

.\Setup.bat
.\GenerateProjectFiles.bat
```

## Create a new Project

We highly recommend creating a new sample game if it's your first time using it. Skip to the next section if you have an existing game project and want to use it.

1. Open the Unreal Editor solution (.sln file) in Visual Studio 2022
1. Set the desired game project as the startup project
1. Build
1. Start debugging (F5)
1. Wait for the Unreal Editor to open
1. Create a new sample C++ game
1. Open that project in Visual Studio

## Connect and use Unreal Cloud DDC

### Edit DefaultEngine.ini

* Go to your Unreal game project's `DefaultEngine.ini` file and add a section using the information below. This is the game project you built in Step 3 above.
* Replace **hostname** with your hostname used during deployment, and set **unique-namespace-for-project**.

```ini
[ConsoleVariables]
DDC.Http.EnableOidc=1

[HordeDDC]
Root=(Type=KeyLength, Length=120, Inner=AsyncPut)
AsyncPut=(Type=AsyncPut, Inner=Hierarchy)
Hierarchy=(Type=Hierarchical, Inner=Local, Inner=Shared)
Local=(Type=FileSystem, PurgeTransient=true, UnusedFileAge=34, PromptIfMissing=true, Path=%ENGINEDIR%DerivedDataCache, EnvPathOverride=UE-LocalDataCachePath, EditorOverrideSetting=LocalDerivedDataCache)
Shared=(Type=Http, Host="https://<hostname>", ResolveHostCanonicalName=false, EnvHostOverride="UE-CloudDataCacheHost", CommandLineHostOverride=CloudDataCacheHost, Namespace="jupiter", StructuredNamespace="<unique-namespace-for-project>", OAuthProviderIdentifier="HordeStorage", ReadOnly=false)
```

### Add a new json file

* Add a new `Programs/OidcToken/oidc-configuration.json` file under the game project root folder with the information below.
* Substitute your values for **tenant-id**, **client-id**, **client-name**, and **app-uri**.

```json
{
    "OidcToken": {
        "Providers": {
            "HordeStorage": {
                // The /v2.0 at the end is important!
                "ServerUri": "https://login.microsoftonline.com/<tenant-id>/v2.0",
                // Client App Registration
                "ClientId": "<client-id>",
                "DisplayName": "<client-name>",
                // Port 6543 is chosen arbitrarily.
                "RedirectUri": "http://localhost:6543/callback",
                // The custom scope is from app registration whose scope is requested by the client app.
                "Scopes": "api://<app-uri>/Horde.ReadWrite openid profile offline_access"
            }
        }
    }
}
```

### Modify Visual Studio property

In Visual Studio, for the startup project, an additional property must be set.

* Go to Visual Studio
* Under debug properties and add `-ddc=HordeDDC` at the end of the command line and then start debugging.
* You may also use `-DDC-Local-MissRate=10` for testing which will force a percentage of cache misses and generate traffic to the deployment.

## See also

* [Tips, tricks, and techniques for setting up Visual Studio to work with Unreal Engine](https://docs.unrealengine.com/5.0/setting-up-visual-studio-development-environment-for-cplusplus-projects-in-unreal-engine/)
